# KI-Transformation Pilot: Dokumentation (LinkVis Migration)

> [!IMPORTANT]
> **METHODISCHER HINWEIS:**
> Diese Dokumentation stellt eine **theoretische Pilotierung** dar. Der transformierte Java-Code dient als **strukturelles Zielbild (Blueprint)**. 
> Vor der Übernahme in die produktive Codebase muss durch einen Entwickler folgendes sichergestellt werden:
> 1. **Kompilierbarkeit:** Prüfung gegen die tatsächlichen Abhängigkeiten im `api-apt` / `api-dao` Projekt (Spring Boot Version, Bibliotheken).
> 2. **Integration:** Einbindung in die bestehende Package-Struktur und Konfiguration (JPA-Entity-Mappings).
> 3. **Funktionale Prüfung:** Durchführung der im [Validierungskonzept](./Validierungsnachweis.md) definierten Dual-Run-Tests.
> 4. **Review:** Manueller Check der KI-generierten Logik auf Sicherheitsaspekte (z.B. SQL-Injection in dynamischen Abfragen).

Dieses Dokument beschreibt die Ergebnisse der KI-gestützten Transformation von drei repräsentativen Funktionen aus Oracle PL/SQL in die Zielarchitektur (Java 21 / Spring Boot 3.x).


---

## 1. Übersicht der Pilot-Funktionen

| Funktion | Komplexität | Paket | Fokus der Transformation |
|:---|:---|:---|:---|
| `FK_CALC_DISTANCE` | Einfach | `PKG_APT_UTIL` | Mathematische Korrektheit & Utility-Service |
| `fk_getPermittedUsers` | Mittel | `PKG_APT_RECHTE` | SQL-zu-JPA Umwandlung & Rechteprüfung |
| `fk_setRevStatusDatum` | Komplex | `PKG_APT_STATUS` | Geschäftsregeln & Status-Engine |

---

## 2. Transformation: FK_CALC_DISTANCE (Einfach)

### Original (PL/SQL)
```sql
function FK_CALC_DISTANCE(i_nLngA, i_nLatA, i_nLngB, i_nLatB) return number is
    PI constant number := 3.14159265358979;
    DURCHM constant number := 6378.388;
    -- ... Haversine Formel ...
begin
    v_nDist := Sin(v_nLatARad) * Sin(v_nLatBRad) + Cos(Abs(v_nLngARad - v_nLngBRad)) * Cos(v_nLatARad) * Cos(v_nLatBRad);
    -- ... Rundung & Rückgabe ...
end;
```

### KI-Prompt (Strategie)
"Konvertiere die folgende PL/SQL-Funktion zur Distanzberechnung in eine zustandslose Spring Boot `@Component` namens `GeoUtils`. Nutze `java.lang.Math` und implementiere JUnit 5 Tests zur Validierung."

### Transformierter Java-Code (Zielbild)
```java
@Component
public class GeoUtils {
    private static final double EARTH_RADIUS = 6378.388;

    public Double calculateDistance(Double lngA, Double latA, Double lngB, Double latB) {
        if (lngA == null || latA == null || lngB == null || latB == null) return null;
        
        double latARad = Math.toRadians(latA);
        double lngARad = Math.toRadians(lngA);
        // ... Logik ...
        return distance;
    }
}
```

### Abweichungsanalyse
- **Datentypen:** `NUMBER` wurde zu `Double`.
- **Methodik:** `Math.toRadians()` ersetzt manuelle Multiplikation mit PI/180.
- **Handling:** NULL-Prüfung wurde explizit als Java-Standard implementiert.

---

## 3. Transformation: fk_getPermittedUsers (Mittel)

### Original (PL/SQL)
- Iteration über `ADM_PERSON`.
- Verschachtelter Loop über `ADM_BERECHTIGUNGEN`.
- Nutzung von `BITAND` zur Maskenprüfung.
- Rückgabe als `CLOB` (String mit `;`).

### KI-Prompt (Strategie)
"Transformiere diese PL/SQL-Funktion in einen Spring Data Service. Ersetze die bitweise Prüfung durch eine modernere JPA-Abfrage oder eine Collection-basierte Logik. Rückgabetyp soll eine `List<String>` sein."

### Transformierter Java-Code (Zielbild)
```java
@Service
public class PermissionService {
    @Autowired private PersonRepository personRepo;

    public List<String> getPermittedUsers(String transaction, Integer pbId, Integer accessMask) {
        List<Person> activePersons = personRepo.findByInaktivFalse();
        return activePersons.stream()
            .filter(p -> isPermitted(p, transaction, pbId, accessMask))
            .map(p -> p.getKuerzel() + "/" + p.getName())
            .toList();
    }
    // ... Hilfsmethode isPermitted mit JPA-Abfrage ...
}
```

### Abweichungsanalyse
- **Architektur:** Von Cursor-basiertem `CLOB`-Zusammenbau zu Java Streams und DTOs.
- **Performance:** Die N+1 Query Problematik des PL/SQL-Loops muss in Java durch `@EntityGraph` oder `JOIN FETCH` gelöst werden.

---

## 4. Transformation: fk_setRevStatusDatum (Komplex)

### Original (PL/SQL)
- ~1.000 Zeilen Code.
- Rekursive Aufrufe für abhängige Statusänderungen.
- Direkte `up_dbg` Logging-Aufrufe.

### KI-Prompt (Strategie)
"Analysiere diese komplexe Status-Engine. Erstelle ein Struktur-Konzept in Java, das das 'Strategy Pattern' oder eine 'State Machine' nutzt, um die harten IF/ELSE-Blöcke abzulösen. Extrahiere die Validierungsregeln in eigene Prädikate."

### Abweichungsanalyse (Herausforderungen)
- **Log-Handling:** `up_dbg` muss durch SLF4J ersetzt werden.
- **Transaktionskontrolle:** PL/SQL-interne `COMMIT`s oder `SAVEPOINT`s müssen durch `@Transactional` gesteuert werden.
- **Komplexität:** Die KI kann die 1.000 Zeilen nicht in einem Durchgang fehlerfrei migrieren; ein modularer Ansatz (Regel für Regel) ist notwendig.

---

## 5. Fazit & Empfehlung für den Rollout

1. **Effizienz:** Die KI reduziert den Schreibaufwand für Utility-Funktionen um ~90 %.
2. **Qualitätskontrolle:** Manuelle Nachbearbeitung ist bei komplexen SQL-Joins (wegen Hibernate-Mapping) und Transaktionssteuerung (20-30 %) unumgänglich.
3. **Prompt-Empfehlung:** Prompts sollten immer die Ziel-DTOs und Exception-Klassen mitgeben, um konsistenten Code zu erhalten.

**Status:** ✅ Pilot-Transformation abgeschlossen
