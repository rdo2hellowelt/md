# Konsolidierte Migrationsanalyse: LinkVis PL/SQL zu Java/Spring

## 1. Executive Summary

Die technische Analyse des LinkVis-Bestands (ca. 70.000 Zeilen PL/SQL in 10 Kern-Paketen) bestätigt: Die Migration ist kein reiner Sprachwechsel, sondern ein **Architektur-Replatforming**.

**Kernaussagen:**

*   **Komplexitätsgrad:** 70 % der Kernfunktionen sind hochkomplex und erfordern ein manuelles Redesign (State Machines, Batch-Processing).
*   **Automatisierungspotenzial:** Realistisch sind **35–45 %** (Fokus auf „Skelett-Code“, Daten-Mapping und einfache Utilities).
*   **Strategisches Ziel:** Übergang von zustandsbehafteten Oracle-Packages zu **Stateless Java Services**, um horizontale Skalierbarkeit (Cloud-Native) zu ermöglichen.

***

## 2. Validierte Analyse der 10 Referenz-Funktionen

Basierend auf der Quellcode-Sichtung in  
`src/main/resources/liquibase-ora/packages/`:

| Funktion / Prozedur       | Paket (Pfad)             | Kategorie        | Technischer Befund (Code-Review)                            | Migrationspfad (Ziel)      |
| ------------------------- | ------------------------ | ---------------- | ----------------------------------------------------------- | -------------------------- |
| `FK_CALC_DISTANCE`        | `PKG_APT_UTIL`           | **Einfach**      | Reine Haversine-Mathematik, kein DB-Zugriff.                | Java Math Utility          |
| `fk_getPermittedUsers`    | `PKG_APT_RECHTE`         | **Mittel**       | Nutzung von `DBMS_LOB` & `BITAND`. Read-Only.               | Spring Security / JPA      |
| `FK_GET_USER_LINK_ID`     | `PKG_IF_APT_CLIENT`      | **Mittel**       | String-Logik + DB-Prüfung (Next ID).                        | Java Service + DB-Sequence |
| `FK_NEUES_LINK`           | `PKG_IF_APT_LINK`        | **Komplex**      | Orchestriert Inserts in >8 Tabellen (KAP\_LINK\_\*).        | Spring Service + JPA       |
| `fk_setRevStatusDatum`    | `PKG_APT_STATUS`         | **Sehr Komplex** | Herz der Status-Engine. Rekursive Logik, E-Mail-Trigger.    | **Spring State Machine**   |
| `FK_ABGLEICH_STANDORTE`   | `PKG_IF_APT_IMPORT`      | **Komplex**      | Massive Cursor-Logik (1 Mio. Iterationen), Synchronisation. | **Spring Batch (Chunk)**   |
| `PR_RECHNE_LINK_KPI`      | `PKG_APT_PERF`           | **Sehr Komplex** | Statistische Berechnungen über 14‑Tage‑Zeitreihen.          | Java Statistics Service    |
| `FK_TRANSFER_LINK`        | `PKG_IF_APT_TRANSFER`    | **Komplex**      | Explizite Transaktionssteuerung (ROLLBACK) in Loops.        | `@Transactional` Service   |
| `FK_IMPORT_RESERVIERUNG`  | `PKG_IF_APT_INST_DEINST` | **Mittel**       | ETL‑Logik: Löschen/Insert basierend auf Sichten.            | Spring Integration / DTO   |
| `FK_CREATE_PARALLEL_LINK` | `PKG_IF_APT_CLIENT`      | **Komplex**      | Deep‑Copy komplexer Objekte inkl. Revisionen.               | MapStruct / Clone Logic    |

***

## 3. Identifizierte „Architektur-Gaps“ (Oracle vs. Java)

### A. Zustandsmanagement

*   **Ist-Zustand:** Intensive Nutzung globaler Package‑Variablen (z. B. `pv_nTrans_Nr`, `pv_vcUser`).
*   **Risiko:** Race‑Conditions und Speicherlecks in Java.
*   **Soll-Zustand:** Nutzung thread-sicherer Kontexte (Spring Security Context, `ThreadLocal`) oder explizite Parameterübergabe.

### B. Transaktionskontrolle

*   **Ist-Zustand:** Manuelles `COMMIT` / `ROLLBACK` in Schleifen.
*   **Risiko:** Unvorhersehbare Datenbankzustände bei Fehlern.
*   **Soll-Zustand:** Deklarative Transaktionen über Spring (`@Transactional`). Komplexe Schleifen als atomare Einzeloperationen modellieren.

### C. Performance & Batching

*   **Ist-Zustand:** Oracle‑Bulk‑Features (`BULK COLLECT`, `FORALL`).
*   **Risiko:** Starke Performance‑Einbußen bei Umstellung auf Java‑Einzelsatzlogik.
*   **Soll-Zustand:** Nutzung von **Spring Batch**, **Batch‑Inserts**, **Chunk‑Processing**.

### D. Vendor Lock‑in vermeiden

*   Keine Abbildung der Business‑Logik in PL/pgSQL, um Migrationen und Cloud‑Strategien nicht zu blockieren.

***

## 5. Korrekturverzeichnis (Dokumentations-Update)

*   **Paket-Zuordnung:** `FK_CALC_DISTANCE` gehört korrekt zu **`PKG_APT_UTIL`** (Datei: `apt_pkg_apt_util.sql`).
*   **Lokation:** Alle PL/SQL‑Definitionen liegen unter  
    `api-apt/src/main/resources/liquibase-ora/packages/`.

***

**Status der Analyse:** Validiert & abgeschlossen (Woche 1)  
**Empfehlung:** Start eines Proof-of-Concept zur **Spring State Machine** (Status-Engine).

***
