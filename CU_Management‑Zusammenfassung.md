# Management-Zusammenfassung: PL/SQL-Migration LinkVis (W1)

## 1. Ausgangslage & Zielsetzung
Das Projekt LinkVis strebt den strategischen **Oracle-Exit** und die Migration der Geschäftslogik in eine moderne, herstellerunabhängige Cloud-Native-Architektur an.
- **Ziel:** Ablösung von 10 Kern-PL/SQL-Paketen (ca. 80.000+ Zeilen Code).
- **Zielstack:** Java 17+, Spring Boot 3.x, PostgreSQL.
- **KI-Rolle:** Einsatz von KI als "Co-Developer" zur Beschleunigung von Analyse und Code-Skelettierung.

## 2. Ergebnisse der Pilot-Phase (Woche 1)
Die detaillierte Analyse von 10 repräsentativen Funktionen zeigt ein klares Bild der Komplexitätsverteilung:

### Automatisierungsgrad & Komplexität
- **Kategorie "Einfach" (10 %):** Mathematische Hilfsfunktionen lassen sich zu **90 %** automatisiert portieren.
- **Kategorie "Mittel" (20 %):** Berechtigungen und ID-Generierung erreichen ca. **50 %** Automatisierung.
- **Kategorie "Komplex" (70 %):** Kernlogik (Status-Engine, Synchronisation) erfordert ein **manuelles Redesign** (Automatisierung ca. **15–25 %**).
- **Gesamt-Erwartung:** Ein realistischer Automatisierungsgrad liegt bei **35–45 %**.

## 3. Strategische Weichenstellungen
- **Vom monolithischen Paket zum Stateless Service:** Die Logik wird vollständig in Java-Services überführt, um Testbarkeit und Skalierbarkeit zu gewährleisten.
- **Einsatz spezialisierter Frameworks:**
  - **Spring State Machine:** Zur Ablösung der hochkomplexen PL/SQL-Statuslogik.
  - **Spring Batch:** Zur Performance-Optimierung massiver Datenabgleiche (Cursor-Ersatz).
- **Qualitätssicherung:** Einführung des **"Dual-Run" Verfahrens**, bei dem Alt- und Neu-Code mit identischen Testdaten verglichen werden.

## 4. Risiken & Herausforderungen
- **Proprietäre Features:** Oracle-spezifische Funktionen (Trigger-Kaskaden, autonome Transaktionen) können nicht 1:1 übersetzt werden und verursachen manuellen Redesign-Aufwand.
- **Implizite Logik:** Versteckte Seiteneffekte in PL/SQL-Paketen erfordern tiefgehende fachliche Reviews.

## 5. Fazit & Empfehlung
Die Migration ist **kein rein technisches "Umkopieren"**, sondern ein wertvolles **Architektur-Refactoring**.
- **Empfehlung:** Fortführung des Projekts mit Fokus auf die **"Quick Wins"** (Phase 1), um das Framework zu stabilisieren, gefolgt vom Redesign der Status-Engine (Phase 2).
- **KI-Nutzen:** Die KI verkürzt die Analysephase der massiven PL/SQL-Bestände erheblich und liefert hochwertige Code-Grundgerüste.
