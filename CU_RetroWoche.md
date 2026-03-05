# **Retrospektive Woche 1 & Agile Planung Woche 2**

Dieses Dokument dient der Dokumentation der Ergebnisse von Woche 1 (W1) sowie der strukturierten Vorbereitung und Selbstorganisation für die Woche 2 (W2) gemäß dem agilen Vorgehensmodell.

---

## **TEIL A: Retrospektive Woche 1 (W1)**

### **1. Ziel und erwartete Ergebnisse (W1)**
**Übergeordnetes Ziel:**  
Schaffung der methodischen und technischen Entscheidungsgrundlagen für die Migration. Nachweis der Machbarkeit (PoC) durch KI-gestützte Transformation und Definition eines belastbaren Validierungsansatzes.

**Erreichte Ergebnisse:**
- **Validierungskonzept:** Definition des "Dual-Run" Verfahrens (`md\Validierungsnachweis.md`).
- **Funktionaler Nachweis:** Erfolgreiche Pilot-Validierung der Funktion `FK_CALC_DISTANCE` (`md\Validierungsnachweis_Beispiel_FK_CALC_DISTANCE.md`).
- **KI-Pilot:** Transformation von 3 Funktionen unterschiedlicher Komplexität inkl. Prompt-Strategie (`md\KI_TRANSFORMATION_PILOT_DOKUMENTATION.md`).
- **Entscheidungsvorlage:** Konsolidierte Präsentation der Ergebnisse für Technik und Management (`md\ERGEBNIS_PRÄSENTATION_W1_KI_PILOT.md`).

### **2. Eigene Planung W1 (V-Plan)**
**Phasen-Modell:**
1. **Analyse & Extraktion:** Sichten der 10 Pilot-Funktionen im PL/SQL-Source (PKG_IF_APT_*).
2. **Methodik-Design:** Entwurf der Zielarchitektur (Spring Boot) und des Validierungswegs.
3. **Durchführung Pilot:** KI-Transformation und manueller Code-Abgleich (Simulation).
4. **Dokumentation:** Erstellung der Prozessbeschreibungen und der Ergebnispräsentation.

### **3. Abhängigkeiten (W1)**
- **Informationen:** Zugriff auf die Oracle-Packages (Liquibase-SQL-Files) war essenziell.
- **Freigaben:** Klärung der Zielarchitektur (Java 21 / Spring Boot 3) durch bestehende Architektur-Guidelines.
- **Rollen:** Input durch Christoph für die fachliche Einordnung der Komplexität.

### **4. Kritische Punkte (W1)**
- **Mathematische Präzision:** Oracle `NUMBER` vs. Java `Double` (Rundungsdifferenzen bei Haversine).
- **Scope-Abgrenzung:** Fokus auf funktionale Logik, Ausklammern der physischen DB-Migration (PostgreSQL-Setup).
- **KI-Qualität:** Sicherstellung, dass die KI keine Oracle-spezifischen Built-ins (wie `BITAND`) blind übernimmt, sondern Java-Äquivalente nutzt.

### **5. Qualitätskriterien (W1)**
- **Vollständigkeit:** Jede Pilot-Funktion hat eine Ziel-Struktur in Java und eine Differenzanalyse.
- **Fehlertoleranz:** Mathematische Funktionen dürfen max. **0,001 % Abweichung** zum PL/SQL-Referenzwert aufweisen.
- **Konsistenz:** Alle Prompts folgen dem definierten Zielbild (Stateless Services, SLF4J-Logging).

---

## **TEIL B: Agile Planung Woche 2 (W2)**

In Woche 2 wechseln wir in den **"Serien-Modus"** und stabilisieren das Framework.

### **1. Sprint Backlog W2 (Scope Lock)**
Folgende Aufgaben sind für W2 fest vereinbart und ab Montag-Mittag eingefroren:

1.  **W2 - Transformation Playbook finalisieren:** Standardisierung der Prompts und Workflows.
2.  **W2 - Erweiterter KI-Pilot (10–20 Funktionen):** Skalierung der Transformation auf das gesamte Pilot-Set.
3.  **W2 - Golden-Master-Testframework implementieren:** Automatisierter Vergleich von Input/Output-Daten.
4.  **W2 - Tool-Evaluierung & Entscheidungsvorlage:** Vergleich von KI-Modellen und IDE-Plugins.
5.  **W2 - Serienmigration vorbereiten:** Infrastruktur-Check für den Massen-Rollout.

### **2. Wochenplan W2 (Rhythmus)**

| Tag | Fokus | Key Task |
|:---|:---|:---|
| **Montag** | **Planning & Playbook** | Sprint Planning; Finalisierung des Playbooks. |
| **Dienstag** | **Massen-Pilot (A)** | Transformation der ersten 10 Funktionen; Testframework-Setup. |
| **Mittwoch** | **Massen-Pilot (B)** | Transformation der restlichen 10 Funktionen; Golden-Master-Checks. |
| **Donnerstag**| **Evaluierung** | Tool-Vergleich; Erstellung der Entscheidungsvorlage. |
| **Freitag** | **Review & Retro** | Demo der 20 Funktionen; Abnahme durch Christoph; Vorbereitung W3. |

---

## **3. Ressourcen-Planung & Rollenteilung**

Für einen erfolgreichen Projektdurchlauf ist eine klare zeitliche Trennung zwischen **Blueprint-Erstellung** und **Live-Implementierung** notwendig:

| Phase | Akteur | Fokus | Status |
|:---|:---|:---|:---|
| **Woche 2** | **KI-Agent (Rene/Gemini)** | **Theoretische Skalierung:** Erstellung von 10–20 Code-Blueprints, Finalisierung des Playbooks und Tool-Evaluierung. | **Theoretisch / Blueprint** |
| **Woche 3+** | **Menschliche Entwickler** | **Live-Integration:** Einspielen des Codes in die IDE, Kompilierung, Anbindung an die Test-Datenbanken und Durchführung der Dual-Runs. | **Praktisch / Live-Run** |

**Empfehlung:** Das Onboarding des Entwickler-Teams sollte gegen Ende von Woche 2 vorbereitet werden, damit ab Woche 3 die „Baupläne“ aus dem Playbook in lauffähigen Code überführt werden können.

---

## **4. Selbstorganisation & "Tick-Modus"**


- **Proaktivität:** Ich arbeite die Aufgaben autonom ab, melde aber Blockaden (z.B. fehlender Zugriff auf komplexe Unter-Packages) sofort via O-Ton.
- **Scope-Disziplin:** Neue Anforderungen unter der Woche werden im Backlog für W3 geparkt, um die Ziele von W2 nicht zu gefährden.
- **Qualität vor Geschwindigkeit:** Eine Funktion gilt erst als "Done", wenn der Golden-Master-Test erfolgreich war.

---
**Freigabe durch Christoph erforderlich:**  
[ ] Ziele W1 akzeptiert  
[ ] Scope W2 (5 Kernaufgaben) bestätigt
