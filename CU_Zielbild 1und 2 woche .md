# Zielbild & Roadmap: Migration Woche 1 & 2 (LinkVis) - AKTUALISIERT

Dieses Dokument fasst die Ergebnisse der ersten Woche (W1) zusammen und definiert die operativen Ziele für die detaillierte Analyse- und Pilotphase in Woche 2 (W2) auf Basis des **KI-gestützten Pattern-Ansatzes**.

---

## 1. Rückblick Woche 1: Fundament & Machbarkeitsnachweis (W1)
In Woche 1 wurde die methodische Basis für den Oracle-Exit gelegt. Durch den erfolgreichen "Dual-Run" Pilot wurde die funktionale Gleichwertigkeit bewiesen.

### Zentrale Erfolge (W1)
- **Methodik-Check:** Etablierung des **"Dual-Run" Validierungsmodells** (`md\Validierungsnachweis.md`).
- **PoC Abschluss:** Volle Transformation und mathematische Validierung der `GeoUtils` (`FK_CALC_DISTANCE`).
- **KI-Hebel:** Nachweis, dass Utility-Logik zu **90-95 %** und komplexe Logik (nach Setup) zu **~85 %** durch KI-Blueprints beschleunigt werden kann.
- **Strategie:** Wechsel vom reinen Refactoring zum **Pattern-basierten KI-Rollout**.

---

## 2. Zielbild Woche 2: Skalierung & Framework-Stabilisierung (W2)

Das Ziel der zweiten Woche ist der Übergang von der Einzelfall-Pilotierung zur **Serien-Produktion** der Migrations-Blueprints.

### 2.1 Operative Ziele (W2)
1.  **Transformation Playbook:** Finalisierung der Standard-Prompts für den Massen-Rollout.
2.  **Erweiterter Pilot (10–20 Funktionen):** Skalierung des KI-Verfahrens auf das gesamte Pilot-Set zur Validierung der Pattern-Qualität.
3.  **Golden-Master-Framework:** Implementierung eines automatisierten Test-Tools zum Vergleich von Input/Output-Daten (Oracle vs. Java).
4.  **Tool-Evaluierung:** Auswahl der optimalen KI-Modelle und IDE-Plugins für das Entwickler-Onboarding.

### 2.2 Meilensteine Woche 2
- [ ] **M2.1:** **Transformation Playbook v1.0** (Einfach, Mittel, Komplex).
- [ ] **M2.2:** **Golden-Master-Prototyp** (Automatisierter Dual-Run für Web-Services).
- [ ] **M2.3:** **Blueprint-Katalog** für 20 Kernfunktionen (Serienreife).
- [ ] **M2.4:** **Entscheidungsvorlage Tool-Stack** (KI-Modelle & Lizenzen).

---

## 3. Validierter Automatisierungs-Rahmen (KI-Hebel)

| Kategorie | Anteil (LinkVis) | Zeitersparnis durch KI | Strategie |
|:---|:---|:---|:---|
| **Einfach** | 10 % | **~90 %** | Vollautomatisierte Portierung |
| **Mittel** | 20 % | **~60-80 %** | Blueprint + JPA-Refactoring |
| **Komplex** | 70 % | **~30-70 %** | Pattern-basiertes Design (State Machine) |

---

## 4. Nächste Schritte (Start W2)

1.  **Sprint Planning (Montag):** Gemeinsame Festlegung des 20-Funktionen-Scopes (Scope Lock).
2.  **Playbook-Produktion:** Dokumentation der erfolgreichen Prompts aus Woche 1 als Standard.
3.  **Onboarding-Vorbereitung:** Erstellung der Profile für die Java-Entwickler (Start W3).

**Fazit:** Die Strategie für Woche 2 zielt darauf ab, die **"Fabrik"** für die Migration aufzubauen, damit ab Woche 3 die produktive Integration durch das Entwickler-Team erfolgen kann.
