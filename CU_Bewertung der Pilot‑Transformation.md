# Bewertung der Pilot‑Transformation: LinkVis Projekt-Spezifisch (Gap-Analyse W1)

Dieses Dokument bewertet den Automatisierungsgrad der PL/SQL-Migration für das LinkVis-Ökosystem basierend auf der Analyse von 10 repräsentativen Funktionen und der systematischen Gap-Analyse der Pilot-Transformation.

---

## 1. Realistischer Automatisierungsgrad (LinkVis-Bestand)

Die Analyse zeigt, dass der LinkVis-Bestand aufgrund seiner gewachsenen Struktur (seit 2006) eine hohe Komplexitätsdichte aufweist.

| Kategorie | Anteil (LinkVis) | Automatisierung | Manueller Aufwand | Beschreibung / Beispiele |
|:----------|:-----------------|:----------------|:------------------|:--------------------------|
| **Einfach** | **10 %** | **90 %** | **10 %** | Reine Mathematik/Utility (`FK_CALC_DISTANCE`) |
| **Mittel** | **20 %** | **50 %** | **50 %** | ID-Generierung, Rechteprüfung (`FK_GET_USER_LINK_ID`) |
| **Komplex** | **70 %** | **15–25 %** | **75–85 %** | Status-Engine, Sync-Logik, Multi-Table-Inserts |

### Zusammenfassende Einschätzung
**Gesamt-Automatisierungsgrad: ≈ 35–45 %**  
Der Großteil des Codes (ca. 60 %) erfordert ein **Architektur-Redesign**, um technische Schulden nicht 1:1 in die neue Java-Welt zu übernehmen.

---

## 2. Systematische Bewertung der Pilot-Transformation

### 2.1 Wie viel Code war direkt verwendbar?
*   **Direkt verwendbar (~15 %):** Mathematische Formeln (Haversine), einfache String-Konpaktierung und statische SQL-Statements (nach JPA-Konvertierung).
*   **Als Skelett nutzbar (~30 %):** Die Grundstruktur von Prozeduren (Parameter, Variablendeklarationen) dient als Vorlage für Java-Methoden, erfordert aber eine vollständige Logik-Migration.
*   **Nicht verwendbar (~55 %):** Oracle-spezifische Prozeduren wie `DBMS_LOB`, `DBMS_OUTPUT`, Cursor-Management und explizite Transaktionssteuerung (`COMMIT`/`ROLLBACK`).

### 2.2 Manuelle Korrekturbedarfe
*   **Exception-Handling:** Transformation von `RAISE_APPLICATION_ERROR` in eine strukturierte Java-Exception-Hierarchie (z. B. `BusinessException`).
*   **Transaktions-Management:** Entfernen von Commits innerhalb von Schleifen; stattdessen Nutzung von Spring's `@Transactional`.
*   **Side-Effects:** Logik, die in PL/SQL über Trigger gelöst war (z. B. Historisierung in `KAP_LINK_REVISION`), muss manuell in den Java-Service-Layer integriert werden.

---

## 3. Fehleranalyse & Problemfelder

### 3.1 Technische Fehlerquellen
*   **NULL-Semantik:** Oracle behandelt leere Strings oft als NULL. In Java führt dies zu `NullPointerException` oder fehlerhaften Logik-Ergebnissen, wenn nicht explizit geprüft wird.
*   **Datentyp-Mismatch:** Mapping von Oracle `NUMBER` (ohne Precision) auf `Long`, `Integer` oder `BigDecimal`.
*   **Bulk-Performance:** 1:1 Portierung von Cursor-basierten Loops führt in Java/JPA zu "N+1 Query"-Problemen.

### 3.2 Fachliche Fehlerquellen
*   **Versteckte Globale Zustände:** Nutzung von Package-Variablen (`pv_nTrans_Nr`) führt in einer multi-threaded Java-Umgebung zu Race Conditions. Diese müssen durch Request-Scoped Beans oder ThreadLocals ersetzt werden.
*   **Namespace-Logik:** Die komplexe "Jump + 20"-Logik bei der ID-Vergabe (`FK_GET_USER_LINK_ID`) ist hochgradig kundenindividuell und führt bei ungenauer Portierung zu Dateninkonsistenzen in iQ.link.

---

## 4. Pattern-Evaluierung

### 🟢 Erfolgreiche Patterns (Good Fit)
*   **Strategy Pattern:** Für die verschiedenen Link-Typen und deren Validierung (statt riesiger `IF-THEN-ELSE` Blöcke).
*   **JPA Repositories:** Ersetzen komplexe Cursor-Logik durch deklarative Abfragen.
*   **DTO-Mapping:** Saubere Trennung zwischen DB-Entität und API-Response.

### 🔴 Problembehaftete Patterns (Bad Fit)
*   **1:1 Portierung von "God Packages":** PL/SQL-Pakete mit >5.000 Zeilen müssen in modulare Services aufgeteilt werden.
*   **In-Loop-Commits:** Führen zu instabilen Zuständen bei Abbruch; müssen durch atomare Transaktionen ersetzt werden.

---

## 5. Fazit & Problemfelderliste (W1)

**Kritische Problemfelder:**
1.  **Status-Engine (Sehr Komplex):** Erfordert ein Redesign als State Machine.
2.  **Synchronisations-Logik:** Muss auf Spring Batch umgestellt werden.
3.  **Globale Package-Variablen:** Erfordern ein neues Thread-Safe Konzept in Java.
4.  **Integration externer Views:** Zugriff auf iQ.link (`V_IF_APT_LINKIDS`) muss performant über JPA-Interfaces gelöst werden.

**Empfehlung:** Die Pilot-Transformation bestätigt, dass eine rein KI-basierte Übersetzung nicht ausreicht. Der Fokus muss auf der **manuellen Architektur-Orchestrierung** durch Senior-Entwickler liegen, während die KI für die Implementierung der atomaren Logik-Bausteine eingesetzt wird.
