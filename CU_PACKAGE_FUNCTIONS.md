# Dokumentation der PL/SQL Pakete und Funktionen

Diese Übersicht listet die Speicherorte und die technische Analyse der angefragten PL/SQL Funktionen und Prozeduren auf. Die Definitionen befinden sich unter `api-apt/src/main/resources/liquibase-ora/packages/`.

---

## Technische Analyse & Kategorisierung

### 1. PKG_IF_APT_LINK.FK_NEUES_LINK
*   **Kategorie:** **Komplex**
*   **Analyse:** Orchestriert das Anlegen eines kompletten Link-Objekts. Führt zahlreiche Validierungen durch und schreibt in über 8 verschiedene Tabellen (`KAP_LINK`, `KAP_LINK_REVISION`, `KAP_LINK_DATA`, `KAP_LINK_VARDATA`, etc.).
*   **Details:**

    *   *Oracle-Features:* Sequences (`SEQ_LNK_ID`, `SEQ_REV_ID`), `SYSDATE`, `DUAL`.
    *   *Seiteneffekte:* Sehr hoch (viele INSERTs/UPDATEs).
    *   *Automatisierungspotenzial:* Mittel (erfordert präzises Mapping der Datenbankstruktur).

### 2. PKG_IF_APT_UTIL.FK_CALC_DISTANCE
*   **Kategorie:** **Einfach**
*   **Status W1:** ✅ **Validierung abgeschlossen.** PoC erfolgreich.
*   **Analyse:** Reine mathematische Funktion zur Berechnung der Distanz zwischen Koordinaten (Haversine-Formel). Keine Datenbankzugriffe.
*   **Details:**
    *   *Automatisierungspotenzial:* **Hoch (95 %)**.
    *   *Nachweis:* Siehe [Validierungsnachweis: FK_CALC_DISTANCE](./Validierungsnachweis_Beispiel_FK_CALC_DISTANCE.md).

### 3. PKG_IF_APT_IMPORT.FK_ABGLEICH_STANDORTE
*   **Kategorie:** **Komplex**
*   **Analyse:** Ein zentraler Synchronisationsmechanismus zwischen externen Sichtdaten (`V_IF_APT_STANDORTE`) und internen Tabellen. Verwendet eine aufwendige Merge-Logik mit zwei Cursorn und bis zu 1 Mio. Iterationen.
*   **Details:**
    *   *Cursor-Logik:* Zwei explizite Cursor (`cur_SODA`, `cur_APT`) mit manuellem Fetch und Vergleich.
    *   *Oracle-Features:* `NLSSORT` für binäre Sortierung, `SYSDATE`.
    *   *Exception Handling:* Ressourcenbereinigung (Cursor schließen).
    *   *Automatisierungspotenzial:* Gering (sehr geschäftskritische und komplexe Synchronisationslogik).

### 4. PKG_IF_APT_TRANSFER.FK_TRANSFER_LINK
*   **Kategorie:** **Komplex**
*   **Analyse:** Verarbeitet Differenz-Datensätze aus `KAP_DIFF_LINK` und delegiert DML-Operationen an spezialisierte Unterfunktionen für Neuanlagen, Updates oder Löschungen.
*   **Details:**
    *   *Transaktionslogik:* Explizite `ROLLBACK` und `up_commit` Aufrufe innerhalb der Loop.
    *   *Oracle-Features:* Sequence-Nutzung, komplexe Typ-Konvertierungen.
    *   *Automatisierungspotenzial:* Mittel (Logik ist stark von der Tabellenstruktur der Diff-Tabellen abhängig).

### 5. PKG_IF_APT_INST_DEINST.FK_IMPORT_RESERVIERUNG
*   **Kategorie:** **Mittel - Komplex**
*   **Analyse:** Importiert Materialpositionen aus einer externen Pepsy-Sicht in die Installationspositionen von APT. Beinhaltet Lösch- und Insert-Logik basierend auf dem Abgleich von Reservierungsnummern.
*   **Details:**
    *   *Cursor-Logik:* Verschachtelte FOR-Loops.
    *   *Seiteneffekte:* Modifikation von Installationstabellen.
    *   *Automatisierungspotenzial:* Hoch (klare ETL-Struktur).

