## 🎯 Kernaussage
Für eine erfolgreiche Migration müssen PL/SQL‑Pakete **strukturell zerlegt**, **abhängigkeitsfrei dokumentiert**, **Oracle‑spezifische Features identifiziert** und anschließend in **äquivalente Konstrukte der Zielplattform** überführt werden. 

Tools wie *Ora2Pg* unterstützen, ersetzen aber keine manuelle Architektur‑Analyse.  
  [Ora2Pg](https://ora2pg.darold.net/slides/ora2pg-best-practices.pdf)

---

## 🧩 1. Welche Pakete analysieren? (Repräsentative Auswahl)
Wir wählen 5–10 Pakete, die möglichst viele der folgenden Merkmale abdecken:

- **Geschäftslogik** (z. B. Validierungen, Berechnungen)
- **Datenzugriff** (CRUD‑Operationen, Bulk‑Verarbeitung)
- **Transaktionslogik** (COMMIT/ROLLBACK‑Steuerung)
- **Oracle‑Spezifika** (z. B. DBMS_*‑Pakete, Autonomous Transactions)
- **Performance‑kritische Bereiche** (Pipelined Functions, Bulk Collect)
- **Scheduler‑ oder Trigger‑Abhängigkeiten**

Diese Auswahl bildet die Grundlage für eine realistische Aufwandsschätzung.

---

## 🧭 2. Analyseleitfaden für jedes PL/SQL‑Paket

### 🔍 A. Strukturelle Analyse
- Anzahl und Typen der **Funktionen/Prozeduren**
- Parameter, Rückgabewerte, Exceptions
- Nutzung von **Package‑Variablen** (Stateful Logic → kritisch für Migration)
- Abhängigkeiten zu Tabellen, Views, Sequenzen, anderen Paketen

### 🧪 B. Technische Analyse
- Oracle‑spezifische Features:
  - `DBMS_OUTPUT`, `DBMS_SCHEDULER`, `DBMS_CRYPTO`, `DBMS_LOB`
  - `%TYPE`, `%ROWTYPE`
  - `PRAGMA AUTONOMOUS_TRANSACTION`
  - `BULK COLLECT`, `FORALL`
  - Pipelined Table Functions
- SQL‑Features:
  - Hierarchical Queries (`CONNECT BY`)
  - Analytical Functions
  - MERGE‑Statements

### 🧠 C. Logische Analyse
- Geschäftsregeln
- Validierungslogik
- Fehlerbehandlung
- Transaktionsgrenzen

### 📊 D. Komplexitätsbewertung
- LOC (Lines of Code)
- Cyclomatic Complexity
- Anzahl externer Abhängigkeiten
- Nutzung dynamischen SQLs

Tools wie **Visual Expert** unterstützen diese statische Analyse.  
  [Visual Expert](https://www.visual-expert.com/EN/stored-procedure-pl-sql-oracle-plsql/oracle-migration.html)

---

## 🔄 3. Migrationspfade für typische PL/SQL‑Konstrukte

### 🟦 A. Datenzugriff
| Oracle PL/SQL | PostgreSQL PLpgSQL | Hinweise |
|---------------|--------------------|----------|
| `%TYPE`, `%ROWTYPE` | ähnlich verfügbar | meist 1:1 migrierbar |
| `BULK COLLECT`, `FORALL` | `FOREACH`, `INSERT … SELECT` | Performance‑Tuning nötig |
| Sequenzen | Sequenzen | meist kompatibel |

### 🟦 B. Oracle‑spezifische Pakete
| Oracle | Alternative |
|--------|-------------|
| `DBMS_SCHEDULER` | PostgreSQL `pg_cron` oder externe Scheduler |
| `DBMS_CRYPTO` | `pgcrypto` |
| `DBMS_OUTPUT` | Logging‑Framework |

### 🟦 C. Transaktionslogik
- Autonomous Transactions → **nicht direkt verfügbar**, muss umgebaut werden (z. B. separate Worker‑Prozesse oder Event‑Queues).

### 🟦 D. Pipelined Functions
- In PostgreSQL → **Set‑Returning Functions (SRF)**  
- Performance‑Verhalten unterscheidet sich → Tests notwendig.

---

## 🛠️ 4. Tool‑Unterstützung
### Ora2Pg (für Oracle → PostgreSQL)
- erkennt PL/SQL‑Konstrukte
- generiert PLpgSQL‑Code
- liefert Migrationsberichte  
  [Ora2Pg](https://ora2pg.darold.net/slides/ora2pg-best-practices.pdf)

### Visual Expert
- tiefgehende Code‑Analyse
- Abhängigkeitsgraphen
- Impact‑Analysen  
  [Visual Expert](https://www.visual-expert.com/EN/stored-procedure-pl-sql-oracle-plsql/oracle-migration.html)

---

## 🚧 5. Typische Migrationsprobleme
- Package‑Variablen → Zielsysteme unterstützen keine stateful Packages
- Exception‑Handling → Unterschiede in Fehlercodes
- Performance‑Regressionen durch fehlende Oracle‑Optimierungen
- Proprietäre SQL‑Features (z. B. `CONNECT BY`)
- Unterschiedliche Transaktionsmodelle

---

## 📐 6. Ergebnis: Migrationsbericht pro Paket
Für jedes der 5–10 Pakete entsteht idealerweise:

1. **Funktionsmatrix** (Welche Funktion macht was?)
2. **Abhängigkeitsdiagramm**
3. **Oracle‑Spezifika‑Liste**
4. **Migrationsrisiken** (z.B. Trigger-Kaskaden, State-Management)
5. **Realistische Automatisierungsrate** (Basierend auf W1: **35–45 %**)
6. **Empfohlener Zielarchitektur‑Ansatz** (z.B. Spring State Machine, Batch)

---

## 💡 Erkenntnis aus Woche 1 (LinkVis Pilot)
Die Analyse der ersten 10 Kernpakete zeigt, dass **70 % der Funktionen als komplex** einzustufen sind. Dies verschiebt den Fokus von einer reinen "Code-Übersetzung" hin zu einem **Architektur-Redesign** im Java-Service-Layer.
