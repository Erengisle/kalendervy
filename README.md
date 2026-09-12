# Veckoplanering från Google Kalender och läsårsplan

Google Apps Script som fyller en veckoplanering i Google Kalkylark. Du skriver
ett veckonummer i en cell, väljer **Veckoplanering → Fyll veckan**, och skriptet
hämtar veckans lektioner från Google Kalender och veckans händelser från skolans
läsårsplan och skriver in dem i rätt dagkolumn.

Skriptet rör aldrig formateringen. Det rensar bara innehåll (`clearContent`) och
skriver värden, så mallens ramar, färger och radhöjder överlever varje körning.

## Installation

1. Öppna kalkylarket med veckoplaneringsmallen → **Tillägg → Apps Script**.
2. Klistra in koden och spara.
3. Justera `CONFIG` (se nedan).
4. Ladda om kalkylarket. Menyn **Veckoplanering** dyker upp.
5. Första körningen begär behörighet till kalender och kalkylark.

## Användning

| Menyval | Vad det gör |
| --- | --- |
| Fyll veckan | Läser veckonumret i `E1` och fyller dagkolumnerna |
| Rensa dagkolumnerna | Tömmer A4:E16 utan att röra formateringen |
| Aktivera automatisk ifyllning | Installerar en trigger som kör ifyllningen så fort `E1` ändras |
| Stäng av automatisk ifyllning | Tar bort triggern |

Automatiken är avstängd som standard. Den skriver över allt du själv har skrivit
i dagkolumnerna varje gång `E1` ändras, och den gör varje redigering långsammare
eftersom både kalendern och det andra kalkylarket måste läsas om.

## Mallens uppbyggnad

Skriptet utgår från den här layouten. Ändrar du den, uppdatera `CONFIG.blad`.

| Cell/område | Innehåll |
| --- | --- |
| `E1` | Veckonummer — det enda du fyller i själv |
| `E2` | År (tomt = innevarande år) |
| `F1` | Måndagens datum, skrivs av skriptet |
| `A1` | Rubrik: "Veckoplanering – vecka N" |
| `A2` | Datumintervall: "14 september – 18 september 2026" |
| Rad 3, A–E | Veckodagsrubriker: "Måndag  14/9" osv. |
| A4:E16 | Dagkolumnerna — hit skrivs lektioner och händelser |
| Rad 17 | Kommentarsrad, rörs inte |

Celler som innehåller **formler** hoppas över. Om `A1`, `A2`, `F1` eller
veckodagsraden redan räknar ut sig själva utifrån `E1` lämnas de alltså ifred,
och du kan välja fritt mellan formler och skriptgenererade värden.

Får en dag fler poster än de tretton raderna slås överskottet ihop på sista
raden. Skriptet infogar inte nya rader, eftersom det skulle bryta mallen.

## Inställningar i CONFIG

### Kalendern

| Inställning | Betydelse |
| --- | --- |
| `kalenderId` | `'primary'` för din egen kalender, annars kalenderns id från Inställningar → Integrera kalender |
| `titelFilter` | Lista med ord, t.ex. `['SVA', 'Historia']`. Bara händelser vars titel innehåller något av orden tas med. Tom lista = alla händelser. |
| `taMedHeldagshändelser` | `false` sållar bort heldagshändelser |
| `visa.tid` | Klockslag först i raden, t.ex. `08:00–09:30` |
| `visa.titel` | Händelsens rubrik |
| `visa.plats` | Sal eller plats inom parentes |
| `visa.beskrivning` | Kalenderns anteckningsfält, avskalat från html |

### Läsårsplanen

| Inställning | Betydelse |
| --- | --- |
| `kalkylarkId` | Id:t för läsårsplanen (den långa strängen i dess url) |
| `flikar` | Vilka flikar som genomsöks, t.ex. `['HT2026', 'VT 2026']` |
| `placering` | `'topp'` lägger händelserna ovanför lektionerna, `'botten'` under |
| `prefix` | Tecken framför varje händelse, t.ex. `'» '`, om du vill skilja dem från lektionerna |
| `veckonoteringar` | Tar med A/B-vecka och noteringskolumnen längst till höger, överst i måndagskolumnen |

### Bladet

`CONFIG.blad` talar om var i mallen sakerna står: `vecka`, `år`, `måndagsdatum`,
`rubrik`, `datumintervall`, `veckodagsrad`, `förstaRad`, `sistaRad`,
`förstaKolumn` och `antalDagar`. `namn: ''` betyder att den aktiva fliken
används; sätt ett fliknamn om du vill låsa skriptet till en bestämd flik.

## Så läses läsårsplanen

Läsårsplanen är ett kalendarium, inte en tabell med rubrikrad. Uppbyggnaden är:

- ett block per vecka, flera rader högt
- veckonumret i kolumn A, och `A-vecka`/`B-vecka` på en egen rad i samma kolumn
- varje veckodag upptar två kolumner: datum (`17 aug.`) i den vänstra, innehåll i båda
- kolumnerna längst till höger är noteringar som gäller hela veckan

Skriptet hittar rätt block genom att leta efter veckonumret i kolumn A och sedan
kontrollera att blockets första datum stämmer med måndagens datum för veckan.
Det gör att vecka 1 i HT-fliken inte förväxlas med vecka 1 i VT-fliken. Åren
behöver aldrig räknas ut, eftersom matchningen sker på veckonummer.

Ändrar skolan layouten — lägger till en kolumn, flyttar veckonumret — slutar
matchningen fungera och inga händelser skrivs in. Skriptet säger då till i
notisen längst ner: "ingen matchande vecka i läsårsplanen".

## Id i skriptegenskaper (valfritt)

Ligger repot publikt kan du flytta ut kalkylarks-id:t ur koden. Lägg det i
**Projektinställningar → Skriptegenskaper** med namnet `LASARSPLAN_ID` och byt
raden i `CONFIG` mot:

```js
kalkylarkId: PropertiesService.getScriptProperties().getProperty('LASARSPLAN_ID'),
```

## Felsökning

| Symptom | Trolig orsak |
| --- | --- |
| "Skriv ett veckonummer i E1 först" | `E1` är tom eller innehåller text |
| Rubrikerna uppdateras inte | Cellerna innehåller formler och hoppas därför över |
| Inga lektioner | `titelFilter` sållar bort dem, eller fel `kalenderId` |
| Inga händelser från läsårsplanen | Fel fliknamn, eller att layouten har ändrats |
| Fel vecka hämtas | `E2` innehåller fel år — veckonumrering räknas per år |
