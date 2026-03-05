## 1. Management Summary: Ausgangslage & Zielsetzung

**Aktuelle Lage:**
Die Geschäftslogik von LinkVis ist in **Oracle PL/SQL-Packages** (ca. 50.000 Zeilen) innerhalb der Datenbank festgeschrieben. Dies führt zu einer starken Abhängigkeit von Oracle ("Vendor Lock-in") und erschwert die Wartung sowie die Cloud-Fähigkeit.

**Das Zielbild:**
Wir transformieren die Logik in eine moderne, herstellerunabhängige **Java-Architektur (Spring Boot)**. 
- **Business-Logik:** Wandert vollständig in den **Application Layer** (Java-Services).
- **Datenbank:** Wechsel zu **PostgreSQL**. Die Datenbank dient künftig nur noch als effizienter Datenspeicher, nicht mehr als Träger von Geschäftsregeln.

---

## 2. Realistische Erwartung: Automatisierung & Komplexität

Die Analyse der ersten 10 Kernpakete zeigt, dass die Migration kein reines "Übersetzen", sondern ein **Architektur-Refactoring** ist:

- **Einfach (10 %):** Mathematische Hilfsfunktionen (z.B. Distanzberechnung) lassen sich zu **90 %** automatisiert portieren.
- **Mittel (20 %):** Berechtigungen und ID-Generierung erreichen ca. **50 %** Automatisierung.
- **Komplex (70 %):** Kernlogik (Status-Engine, Synchronisation) erfordert ein **manuelles Redesign** (Automatisierung ca. **15–25 %**).
- **Gesamt-Erwartung:** Ein realistischer Automatisierungsgrad liegt bei **35–45 %**.

---

## 3. Die neue Architektur (Schichtenmodell)

Wir unterteilen den Java-Code in drei klare Schichten (Layers):

1.  **Domain Layer (Kern):** Beinhaltet die fachlichen Objekte (z.B. `LinkEntity` für Funkstrecken).
2.  **Application Layer (Steuerung):** Services koordinieren die fachlichen Abläufe und Transaktionen.
3.  **Infrastructure Layer (Technik):** Repositories führen die Schreib-/Lesezugriffe auf PostgreSQL aus.

---

## 4. Zentrale Leitplanken & Strategische Weichenstellungen

- **Logik-Hoheit:** Die Geschäftslogik liegt im **Service Layer**, niemals in der Datenbank.
- **Transaktions-Kontrolle:** Steuerung ausschließlich über **`@Transactional`** in Java. Keine manuellen Commits in der DB.
- **Spezialisierte Frameworks:** 
  - **Spring State Machine:** Zur Ablösung der hochkomplexen PL/SQL-Statuslogik.
  - **Spring Batch:** Zur Performance-Optimierung massiver Datenabgleiche.
- **Oracle-Freeze:** Neue fachliche Anforderungen werden direkt im Zielbild (Java) implementiert.

---

## 5. Die Brücke: Beispiel "Neue Funkstrecke" (`FK_NEUES_LINK`)

- **Heute (PL/SQL):** Eine "Black Box" in der Datenbank schreibt in 8 Tabellen. Fehler sind schwer nachvollziehbar.
- **Morgen (Java):** Ein koordinierter Service führt den Prozess Schritt für Schritt aus. Durch `@Transactional` ist absolute Datensicherheit garantiert (alles oder nichts). Fehler werfen klare **Exceptions**.

---

## 6. Fehlerbehandlung (Exceptions) & Protokollierung (Logging)

- **Fachlich:** `StatusTransitionException` (Zweck: Verhindert ungültige Statusänderungen). 
  *API-Antwort:* `400 Bad Request` – "Statusänderung nicht zulässig."
- **Technisch:** `DatabaseConnectionException` (Zweck: Infrastrukturprobleme). 
  *API-Antwort:* `503 Service Unavailable`.

**Logging:** **INFO** für Geschäftsvorfälle, **ERROR** für kritische Fehler, **DEBUG** für die Entwicklung.

---

## 7. Qualitätssicherung & KI-Nutzen

**Der "Dual-Run":** Wir vergleichen Alt- und Neu-Code mit identischen Testdaten, um fachliche Identität sicherzustellen.

**KI als Beschleuniger:**
- **Analyse:** Schnelles Erfassen komplexer Logik-Strukturen.
- **Code-Skelettierung:** Generierung von Routine-Code (Entities, Repositories).
- **SQL-Portierung:** Unterstützung beim Umschreiben nach PostgreSQL.

**Wichtig:** Jedes KI-Ergebnis wird zwingend durch einen **Senior-Entwickler manuell geprüft**.

