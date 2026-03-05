# Quick Wins: Analyse & Code-Skelette (Java/Spring)

> [!CHECK]
> **STATUS WOCHE 1:**  
> Die hier analysierten Quick Wins dienten als Basis für den erfolgreichen **Proof of Concept (PoC)** in Woche 1. Die funktionale Gleichwertigkeit wurde für `FK_CALC_DISTANCE` bereits methodisch nachgewiesen.

Diese Datei enthält die technische Analyse und die ersten Java-Code-Vorschläge für die als "Quick Wins" identifizierten PL/SQL-Funktionen.

---

## 1. Geo-Berechnung (`FK_CALC_DISTANCE`)
**Original:** `PKG_APT_UTIL.FK_CALC_DISTANCE` (Kategorie: Einfach)

### Technische Analyse
- **Eingabe:** Längen- und Breitengrade (Punkt A und B) in Grad.
- **Logik:** Nutzt die Haversine-Formel zur Distanzberechnung auf einer Kugel (Erdradius ~6378 km).
- **Abhängigkeiten:** Keine (Zustandslos).
- **Status W1:** ✅ Validierung abgeschlossen. Siehe [Validierungsnachweis: FK_CALC_DISTANCE](./Validierungsnachweis_Beispiel_FK_CALC_DISTANCE.md).

### Java-Skelett (Utility)
```java
// ... (Code unverändert)
```

import org.springframework.stereotype.Component;

@Component
public class GeoUtils {
    private static final double EARTH_RADIUS_KM = 6378.388;

    public Double calculateDistance(Double lngA, Double latA, Double lngB, Double latB) {
        if (lngA == null || latA == null || lngB == null || latB == null) return null;
        if (lngA < 0.01 || latA < 0.01) return null; // Plausibilitätsprüfung aus PL/SQL

        double lngARad = Math.toRadians(lngA);
        double latARad = Math.toRadians(latA);
        double lngBRad = Math.toRadians(lngB);
        double latBRad = Math.toRadians(latB);

        double dist = Math.acos(Math.sin(latARad) * Math.sin(latBRad) 
                    + Math.cos(latARad) * Math.cos(latBRad) * Math.cos(lngBRad - lngARad)) 
                    * EARTH_RADIUS_KM;

        return Double.isNaN(dist) ? 0.0 : dist;
    }
}
```

---

## 2. Rechteprüfung (`fk_getPermittedUsers`)
**Original:** `PKG_APT_RECHTE.fk_getPermittedUsers` (Kategorie: Mittel)

### Technische Analyse
- **Eingabe:** Benutzer, Transaktion, PB_ID, AccessMask (Bitmaske).
- **Logik:** Iteriert über alle aktiven Personen und prüft Berechtigungen gegen eine Bitmaske (Lesen=1, Neu=2, Ändern=4, Löschen=8, Admin=16).
- **Besonderheit:** Admins (`per.ADMIN = '1'`) haben immer Zugriff.

### Java-Skelett (Service)
```java
@Service
public class PermissionService {
    @Autowired private PersonRepository personRepository;
    @Autowired private PermissionRepository permissionRepository;

    public String getPermittedUsers(String transactionCode, Long pbId, int accessMask) {
        List<Person> activePersons = personRepository.findByInaktivFalseOrderByKuerzel();
        StringBuilder result = new StringBuilder();

        for (Person per : activePersons) {
            boolean show = false;
            if (accessMask == 0 || pbId == null || "1".equals(per.getAdmin())) {
                show = true;
            } else {
                int access = calculateAccessValue(per.getId(), transactionCode, pbId);
                show = (access & accessMask) != 0;
            }

            if (show) {
                result.append(per.getKuerzel()).append("/").append(per.getName()).append(";");
            }
        }
        return result.toString();
    }

    private int calculateAccessValue(Long personId, String transactionCode, Long pbId) {
        // Hier: JPA Join Query über ADM_BERECHTIGUNGEN, ADM_TRANSAKTIONEN etc.
        // Rückgabe als Bitmaske (1, 2, 4, 8, 16)
        return permissionRepository.findAccessMask(personId, transactionCode, pbId);
    }
}
```

---

## 3. Link-ID Generierung (`FK_GET_USER_LINK_ID`)
**Original:** `PKG_IF_APT_CLIENT.FK_GET_USER_LINK_ID` (Kategorie: Mittel)

### Technische Analyse
- **Eingabe:** Wunsch-ID, SZN_ID (Szenario).
- **Logik:** Zerlegt die ID in Präfix (Region+Typ) und laufende Nummer. Prüft Verfügbarkeit in `KAP_LINK` und einer View (`V_IF_APT_LINKIDS`).
- **Besonderheit:** Wenn ID belegt ist, wird die nächste freie Nummer im Bereich gesucht.

### Java-Skelett (Service)
```java
@Service
public class LinkIdService {
    @Autowired private LinkRepository linkRepository;

    public String getAvailableLinkId(String preferredId, Long sznId) {
        if (preferredId == null) return null;

        // 1. Existenzprüfung (analog zu PL/SQL v_OK)
        boolean isUsed = linkRepository.existsByUserLinkIdAndSznId(preferredId, sznId);
        
        if (!isUsed) {
            return preferredId;
        }

        // 2. Neuvorschlag Logik (FK_NEXT_FREE_USER_LINK_ID)
        String prefix = preferredId.substring(0, 5);
        Integer startNr = Integer.parseInt(preferredId.substring(5));
        
        return findNextFreeId(prefix, startNr);
    }

    private String findNextFreeId(String prefix, Integer startNr) {
        // Suche höchste belegte Nummer im Bereich und addiere 1
        // Oder nutze eine DB-Sequence/Custom Generator
        return linkRepository.findFirstFreeIdInPrefixRange(prefix, startNr);
    }
}
```

---

## Nächste Schritte
1. **Unit Tests:** Erstellung von JUnit-Tests für `GeoUtils`, um die mathematische Korrektheit gegen die PL/SQL-Ergebnisse zu validieren.
2. **Repository-Design:** Definition der benötigten JPA-Methoden für `PermissionService` und `LinkIdService`.
3. **Integration:** Einbindung der Services in den bestehenden Spring Boot Context.
