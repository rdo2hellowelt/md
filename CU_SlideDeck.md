# Slide Deck: LinkVis Migrationsanalyse (PL/SQL → Java/Spring Boot)

**Meeting mit Senior Developer | Technische Strategie & Roadmap**

***

## **Slide 1: Executive Summary & Projektkontext**

*   **Bestand:** \~70.000 Zeilen PL/SQL verteilt auf 10 Kernpakete.
*   **Ziel:** Vollständiges **architektonisches Re-Platforming** von einer zustandsbehafteten Oracle-DB-Logik hin zu **stateless Java/Spring-Boot-Services** (Zieldatenbank: PostgreSQL).
*   **Zentrale Erkenntnisse:**
    *   **70 % der Kernlogik ist „hochkomplex“** (State Machines, verschachtelte Cursor, rekursive Trigger).
    *   **Automatisierungspotenzial (KI/LLM):** 35–45 % (Skeleton-Code, DTO-Mapping, einfache Berechnungen).
    *   **Strategische Neuausrichtung:** Weg von DB-zentrierter Business-Logik hin zu **Cloud-Native Spring Services**.

***

## **Slide 2: Analyse-Framework (Bewertungsdimensionen)**

Zur Bewertung des Migrationsaufwands wurden 10 repräsentative Funktionen anhand folgender Kriterien analysiert:

*   **Logik-Komplexität:** Verzweigungsgrad, rekursive/verschachtelte Berechnungen.
*   **Datenzugriffsmuster:** Einfache `SELECT`s vs. komplexe Cursor und `BULK COLLECT`.
*   **Oracle-Spezifika:** Abhängigkeiten von `DBMS_*`, `%TYPE`, `%ROWTYPE`, autonomen Transaktionen.
*   **Side-Effects:** Nutzung globaler Package-Variablen, Multi-Table-Inserts, externe Trigger (E-Mail, Audit).
*   **Transaktionsmodell:** Manuelle `COMMIT`/`ROLLBACK`-Steuerung in Schleifen.

***

## **Slide 3: Komplexitätsverteilung & Automatisierungsstrategie**

| Kategorie                   | Anteil | Strategie                                                 | Automatisierung (KI) |
| --------------------------- | ------ | --------------------------------------------------------- | -------------------- |
| **Einfach (Utility)**       | 10 %   | 1:1 Portierung der mathematischen Logik in Java-Utilities | **Hoch (90 %)**      |
| **Mittel (Orchestrierung)** | 20 %   | Refactoring zu JPA / Spring Security                      | **Mittel (50 %)**    |
| **Komplex (Core Engines)**  | 70 %   | **Manuelles architektonisches Redesign**                  | **Gering (20 %)**    |

**Fazit:** KI ist ein „Force Multiplier“ für Boilerplate –  
aber die „Gehirnlogik“ der Anwendung benötigt Senior Engineering.

***

## **Slide 4: Deep Dive – 10 Repräsentative Funktionen**

| Funktion                  | Paket                 | Komplexität      | Vorgeschlagener Migrationspfad      |
| ------------------------- | --------------------- | ---------------- | ----------------------------------- |
| `FK_CALC_DISTANCE`        | `PKG_APT_UTIL`        | Einfach          | Java Math Utility                   |
| `fk_getPermittedUsers`    | `PKG_APT_RECHTE`      | Mittel           | Spring Security / JPA               |
| `fk_setRevStatusDatum`    | `PKG_APT_STATUS`      | **Sehr komplex** | **Spring State Machine**            |
| `FK_ABGLEICH_STANDORTE`   | `PKG_IF_APT_IMPORT`   | **Komplex**      | **Spring Batch (Chunk Processing)** |
| `FK_NEUES_LINK`           | `PKG_IF_APT_LINK`     | Komplex          | Orchestrierter JPA-Service          |
| `PR_RECHNE_LINK_KPI`      | `PKG_APT_PERF`        | Komplex          | Java Statistics Service             |
| `FK_TRANSFER_LINK`        | `PKG_IF_APT_TRANSFER` | Komplex          | `@Transactional` Service            |
| `FK_CREATE_PARALLEL_LINK` | `PKG_IF_APT_CLIENT`   | Komplex          | MapStruct / Deep-Copy-Logik         |

***

## **Slide 5: Architektur-Gaps (Oracle vs. Java)**

*   **Zustandsmanagement:**  
    Ablösung globaler Package-Variablen (z. B. `pv_nTrans_Nr`) durch Thread-Safe-Kontexte (`ThreadLocal`, Spring Security).

*   **Transaktionsintegrität:**  
    Wechsel von manuellen DB-Commits → deklarative `@Transactional`-Grenzen.

*   **Batch-Performance:**  
    Ersatz von Oracle-`BULK COLLECT` durch **Spring Batch** für PostgreSQL.

*   **Vendor-Unabhängigkeit:**  
    Keine Abbildung von Business-Logik in PL/pgSQL; PostgreSQL bleibt **reiner Datenspeicher**.

***

## **Slide 6: Skalierbarkeit & Risiken**

*   **Horizontale Skalierbarkeit:** Stateless Services ermöglichen Cloud/Kubernetes-Betrieb.
*   **Engpässe:**  
    Die Status-Engine (`PKG_APT_STATUS`) ist kritisch – geringe Parallelisierbarkeit während der Migration.
*   **Technische Risiken:**
    *   **Logikverlust:** Implizite DB-Trigger vs. explizite Spring-/Envers-Logik.
    *   **Performance-Verlust:** Risiko durch Single-Row-Verarbeitung vs. Oracle-Cursor.
    *   **Deadlocks:** Falsch gesetzte `@Transactional`-Scopes in komplexen Schleifen.

***

## **Slide 7: Qualitätssicherung – Das „Dual-Run“-Protokoll**

Jede komplexe Migration durchläuft den **Functional Equivalence Check**:

1.  **Referenzlauf (Oracle):**  
    Ausführung der Original-PL/SQL-Funktion mit Testdaten.
2.  **Validierungslauf (Java):**  
    Ausführung des neuen Java-Services auf PostgreSQL.
3.  **Assertions:**  
    Automatischer Vergleich von:
    *   Rückgabewerten (DTOs)
    *   Seiteneffekten (DB-Zustände / History-Tabellen)
    *   Exception-Mapping (PL/SQL → Domain Exceptions)

**Benchmark:** Batch-Prozesse müssen **mindestens 20 % schnellere Durchsatzwerte** durch Chunking zeigen.

***

## **Slide 8: Empfehlungen & Nächste Schritte**

1.  **Proof-of-Concept erforderlich:**  
    Umsetzung von `PKG_APT_STATUS` (Status Engine) mittels **Spring State Machine** als Validierung des Architekturansatzes.

2.  **Spring-Batch-Blueprint:**  
    Standardisiertes Pattern für `FK_ABGLEICH_STANDORTE` (Millionen-Row-Synchronisation).

3.  **Infrastruktur:**  
    Einrichtung von **Testcontainers** (Oracle + PostgreSQL) zur kontinuierlichen Dual-Run-Validierung im CI/CD.

4.  **Team-Skalierung:**  
    Einsatz/Recruiting senioriger Java-Entwickler für das Redesign der 70 % komplexen Logik.

***
