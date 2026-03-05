# Präsentation: KI-Pilot PL/SQL-Migration LinkVis (Woche 1)

*Statusbericht und Entscheidungsvorlage für das Management*

---

## Slide 1: Mission & Zielsetzung
- **Strategisches Ziel:** Ablösung von Oracle PL/SQL (Vendor Lock-in) hin zu **Java 17+ / Spring Boot 3.x / PostgreSQL**.
- **KI-Einsatz:** Nutzung generativer KI als "Co-Developer" zur Analyse von 50.000+ Zeilen Code.
- **Woche 1 Fokus:** Realitätscheck an 10 Kernfunktionen (u.a. `PKG_APT_STATUS`, `PKG_APT_RECHTE`).

---

## Slide 2: Ergebnisse der Komplexitätsanalyse
Die Analyse zeigt eine hohe Dichte an geschäftskritischer Logik:
- **Einfach (10 %):** Reine Utility/Math-Funktionen (z.B. `FK_CALC_DISTANCE`).
- **Mittel (20 %):** Berechtigungen und ID-Orchestrierung.
- **Komplex (70 %):** Kern-Engines (Status-Management, Synchronisation, Multi-Table-Inserts).

---

## Slide 3: Realistischer Automatisierungsgrad
Entgegen allgemeiner Marktprognosen (50 %+) liegt die Erwartung für LinkVis bei:
- **35 – 45 % Gesamtautomatisierung**
- **Warum?** 
  - Hoher Anteil komplexer State-Logic.
  - Tief verzahnte Oracle-Features (Trigger-Kaskaden).
  - Notwendigkeit eines Architektur-Redesigns (Statelessness).

---

## Slide 4: Die Zielarchitektur (W1-Konsens)
- **Vollständige Entkopplung:** Business-Logik wandert zu 100 % in Java-Services.
- **Zustandssteuerung:** Einsatz von **Spring State Machine** für die Status-Engine.
- **Massenverarbeitung:** Einsatz von **Spring Batch** zur Performance-Optimierung (Cursor-Ersatz).
- **Datenbank:** PostgreSQL als reiner Datenspeicher (keine Logik in der DB).

---

## Slide 5: Validierung & Qualitätssicherung
- **Das "Dual-Run" Verfahren:**
  - Identische Testdaten für PL/SQL und Java.
  - Automatisierter Abgleich der Ergebnisse & DB-Zustände.
- **Definition of Done:** 
  - JUnit-Testabdeckung > 85 %.
  - Fachliche Gleichwertigkeit nachgewiesen.
  - Manuelles Architektur-Review bestanden.

---

## Slide 6: Roadmap Woche 2+ (Operative Phase)
- **Phase 1 (W2): Quick Wins & Framework**
  - Finalisierung der Java-Utility-Infrastruktur (`GeoUtils`, `PermissionService`).
  - Aufbau der JPA-Entity-Basis.
- **Phase 2 (W2-W3): Core-Logik Design**
  - Technisches Design der **State Machine** für die Status-Engine.
  - Implementierung der ersten komplexen Multi-Table-Services.
- **Phase 3: Rollout**
  - Paket-für-Paket Migration basierend auf etablierten Blueprints.

---

## Slide 7: Fazit & Management-Empfehlung
- **KI ist ein massiver Beschleuniger** für Analyse und Skelett-Code.
- **Migration = Refactoring:** Wir modernisieren nicht nur den Code, sondern die gesamte Architektur für die Cloud-Zukunft.
- **Empfehlung:** Fortführung des Projekts in Woche 2 mit Fokus auf die **Quick Wins** und das **State-Machine-Prototyping**.
