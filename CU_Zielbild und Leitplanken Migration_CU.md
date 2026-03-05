## Architektur‑Memo: Zielbild und Leitplanken für eine KI‑gestützte PL/SQL‑Migration

Dieses Memo fasst das Zielbild und die technischen Leitplanken für die Migration von Oracle‑PL/SQL hin zu einer modernen, serviceorientierten Architektur zusammen. Es dient als verbindliche Orientierung für Analyse, Design und Umsetzung der Migration – insbesondere für KI‑gestützte Übersetzungs‑ und Refactoring‑Schritte.

---

## 1. Zielarchitektur für Geschäftslogik und Datenzugriff

### Serviceorientiertes Anwendungsdesign
Die Geschäftslogik wandert aus PL/SQL‑Paketen in klar abgegrenzte Services, typischerweise auf Basis von Spring Boot. Diese Services kapseln Use Cases, orchestrieren Abläufe und steuern Transaktionen. Die Architektur folgt einem modularen Aufbau, der Domänen voneinander trennt und Abhängigkeiten minimiert.

### Persistenzschicht mit klaren Verantwortlichkeiten
Der Datenzugriff erfolgt über Repository‑ oder DAO‑Komponenten. SQL wird transparent im Anwendungscode gehalten, sodass Logik nicht mehr in der Datenbank verborgen ist. Je nach Komplexität kommen JPA, Spring Data oder MyBatis zum Einsatz. 
- Für Massendaten-Synchronisation wird **Spring Batch** zur Chunk-basierten Verarbeitung eingesetzt.
- Die Datenbank dient primär als Speicher, nicht als Logik‑Engine.

### Rolle der Datenbank im Zielbild
PostgreSQL übernimmt die Funktion des relationalen Backends. Logik in der Datenbank wird auf das notwendige Minimum reduziert: Constraints, einfache Trigger und wenige technische Hilfsfunktionen. Komplexe PL/SQL‑Pakete (z.B. die Status-Engine) werden durch **Spring State Machine** in der Anwendung abgelöst.

---

## 2. Zielcode‑Standard und Qualitätsanforderungen

### Struktur und Modularität
Der Zielcode folgt einer klaren Schichtung:
- Domain (Entities, Value Objects, Domain Services)
- Application (Use Cases, Service Layer)
- Infrastructure (Repositories, DB‑Zugriff, Messaging)

Die Struktur orientiert sich an Clean Architecture oder Hexagonal Architecture, um Testbarkeit und Austauschbarkeit zu gewährleisten.

### Coding‑Guidelines und Qualitätssicherung
Java 17+ und Spring Boot 3+ bilden den Standard. Code wird nach SOLID‑Prinzipien gestaltet, mit sprechenden Namen und kleinen, fokussierten Methoden. Qualitätssicherung umfasst Unit‑Tests, statische Analyse und definierte Coverage‑Ziele. KI‑generierter Code wird grundsätzlich manuell geprüft und in die Architektur eingeordnet.

### Umgang mit KI‑Unterstützung
KI dient als Übersetzer, Refactoring‑Helfer und Dokumentationsassistent. Sie liefert Vorschläge, aber keine finalen Implementierungen. Entwickler prüfen, korrigieren und integrieren die Ergebnisse. Die KI unterstützt bei der Zerlegung von PL/SQL‑Paketen, der Generierung von Java‑Skeletons und der Ableitung von SQL‑Statements für PostgreSQL.

---

## 3. Transaktionsmodell im Anwendungslayer

### Steuerung über den Service Layer
Transaktionen werden ausschließlich im Anwendungscode gesteuert, typischerweise über Spring‑Annotationen. Die Datenbank führt keine impliziten oder autonomen Transaktionen mehr aus. Die Grenzen einer Transaktion orientieren sich an fachlichen Use Cases.

### Ablösung von PL/SQL‑Transaktionslogik
COMMIT/ROLLBACK‑Anweisungen in PL/SQL entfallen vollständig. Autonomous Transactions werden durch asynchrone Mechanismen ersetzt, etwa Messaging oder separate Services. Das Ziel ist ein konsistentes, nachvollziehbares Transaktionsverhalten im Anwendungscode.

---

## 4. Logging‑ und Exception‑Konzept

### Zentrales Logging
Logging erfolgt strukturiert über SLF4J und ein zentrales Logging‑Framework. Log‑Ebenen sind klar definiert, und technische Details werden nur in Debug‑ oder Trace‑Level protokolliert. DBMS_OUTPUT‑Mechanismen werden vollständig ersetzt.

### Einheitliches Fehlerhandling
Fachliche Fehler werden als Domain‑Exceptions modelliert, technische Fehler als separate Exception‑Typen. Ein globales Exception‑Handling sorgt für konsistente API‑Antworten und Fehlercodes. PL/SQL‑Fehlercodes werden in ein einheitliches Fehlerkonzept überführt.

---

## 5. Rahmenbedingungen und Migrationskontext

### Oracle‑Exit als strategisches Ziel
Die Migration zielt auf eine vollständige Ablösung von Oracle als Laufzeitplattform. Neue Entwicklungen erfolgen ausschließlich im Zielstack. Bestehende PL/SQL‑Pakete, Trigger und Views werden schrittweise stillgelegt oder ersetzt.

### PostgreSQL als Zielplattform
PostgreSQL wird als primäre relationale Datenbank genutzt. Oracle‑spezifische Funktionen wie DBMS_SCHEDULER, DBMS_CRYPTO oder hierarchische Abfragen werden durch Java‑Frameworks, PostgreSQL‑Features oder alternative Architekturen ersetzt.

### Nicht‑funktionale Anforderungen
Performance, Sicherheit und Observability sind integrale Bestandteile der Zielarchitektur. Logging, Metriken und Tracing werden zentral bereitgestellt. Rollen und Rechte werden aus der Datenbanklogik in Applikations‑ und DB‑Security überführt.

---

## 6. Scope der KI‑gestützten Migration

### Im Fokus
Die KI unterstützt bei der Analyse von PL/SQL‑Paketen, der Identifikation von Mustern, der Ableitung von Zielstrukturen und der Generierung von Code‑Skeletons. Sie hilft bei der Dokumentation und beim Refactoring.

### Nicht im Fokus
Eine vollautomatische Migration ist nicht vorgesehen. Architekturentscheidungen, fachliche Validierung und finale Implementierung bleiben in der Verantwortung der Entwickler.

---
