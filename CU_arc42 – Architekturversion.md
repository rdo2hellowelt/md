# arc42 – Architekturversion: LinkVis PL/SQL Migration (W1)

Dieses Dokument beschreibt die Zielarchitektur für die Migration der LinkVis Oracle PL/SQL-Logik in eine moderne Java-Umgebung.

## 1. Einleitung und Ziele
Die bestehende Oracle-PL/SQL-Logik wird in eine serviceorientierte Spring-Boot-Architektur überführt.
- **Ziel:** Ablösung des Oracle Vendor Lock-ins, Migration nach PostgreSQL.
- **Fokus:** Verlagerung der Business-Logik aus der DB in den Applikationslayer.
- **Ergebnis W1:** Realistischer Automatisierungsgrad von **35–45 %** aufgrund hoher fachlicher Komplexität.

## 2. Randbedingungen
- **Technik:** Java 17+, Spring Boot 3.x, PostgreSQL.
- **Architektur:** Stateless Services, Three-Tier-Layering.
- **Validierung:** "Dual-Run" Vergleich (PL/SQL vs. Java) ist zwingend.

## 3. Kontextabgrenzung
- **Eingang:** 10 analysierte Kernpakete (u.a. `PKG_APT_STATUS`, `PKG_APT_RECHTE`).
- **Ausgang:** Spring Boot Microservices oder modulare Monolithen.

## 4. Lösungsstrategie
- **Transformation:** 
  - Utilities → Java Math/String API.
  - CRUD → Spring Data JPA.
  - Status-Engine → **Spring State Machine**.
  - Batch/Cursor-Schleifen → **Spring Batch**.
- **Transaktionen:** Steuerung via `@Transactional` (Spring), Eliminierung von DB-internen Commits.
- **Logging:** SLF4J/Logback statt `DBMS_OUTPUT`.

## 5. Bausteinsicht (Zielstruktur)
- **API Layer:** REST-Controller (Spring MVC).
- **Application Layer:** Orchestrierungs-Services, Transaktionsgrenzen.
- **Domain Layer:** Business-Logik, Entities, State-Machine-Definitionen.
- **Infrastructure Layer:** Repositories (JPA/QueryDSL), Batch-Konfigurationen.

## 6. Architekturentscheidungen (W1)
1. **Full Java Logic:** Keine Logik verbleibt in der DB (kein PL/pgSQL für Business Rules).
2. **Statelessness:** Services halten keinen internen State (Ersatz für Package-Variablen).
3. **Explicit History:** Historisierung erfolgt über Java-Services statt über DB-Trigger.
4. **KI-Rolle:** KI agiert als "Übersetzungs-Assistent", die Architekturverantwortung liegt beim Menschen.

## 7. Risiken und Gegenmaßnahmen
- **Trigger-Kaskaden:** Werden durch explizite Event-Handler in Java ersetzt.
- **Performance (Batch):** Chunk-Processing in Spring Batch kompensiert den Wegfall von DB-nahen Cursorn.
- **Seiteneffekte:** Gründliche Analyse der globalen Package-Variablen vor der Migration.

## 8. Migrationsprozess
1. **Analyse:** Komplexitätscheck der PL/SQL-Funktion.
2. **Skelettierung:** KI generiert Service- und Repository-Gerüste.
3. **Implementierung:** Manuelles Refactoring komplexer Logik (z.B. State Machine).
4. **Validierung:** Dual-Run Test gegen Testdaten.
