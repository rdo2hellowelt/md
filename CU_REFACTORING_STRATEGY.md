# Strategie zur Refaktorisierung: LinkVis PL/SQL zu Spring Service Layer

Dieses Dokument definiert die verbindliche Strategie für die Migration der LinkVis-Geschäftslogik aus Oracle PL/SQL in eine moderne, serviceorientierte Java-Architektur.

## 1. Ziel-Technologiestack
- **Runtime:** Java 17+ mit Spring Boot 3.x
- **Persistenz:** Spring Data JPA (Hibernate) für Standard-CRUD; QueryDSL oder MyBatis für hochkomplexe Queries.
- **Zustandssteuerung:** **Spring State Machine** für komplexe Workflow-Logik (Status-Engine).
- **Batch-Verarbeitung:** **Spring Batch** für Cursorschleifen-Logik mit hohem Datenvolumen.
- **Validierung:** Jakarta Bean Validation (Hibernate Validator).

---

## 2. Architektonische Leitplanken

### 2.1 Vom Package zum Stateless Service
- PL/SQL-Packages werden in funktionale **Spring Services** transformiert.
- Package-Variablen (globaler State) werden durch zustandslose Service-Methoden und explizite Parameterübergabe ersetzt.
- Datenbank-Trigger zur Historisierung wandern in den Service-Layer (z. B. via Hibernate Envers oder explizite History-Services).

### 2.2 Transaktions- & Exception-Management
- **Transaktionen:** Weg von `COMMIT`/`ROLLBACK` in der DB hin zu deklarativem `@Transactional` im Java-Service-Layer.
- **Fehlerbehandlung:** PL/SQL `RAISE_APPLICATION_ERROR` und Fehler-Codes werden in eine hierarchische Struktur von **DomainExceptions** übersetzt (z. B. `StatusTransitionException`, `InsufficientRightsException`).
- **Global Handling:** Zentrale Fehlerbehandlung über `@RestControllerAdvice`.

---

## 3. Umsetzungs-Roadmap (Die 10 Kernfunktionen)

### Phase 1: Quick Wins (Utility & Security)
- **`FK_CALC_DISTANCE`**: Portierung als zustandslose `GeoUtils` Komponente. Fokus: Mathematische Korrektheit (JUnit).
- **`fk_getPermittedUsers`**: Ablösung der Bitmasken-Logik durch einen **PermissionEvaluator** in Java. Integration in Spring Security.
- **`FK_GET_USER_LINK_ID`**: Zentralisierung in einen `LinkIdService`. Nutzung von JPA-Existenzprüfungen statt Cursorn.

### Phase 2: Orchestrierung & Datenfluss
- **`FK_NEUES_LINK`**: Auflösung des massiven Inserts in koordinierte Repository-Aufrufe. Nutzung von JPA-Kaskaden statt manueller Sequenz-Steuerung.
- **`FK_CREATE_PARALLEL_LINK`**: Implementierung einer Deep-Copy Logik auf Entity-Ebene (MapStruct oder manueller Klon-Service).
- **`FK_TRANSFER_LINK`**: Orchestrierung des Datentransfers zwischen Diff-Tabellen und Kern-Entities.

### Phase 3: Komplexe Engines (Redesign)
- **`fk_setRevStatusDatum` (Status-Engine)**: 
  - **Strategie:** Vollständiges Redesign als **Spring State Machine**. 
  - **Ziel:** Ablösung der rekursiven PL/SQL-Logik durch explizite Statusübergänge und Event-Listener (z. B. E-Mail-Versand bei Statusänderung).
- **`FK_ABGLEICH_STANDORTE` & `PR_RECHNE_LINK_KPI`**:
  - **Strategie:** Implementierung als **Spring Batch Jobs**. 
  - **Ziel:** Performance-Steigerung durch Chunk-Processing und parallele Ausführung statt serieller Cursor-Schleifen.
- **`FK_IMPORT_RESERVIERUNG`**: Transformation des ETL-Prozesses in einen Java-basierten Import-Service mit Validierungslayer.

---

## 4. Validierungsstrategie
Jede migrierte Funktion muss den **"Dual-Run" Test** bestehen:
1. **Unit Tests:** Validierung der Geschäftsregeln in Isolation (Mockito).
2. **Integration Tests:** Vergleich der Datenbank-Zustände nach Ausführung (PL/SQL vs. Java) mit identischen Testdaten (Testcontainers mit PostgreSQL/Oracle).
3. **Performance-Check:** Java-Implementierung darf (bei Batch-Logik) nicht langsamer sein als die ursprüngliche PL/SQL-Prozedur.

---

## 5. KI-Unterstützung im Prozess
- **Analyse:** KI extrahiert fachliche Regeln aus den 10.000+ Zeilen PL/SQL.
- **Skelettierung:** KI generiert Entities, Repositories und Service-Skelette.
- **Test-Generierung:** KI erstellt JUnit-Testfälle basierend auf den Randbedingungen der PL/SQL-Funktionen.
- **Review:** Manuelle Prüfung jeder KI-generierten Architektur-Entscheidung durch das Team.