**Antwort**
Ja — **man kann das so lassen**, *aber* ich kann dir ein paar gezielte Verbesserungsvorschläge geben, damit es noch klarer, präziser und geschliffener wirkt. Dein Text ist bereits sehr strukturiert, inhaltlich stimmig und professionell. Mit kleinen sprachlichen und stilistischen Optimierungen wird er noch überzeugender.

Hier ist eine **optimierte Version** mit minimalen Anpassungen (sprachlich, stilistisch, klarer Fokus):

---

# Zielbild & Strategie: Migration LinkVis PL/SQL → Spring Boot

## 1. Management Summary: Ausgangslage & Zielsetzung

**Aktuelle Lage:**  
Die Geschäftslogik von LinkVis ist in umfangreichen **Oracle-PL/SQL-Packages** (ca. 50.000 Zeilen) in der Datenbank implementiert. Dies führt zu einem **starken Vendor Lock-in**, erschwert die Wartbarkeit und verhindert eine moderne Cloud-Architektur.

**Zielbild:**  
Die Geschäftslogik wird vollständig in eine herstellerunabhängige **Java-/Spring‑Boot-Architektur** überführt.

- **Business-Logik:** wandert in den **Application Layer** (Java-Services).  
- **Datenbank:** Wechsel zu **PostgreSQL**, künftig ausschließlich als performanter Datenspeicher – **ohne fachliche Logik**.

---

## 2. Realistische Erwartung: Automatisierung & Komplexität

Die Analyse der ersten 10 Kernpakete zeigt: Die Migration ist kein einfaches „Portieren“, sondern ein **Architektur-Refactoring**.

- **Einfach (10 %):** Mathematische Hilfsmethoden (z. B. Distanzberechnungen) → **bis zu 90 %** automatisierbar.  
- **Mittel (20 %):** Berechtigungen, ID-Generierung → Automatisierungsgrad ca. **50 %**.  
- **Komplex (70 %):** Kernlogik (Status-Engine, Synchronisation) → **manuelles Redesign** notwendig (**15–25 %** automatisierbar).  
- **Gesamtfazit:** Ein realistischer Automatisierungsgrad liegt zwischen **35–45 %**.

---

## 3. Zielarchitektur (Drei-Schichten-Modell)

Der Java-Code wird in klare Schichten unterteilt:

1. **Domain Layer:** Fachliche Kernobjekte (z. B. `LinkEntity` für Funkstrecken).  
2. **Application Layer:** Orchestrierung der Prozesse und Transaktionen (Services).  
3. **Infrastructure Layer:** Persistenz- und technische Schnittstellen (Repositories, DB‑Zugriffe).

---

## 4. Leitplanken & Strategische Weichenstellungen

- **Logik-Hoheit:** Die Geschäftsregeln liegen **ausschließlich im Java-Service Layer**.  
- **Transaktionen:** Steuerung über **`@Transactional`**. Keine DB-seitigen Commits.  
- **Frameworks für Speziallogik:**  
  - **Spring State Machine** für komplexe Statussteuerung.  
  - **Spring Batch** für performante Datenabgleiche.  
- **Oracle-Freeze:** Neue Anforderungen werden **nur noch im Zielbild** umgesetzt.

---

## 5. Beispiel „Neue Funkstrecke“ (`FK_NEUES_LINK`)

- **Heute (PL/SQL):** Eine undurchsichtige DB-Blackbox beschreibt 8 Tabellen in einem Schritt – Fehleranalysen sind schwierig.  
- **Morgen (Java):** Ein klar strukturierter Service führt die Schritte sequenziell aus.  
  - Transaktionssteuerung über `@Transactional` → **atomare Verarbeitung: alles oder nichts**  
  - Transparente **Exceptions** bei Fehlern

---

## 6. Fehlerbehandlung & Logging

- **Fachliche Fehler:**  
  `StatusTransitionException`  
  → *API:* `400 Bad Request` – „Statusänderung nicht zulässig“.

- **Technische Fehler:**  
  `DatabaseConnectionException`  
  → *API:* `503 Service Unavailable`.

**Logging-Konzept:**  
- **INFO:** Geschäftsvorfälle  
- **ERROR:** Kritische Fehler  
- **DEBUG:** Entwicklungszwecke

---

## 7. Qualitätssicherung & KI-Unterstützung

**Dual-Run:**  
Alt- und Neubestand werden mit identischen Testdaten verglichen, um vollständige fachliche Gleichheit sicherzustellen.

**KI als Beschleuniger:**

- Analyse komplexer PL/SQL-Strukturen  
- Generierung von Code-Skeletten (Entities, Repositories)  
- Unterstützung beim SQL‑Rewrite nach PostgreSQL  

**Wichtig:** Jede KI-Ausgabe wird von einem **Senior Developer geprüft und freigegeben**.

---
