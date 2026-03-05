# Validierungsnachweis: "Dual-Run" Protokoll (LinkVis Migration)

Dieses Dokument definiert die methodischen Anforderungen für den funktionalen Gleichwertigkeitsnachweis zwischen Oracle PL/SQL und dem neuen Spring Boot Service Layer.

---

## 1. Das Validierungskonzept: Der "Dual-Run"
Um die fachliche Korrektheit der Migration zu garantieren, wird für jede Kernfunktion ein **Dual-Run** durchgeführt:

1.  **Referenzlauf (PL/SQL):** Ausführung der Original-Logik in einer Oracle-Testinstanz mit definierten Test-Szenarien.
2.  **Validierungslauf (Java):** Ausführung des neuen Spring-Services gegen dieselbe Datenbasis (oder ein PostgreSQL-Äquivalent).
3.  **Abgleich:** Automatisierter oder manueller Vergleich der Rückgabewerte, Datenbank-Zustände (Side Effects) und des Exception-Verhaltens.

---

## 2. Abnahmekriterien (Definition of Done)

### 2.1 Funktionale Äquivalenz
- **Rückgabewerte:** Alle Rückgabewerte (inkl. komplexer Objekte/DTOs) müssen fachlich identisch sein.
- **Datenbank-Zustand:** Nach der Ausführung müssen alle betroffenen Tabellen (z. B. `KAP_LINK`, `KAP_LINK_HIST`) dieselben Änderungen aufweisen wie im Referenzlauf.
- **Präzision:** Mathematische Berechnungen (z. B. Distanzen in `FK_CALC_DISTANCE`) dürfen maximal um 0,001 % abweichen (Rundungsdifferenzen Java vs. Oracle).

### 2.2 Exception-Mapping
- PL/SQL-Fehlercodes (`RAISE_APPLICATION_ERROR`) müssen in die korrespondierenden Java **Domain-Exceptions** übersetzt sein.
- Ein technischer Fehler in der DB (z. B. `ConstraintViolation`) muss im Java-Service sauber abgefangen oder als strukturierte Fehlermeldung weitergegeben werden.

### 2.3 Performance-Benchmark
- Die Latenz des Java-Services darf bei Einzelaufrufen nicht signifikant höher sein als in PL/SQL.
- Batch-Prozesse (z. B. `FK_ABGLEICH_STANDORTE`) müssen durch die Nutzung von Spring Batch und Chunking eine **Performance-Steigerung** von mindestens 20 % gegenüber der seriellen Cursor-Logik aufweisen.

---

## 3. Protokoll-Vorlage (Beispiel: Geo-Berechnung)

| Testfall-ID | Szenario / Input | Erwartet (PL/SQL) | IST (Java Service) | Status |
|:---|:---|:---|:---|:---|
| **VAL-01** | Distanz Berlin -> München | `504.123 km` | `504.123 km` | ✅ OK |
| **VAL-02** | Ungültige Koord. (0,0) | `NULL` | `null` | ✅ OK |
| **VAL-03** | Rechteprüfung (Admin) | `11111` | `11111` (Full Access) | ✅ OK |
| **VAL-04** | Statusänderung (60a -> 70) | `Success + Hist-Entry` | `Success + Hist-Entry` | ✅ OK |

---

## 4. Werkzeuge für die Validierung
- **JUnit 5 & AssertJ:** Für den Vergleich komplexer Objektstrukturen.
- **Testcontainers:** Um echte Oracle- und PostgreSQL-Instanzen für Integrationstests hochzufahren.
- **DbUnit / Hibernate-L2-Cache-Prüfung:** Zur Verifizierung der Datenbank-Zustände nach einem Service-Aufruf.
- **Logging-Abgleich:** Vergleich der SLF4J-Logs mit den ursprünglichen `up_log`/`up_dbg` Einträgen aus PL/SQL.

---

## 5. Freigabeprozess
Ein Validierungsnachweis gilt erst dann als erbracht, wenn:
1. Das Dual-Run-Protokoll für alle definierten Testfälle (Positiv, Negativ, Edge-Cases) erfolgreich durchlaufen wurde.
2. Ein manuelles Code-Review der Architektur (Statelessness, Transaktionsgrenzen) durch einen Lead-Developer erfolgt ist.
3. Die Testabdeckung des migrierten Codes > 85 % beträgt.

---

## Anhang: Konkrete Nachweise
Als Referenz für die Umsetzung dieses Konzepts dient folgendes Beispiel:
- [Beispielhafter Validierungsnachweis: FK_CALC_DISTANCE](./Validierungsnachweis_Beispiel_FK_CALC_DISTANCE.md)

