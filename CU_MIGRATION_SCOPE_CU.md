# MIGRATION_SCOPE: Zielbild & Leitplanken

Dieses Dokument definiert den Scope, die Zielarchitektur und die technischen Leitplanken für die KI-gestützte Migration der LinkVis PL/SQL-Logik.

## 1. Zielarchitektur (Service Layer)
- **Migration:** Vollständige Verlagerung der Geschäftslogik aus Oracle PL/SQL in einen **Spring Boot Service Layer** (Java 17+).
- **Zustandslosigkeit:** Umwandlung von zustandsbehafteten Packages in **Stateless Services**.
- **Persistenz:** Nutzung von Spring Data JPA oder MyBatis; SQL bleibt transparent im Anwendungscode (keine Logik in der DB).

## 2. Transaktionsmodell & Steuerung
- **Applikationslayer:** Die Transaktionssteuerung erfolgt ausschließlich über `@Transactional` auf Service-Ebene.
- **Eliminierung:** `COMMIT` und `ROLLBACK` innerhalb der Datenbanklogik sowie `Autonomous Transactions` werden entfernt und durch Applikationslogik oder asynchrone Events ersetzt.

## 3. Logging- & Exception-Konzept
- **Frameworks:** Zentrales Logging via **SLF4J / Logback**.
- **Exceptions:** Fachliche Fehler werden als spezifische Domain-Exceptions abgebildet; technisches Exception-Handling erfolgt global (z.B. `@ControllerAdvice`).
- **Ablösung:** Ersatz von `DBMS_OUTPUT` und `RAISE_APPLICATION_ERROR` durch strukturiertes Logging und Exception-Typen.

## 4. Rahmenbedingungen (Oracle Exit)
- **Ziel-DB:** Strategischer Wechsel von Oracle zu **PostgreSQL**.
- **Minimal-Logik:** Die Datenbank dient primär als Datenspeicher; PL/pgSQL wird nur für rein technische Hilfsfunktionen eingesetzt.

---

## 5. Migration To-Do: Analyse & Paket-Übersicht (W2)

Die folgenden 10 Pakete werden im nächsten Schritt detailliert analysiert, um die Komplexität und die Abhängigkeiten für die Java-Migration zu bestimmen:

- [ ] **`PKG_APT_PERF`**: Performance-Analysen und KPI-Berechnungen.
- [ ] **`PKG_APT_RECHTE`**: Berechtigungssteuerung und Benutzerrechte.
- [ ] **`PKG_APT_STATUS`**: Zentrale Status-Engine für Link-Revisionen.
- [ ] **`PKG_APT_UTIL`**: Hilfsfunktionen und Logging.
- [ ] **`PKG_IF_APT_CLIENT`**: Client-Schnittstellen und Link-ID-Generierung.
- [ ] **`PKG_IF_APT_IMPORT`**: Datenabgleich und Synchronisation mit externen Systemen.
- [ ] **`PKG_IF_APT_INST_DEINST`**: Verwaltung von Installations- und Deinstallationsprozessen.
- [ ] **`PKG_IF_APT_LINK`**: Orchestrierung der Link-Objekt-Erstellung.
- [ ] **`PKG_IF_APT_TRANSFER`**: Verarbeitung von Differenzdatensätzen.
- [ ] **`PKG_IF_APT_USER`**: Benutzerspezifische Schnittstellenlogik.

---

## 6. KI-gestütztes Vorgehen
- **KI als Co-Developer:** Unterstützung bei Analyse, Code-Skelettierung und Testgenerierung.
- **Human-in-the-Loop:** Jede KI-generierte Komponente unterliegt einem manuellen Review auf Architekturkonformität und fachliche Korrektheit.
