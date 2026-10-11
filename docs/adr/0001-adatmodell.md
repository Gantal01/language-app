# ADR 0001: Adatmodell – közös szókincs és személyes gyűjtemények

- **Státusz:** elfogadva
- **Dátum:** 2026-10-11

## Kontextus

Az alkalmazás szavakat tanít, és a szavak többféle forrásból érkezhetnek: könyv szószedetének fotójáról, beillesztett dalszövegből, vagy a felhasználó kézi felvitelével. Spanyol, angol és német nyelven használjuk majd, a fordítások nyelve magyar, de bármelyik nyelvről lehessen tanulni bármelyikre. Legalább ketten használjuk, ezért a rendszer az első verziótól többfelhasználós.

## Döntések

### 1. Két zóna: közös katalógus és személyes gyűjtemények

- **Közös zóna:** `Concept`, `Word`, `ConceptWord`. Minden felhasználó ugyanazt látja.
- **Személyes zóna:** `AppUser`, `Source`, `WordList`, `WordListEntry`. Felhasználónként elkülönül.
- A hivatkozás csak a személyes → közös irányba mehet, fordítva soha.

### 2. Fogalom-alapú szótár

A fordítás nem egy mező, hanem az, hogy több nyelvű szó ugyanahhoz a `Concept`-hez tartozik. A `Concept` nyelvfüggetlen, egyelőre csak azonosító. Így bármelyik nyelvről bármelyikre lehet gyakorolni.

### 3. Szinonimák megengedettek

A `ConceptWord` táblán csak `(ConceptId, WordId)` egyedi, `(ConceptId, Language)` nem. Egy fogalomhoz egy nyelven több szó is tartozhat. Gyakorláskor mindegyik elfogadott válasz. Elsődleges szó jelölőt (`IsPrimary`) nem vezetünk be, amíg nem hiányzik (YAGNI).

### 4. Homográfok külön szóként

Azonos írású, de különböző szavak (pl. `der See` / `die See`, `die Bank` pad / bank) külön `Word` sorok. A `Word` egyedi indexe:
`(Language, Text, PartOfSpeech, Gender, PluralForm)`, `NULLS NOT DISTINCT` beállítással, mert Postgresben alapból két `NULL` különbözőnek számít.

### 5. A névelő nem tárolt adat

A `Word.Gender` (`Masculine`, `Feminine`, `Neuter`, nullable) a nyelvtani nemet tárolja, a névelő ebből áll elő megjelenítéskor. A többes számú névelő szintén levezethető (német: mindig `die`), ezért a `PluralForm` csak a szó többes számú alakját tartalmazza.

### 6. Fejezetszám nullable

A `WordList.ChapterNumber` nullable: könyvnél a fejezet száma, dalnál és saját listánál a `null` igaz információ („nincs fejezetszáma”). Hogy könyvnél kötelező-e, azt a service réteg ellenőrzi. A szavak sorrendjét listán belül nem tároljuk.

### 7. A közös adat csak bővíthető

A felhasználók a közös zónában csak hozzáadhatnak, szerkeszteni és törölni nem tudnak. A `WordListEntry` → `Concept` kapcsolat `Restrict` törlésű. A hibás adatot egyelőre kézzel javítjuk az adatbázisban.

### 8. A szavak származását nem tároljuk

A közös adat nem rögzíti, melyik könyvből vagy dalból került be. A fotót és a dalszöveget sem tároljuk, csak a kinyert szavakat.

### 9. Az AI kimenetét a felhasználó mentés előtt átnézi

A kinyerés (fotó, dalszöveg) csak javaslatot ad. A mentés előtt a service megkeresi a közös adatban már meglévő egyezéseket, és azokat ajánlja fel, a felhasználó pedig javít vagy elfogad.

### 10. Autentikáció az első verzióban

ASP.NET Core Identity (`AddIdentityApiEndpoints`). A személyes adatok szűrése a bejelentkezett felhasználóra egy helyen valósul meg (pl. EF Core globális lekérdezési szűrő), nem metódusonként kézzel.

### 11. Első verzió online-only

Az első verzió online-only: egyetlen Postgres adatbázis, az offline tárolás és szinkron későbbi, külön feladat.

## Következmények

- A `Source`-nak nincs nyelvi párja, az, hogy melyik nyelvről tanulunk, gyakorláskor dől el.
- A `Concept` jelenleg üres burok, később kaphat például rövid angol leírást a többjelentésű szavak megkülönböztetéséhez.
- Az AI szójelentés-egyértelműsítése hibázhat (duplikátum, rossz összekötés), ezt az átnézés lépése és a közös adat bővíthetősége kezeli.
- A származás hiánya miatt egy problémás bejegyzés nem követhető vissza.
- A szerzői jogi kérdés a közös szószedet-adatnál nyitott, ezt külön tisztázni kell.

## Entitások

| Tábla | Mezők |
|---|---|
| `Concept` | `Id` |
| `Word` | `Id`, `Language`, `Text`, `PartOfSpeech`, `Gender?`, `PluralForm?` |
| `ConceptWord` | `ConceptId`, `WordId` |
| `Source` | `Id`, `UserId`, `Title`, `Type` |
| `WordList` | `Id`, `SourceId`, `Title`, `ChapterNumber?` |
| `WordListEntry` | `Id`, `WordListId`, `ConceptId` |

Enumok (stringként tárolva): `SourceType` (`Book`, `Song`, `Custom`), `Language` (`English`, `German`, `Spanish`, `Hungarian`), `PartOfSpeech` (`Noun`, `Verb`, `Adjective`, `Adverb`, `Other`), `GrammaticalGender` (`Masculine`, `Feminine`, `Neuter`).
