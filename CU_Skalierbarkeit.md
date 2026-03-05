# Skalierbarkeit & Risiken: LinkVis PL/SQL-Migration (W1)

Dieses Dokument bewertet die Skalierbarkeit des Migrationsansatzes für das LinkVis-Projekt (ca. 70.000 Zeilen PL/SQL) und identifiziert die kritischen Risikofaktoren sowie die Grenzen der KI-Unterstützung.

---

## 1. Skalierbarkeit des Vorgehens (LinkVis-Fokus)

Die Skalierbarkeit der Migration hängt primär von der **Homogenität der Logik** und der Modularität des Ziel-Designs ab.

### Faktoren, die die Skalierung begünstigen
- **Stateless Service-Architektur:** Der Wechsel von zustandsbehafteten PL/SQL-Packages zu **Stateless Java Services** erlaubt die horizontale Skalierung (Cloud/Kubernetes) und eine parallele Bearbeitung durch mehrere Teams.
- **Blueprint-Ansatz:** Wiederkehrende Muster (z. B. für `GeoUtils`, `PermissionEvaluator`, `Repository-Layer`) dienen als Schablone für die gesamte Migration.
- **Entkopplung:** Durch die Aufteilung der 10 monolithischen Pakete in fachliche Micro-Services wird der Migrations-Durchsatz erhöht.

### Faktoren, die die Skalierung begrenzen (Bottlenecks)
- **Zentralisierte Kernlogik:** Die Status-Engine (`PKG_APT_STATUS`) bildet den kritischen Pfad. Da fast alle Prozesse hierauf zugreifen, ist eine Parallelisierung in diesem Bereich kaum möglich.
- **Sequentielle Datenabhängigkeiten:** Komplexe Trigger-Ketten erfordern eine schrittweise Validierung, um Seiteneffekte in der Datenbank-Integrität zu vermeiden.

---

## 2. Automatisierungspotenzial & KI-Nutzung

Basierend auf der Gap-Analyse der Pilot-Transformation ergibt sich ein differenziertes Bild des Automatisierungspotenzials.

### 2.1 Eignung der Funktionstypen
| Kategorie | Anteil | Skalierbarkeit (KI) | Strategie |
|:---|:---|:---|:---|
| **Einfach (Utility)** | 10 % | **Hoch (ca. 90 %)** | Fast vollständige KI-Portierung (z. B. Mathe, Strings). |
| **Mittel (Logik)** | 20 % | **Mittel (ca. 50 %)** | KI generiert Skelett und DTOs; manuelles Mapping nötig. |
| **Komplex (Core)** | 70 % | **Niedrig (ca. 15-25 %)** | **Manuelles Architektur-Redesign** zwingend erforderlich. |

**Erwarteter Gesamt-Automatisierungsgrad: 35–45 %**

### 2.2 Grenzen der KI-Nutzung
KI-Modelle (LLMs) stoßen bei der LinkVis-Migration an folgende Grenzen:
- **Architektur-Shift:** KI kann Code übersetzen, aber kein Redesign von prozeduraler Logik in eine objektorientierte **State-Machine** oder **Batch-Processing** (Spring Batch) durchführen.
- **Kontext-Limitierung:** Bei Packages mit >5.000 Zeilen verliert die KI den Überblick über globale Abhängigkeiten (Package-Variablen).
- **Oracle-Spezifika:** Proprietäre Funktionen (`DBMS_LOB`, `Autonomous Transactions`) werden oft fehlerhaft oder gar nicht in Java-Äquivalente übersetzt.
- **NULL-Semantik:** Die unterschiedliche Behandlung von leeren Strings zwischen Oracle (NULL) und Java ("") führt zu subtilen Logikfehlern, die KI-Modelle oft übersehen.

---

## 3. Kritische Funktionen (Showstopper)

Folgende Funktionstypen sind kritisch für den Projekterfolg und eignen sich **nicht** für eine Automatisierung:
1.  **Status-Engine (`PKG_APT_STATUS`):** Das Herzstück der Fachlogik; erfordert ein vollständiges Redesign als State-Machine.
2.  **Synchronisations-Logik (`PKG_IF_APT_IMPORT`):** Massendatenverarbeitung mit verschachtelten Cursorn; muss auf **Spring Batch/Chunk-Processing** umgestellt werden.
3.  **Globale Package-Variablen:** Diese müssen manuell in **Thread-Safe** Konzepte (z. B. Request-Scoped Beans) überführt werden, um Race-Conditions in Java zu vermeiden.

---

## 4. Risikoanalyse & Gegenmaßnahmen

### Fachliche Risiken
- **Verlust impliziter Logik:** Im Hintergrund laufende PL/SQL-Trigger.
  - *Gegenmaßnahme:* Explizite Modellierung im Java-Service-Layer (z. B. Hibernate Envers für Historisierung).
- **Fehlinterpretation der Statusregeln:** Falsche Übergänge in der State-Machine.
  - *Gegenmaßnahme:* Validierung via **Dual-Run Protokoll** (Vergleich Alt- vs. Neusystem).

### Technische Risiken
- **Performance-Verlust:** Wegfall von Oracle-Bulk-Collect/Forall.
  - *Gegenmaßnahme:* Nutzung von Spring Batch und optimierten JPA-Queries zur Vermeidung von N+1 Problemen.
- **Transaktions-Deadlocks:** Fehlerhafte `@Transactional`-Grenzen in Java.
  - *Gegenmaßnahme:* Striktes Layering und automatisierte transaktionale Integrationstests.

### Compliance & Rechtliche Risiken
- **Datenkonsistenz (ID-Vergabe):** Fehler bei der Portierung der `Jump+20`-Logik können zu Inkonsistenzen in iQ.link führen.
  - *Gegenmaßnahme:* 100% Testabdeckung der ID-Service-Methoden gegen DB-Sichten.
- **Revisionssicherheit:** Die Nachvollziehbarkeit von Änderungen (Historisierung) muss lückenlos gewährleistet sein.
  - *Gegenmaßnahme:* Audit-Logging auf Applikationsebene; Verzicht auf "versteckte" DB-Logik.
- **Vendor-Lock-in:** Unbeabsichtigte Nutzung von PostgreSQL-spezifischem PL/pgSQL.
  - *Gegenmaßnahme:* Architektur-Vorgabe: Business-Logik **nur** in Java; DB dient nur als Datenspeicher.

---

## 5. Fazit zur Skalierbarkeit (W1)

Das Vorgehen ist **architektonisch hochgradig skalierbar** (Übergang zu Cloud-Native), aber **operativ betreuungsintensiv**. 
- KI dient als **Beschleuniger für die Analyse und Skelettierung** (Mapping).
- Die eigentliche Skalierung des Durchsatzes erfordert ein festes Team, welche die KI-Vorschläge validieren und das Redesign der komplexen Kernelemente steuern. 
- Ein reiner "Automatisierungs-Ansatz" würde die technischen Schulden der PL/SQL-Welt nur in die Java-Welt transformieren, ohne die Vorteile der neuen Architektur zu heben.