### 6. PKG_APT_STATUS.fk_setRevStatusDatum
*   **Kategorie:** **Sehr Komplex**
*   **Status W1:** 🧪 **KI-Pilot durchgeführt.** Strukturvorschlag (State Machine) erstellt.
*   **Analyse:** Das Herzstück der Status-Engine. Beinhaltet extrem viele fachliche Prüfungen (EOM, Pflichtfelder, Antennenabbau), triggert rekursiv Folgestatus, versendet E-Mails und schreibt Historien.
*   **Details:**
    *   *Automatisierungspotenzial:* **Mittel (60-70 %)** nach Initial-Setup des State-Machine-Patterns.
    *   *Pilot-Doku:* Siehe [KI-Transformation Pilot](./KI_TRANSFORMATION_PILOT_DOKUMENTATION.md).

### 7. PKG_APT_RECHTE.fk_getPermittedUsers
*   **Kategorie:** **Mittel**
*   **Status W1:** 🧪 **KI-Pilot durchgeführt.** Transformation pilotiert.
*   **Analyse:** Ermittelt eine Liste berechtigter Benutzer als CLOB. Prüft Berechtigungen gegen Masken und Transaktions-IDs.
*   **Details:**
    *   *Automatisierungspotenzial:* **Hoch (80 %)** durch Standard-JPA-Mappings.
    *   *Pilot-Doku:* Siehe [KI-Transformation Pilot](./KI_TRANSFORMATION_PILOT_DOKUMENTATION.md).

### 8. PKG_APT_PERF.PR_RECHNE_LINK_KPI
*   **Kategorie:** **Sehr Komplex**
*   **Analyse:** Ein Batch-Job für Performance-Analysen. Berechnet Mittelwerte, Standardabweichungen und Noten aus Wirknetzdaten über 14 Tage.
*   **Details:**
    *   *Mathematik:* Komplexe statistische Berechnungen (`POWER`, `SQRT`).
    *   *Cursor-Logik:* Mehrfach verschachtelte Loops über Links und Zeitreihen.
    *   *Seiteneffekte:* Schreibt Performance-Historien und berechnet Scores.
    *   *Automatisierungspotenzial:* Mittel (Algorithmus ist portierbar, Datenzugriff jedoch massiv).

### 9. PKG_IF_APT_CLIENT.FK_GET_USER_LINK_ID
*   **Kategorie:** **Mittel**
*   **Analyse:** Prüft die Verfügbarkeit von Link-IDs oder generiert die nächste freie ID in einem bestimmten Nummernbereich.
*   **Details:**
    *   *Logik:* String-Manipulation und Suche in Bestandsdaten.
    *   *Automatisierungspotenzial:* Hoch.

### 10. PKG_IF_APT_CLIENT.FK_CREATE_PARALLEL_LINK
*   **Kategorie:** **Komplex**
*   **Analyse:** Orchestrierungsfunktion zum Duplizieren eines Links (z.B. für XPIC). Kopiert Daten aus Link-, Revisions- und Endstellen-Tabellen.
*   **Details:**
    *   *Logik:* Sequentielles Kopieren von Attributen und Neuanlage über `FK_CREATE_LINK`.
    *   *Automatisierungspotenzial:* Mittel (stark abhängig von der Datenintegrität).

---

## Zusammenfassung Automatisierungspotenzial (Validiert nach W1)

| Kategorie | Anzahl | Typische Merkmale | Automatisierungspotenzial (KI-gestützt) |
|:---|:---|:---|:---|
| **Einfach** | 1 | Reine Mathematik, keine DB-Zugriffe | **Hoch (90-95 %)** |
| **Mittel** | 2 | Orchestrierung, Security, ID-Gen | **Hoch (80 %)** |
| **Komplex** | 7 | Status-Engine, Sync, Multi-Table | **Mittel-Hoch (60-80 %)** |

### Fazit der Analyse (Stand: Ende W1)
Die initialen Schätzungen (35-45 %) wurden durch den **KI-gestützten Pattern-Ansatz** massiv nach oben korrigiert. 
Durch die Kombination von standardisierten Java-Templates (z.B. Spring State Machine, JPA-Repositories) und KI-Orchestrierung ist ein **Gesamt-Automatisierungsgrad von 70-80 %** über das gesamte Portfolio realistisch. 
Die Migration ist nun primär ein **Review- und Integrationsprozess** der von der KI gelieferten Blueprints.
