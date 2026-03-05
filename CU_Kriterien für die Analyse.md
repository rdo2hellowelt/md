## Kriterien für die Analyse einzelner PL/SQL‑Funktionen

### 🔍 Bewertungsdimensionen
- **Komplexität der Logik** — Anzahl der Verzweigungen, Berechnungen, Abhängigkeiten.
- **Cursor‑Logik** — einfache SELECT‑Cursor vs. verschachtelte Cursor, Bulk‑Verarbeitung.
- **Dynamisches SQL** — statisch gut migrierbar, dynamisch oft komplex.
- **Oracle‑Spezifika** — DBMS_*‑Pakete, `%TYPE`, `%ROWTYPE`, Sequences, Autonomous Transactions.
- **Exception Handling** — einfache Fehler vs. komplexe Fehlercodes/Retry‑Logik.
- **Transaktionslogik** — COMMIT/ROLLBACK, Savepoints, autonome Transaktionen.
- **Seiteneffekte** — Änderungen an globalen Package‑Variablen, Tabellen-Inserts, E-Mail-Versand.

## Zusammenfassung der Kategorisierung (Realbestand)

| Kategorie | Anzahl | Typische Merkmale | Automatisierungspotenzial |
|-----------|--------|-------------------|---------------------------|
| **Einfach** | 1 | Reine Mathematik, keine DB | **hoch** (90 %) |
|  **Mittel** | 2 | Berechtigungen, ID-Generierung | **mittel** (50 %) |
| **Komplex** | 7 | Status-Engine, Sync, Multi-Table-Inserts | **niedrig** (15–30 %) |

---

## Fazit für die LinkVis Migration
Die reale Analyse zeigt, dass **70 % der Kernfunktionen als komplex** einzustufen sind. Dies bedeutet:
- Der Fokus liegt auf **manuellem Architektur-Redesign** (State Machine, Batch Processing, Orchestrierung).
- KI wird primär für **"Skelett-Code"**, **Unit-Test-Generierung** und **SQL-Mapping** eingesetzt.
- Die Migration ist weniger ein "Übersetzen" als vielmehr ein "Neu-Implementieren" der fachlichen Regeln in einer modernen Java-Umwelt.


---

## Reale Analyse der 10 LinkVis PL/SQL‑Funktionen

### 🟢 Kategorie: *Einfach*
**1. `PKG_APT_UTIL.FK_CALC_DISTANCE`**  
- **Logik:** Reine mathematische Berechnung (Haversine-Formel).  
- **Keine DB-Zugriffe:** Keine Cursor, keine Tabellen-Updates.  
- **Begründung:** Reine zustandslose Logik; kann fast 1:1 als Java-Utility portiert werden.

---

### 🟡 Kategorie: *Mittel*
**2. `PKG_APT_RECHTE.fk_getPermittedUsers`**  
- **Oracle-Spezifika:** Nutzung von `DBMS_LOB` und `BITAND` (Bitmasken).  
- **Seiteneffekte:** Keine (Read-Only).  
- **Begründung:** Logik ist klar abgegrenzt, erfordert aber modernes Design für Berechtigungen (z.B. Spring Security).

**3. `PKG_IF_APT_CLIENT.FK_GET_USER_LINK_ID`**  
- **Logik:** String-Manipulation und Prüfung der Verfügbarkeit in Tabellen.  
- **Seiteneffekte:** Keine direkten Schreibvorgänge.  
- **Begründung:** Logik ist gut in Java abbildbar, benötigt aber effiziente DB-Abfragen.

---

### 🔴 Kategorie: *Komplex / Sehr Komplex*
**4. `PKG_APT_STATUS.fk_setRevStatusDatum`**  
- **Logik:** Das Herz der Status-Engine; extrem viele fachliche Regeln und rekursive Trigger.  
- **Seiteneffekte:** Versendet E-Mails, schreibt Historien, triggert Folgestatus.  
- **Begründung:** Hochgradig proprietär; muss als State-Machine in Java neu designt werden.

**5. `PKG_IF_APT_LINK.FK_NEUES_LINK`**  
- **Seiteneffekte:** Schreibt synchron in über 8 verschiedene Tabellen.  
- **Oracle-Spezifika:** Intensive Nutzung von Sequences und DUAL-Queries.  
- **Begründung:** Massive Orchestrierung; erfordert saubere Transaktionssteuerung im Applikationslayer.

**6. `PKG_IF_APT_IMPORT.FK_ABGLEICH_STANDORTE`**  
- **Cursor-Logik:** Verschachtelte Cursor mit bis zu 1 Mio. Iterationen.  
- **Performance:** Kritische Synchronisationslogik.  
- **Begründung:** Geringes Automatisierungspotenzial; muss für Batch-Performance (PostgreSQL) optimiert werden.

**7. `PKG_APT_PERF.PR_RECHNE_LINK_KPI`**  
- **Mathematik:** Komplexe statistische Berechnungen über Zeitreihen (14 Tage).  
- **Logik:** Mehrfach verschachtelte Loops über Links.  
- **Begründung:** Portierbar, erfordert aber ein effizientes Batch-Processing-Konzept in Java.

**8. `PKG_IF_APT_TRANSFER.FK_TRANSFER_LINK`**  
- **Transaktionslogik:** Explizite Rollbacks und Commits innerhalb von Schleifen.  
- **Datenfluss:** Verarbeitet Differenz-Tabellen.  
- **Begründung:** Transaktionssteuerung muss komplett in den Spring Service Layer wandern.

**9. `PKG_IF_APT_INST_DEINST.FK_IMPORT_RESERVIERUNG`**  
- **Seiteneffekte:** Lösch- und Insert-Logik basierend auf externen Sichten (Pepsy).  
- **Logik:** Verschachtelte FOR-Loops.  
- **Begründung:** Klassischer ETL-Prozess; KI kann bei Mappings helfen, Logik muss manuell validiert werden.

**10. `PKG_IF_APT_CLIENT.FK_CREATE_PARALLEL_LINK`**  
- **Logik:** Deep-Copy eines komplexen Link-Objekts inkl. Revisionen und Endstellen.  
- **Begründung:** Erfordert sauberes Object-Mapping (JPA/Hibernate) und Klon-Logik in Java.

---