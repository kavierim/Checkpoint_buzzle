# Paikannusarvoitus

Selainpohjainen GPS-paikannuspeli mobiilille. Upotetaan WordPress-sivulle.

## Sisällysluettelo

- [Miten peli toimii](#miten-peli-toimii)
- [Vaatimukset](#vaatimukset)
- [Tiedostot](#tiedostot)
- [Asennus](#asennus)
  - [1. Karttakuvan valmistelu](#1-karttakuvan-valmistelu)
  - [2. Koordinaattien selvittäminen](#2-koordinaattien-selvittäminen)
  - [3. GPX-tiedoston muokkaus](#3-gpx-tiedoston-muokkaus)
  - [4. Tiedostojen lataus palvelimelle](#4-tiedostojen-lataus-palvelimelle)
  - [5. Upotus WordPressiin](#5-upotus-wordpressiin)
- [GPX-asetukset](#gpx-asetukset)
- [Testaus](#testaus)
- [Ongelmatilanteet](#ongelmatilanteet)

---

## Miten peli toimii

1. Pelaaja avaa sivun puhelimella ja aloittaa pelin
2. Kartalla näkyy keräilypisteet (punaiset) ja loppupiste (kultainen tähti)
3. Pelaaja kävelee fyysisesti pisteisiin – lähellä ollessaan piste muuttuu vihreäksi ja näyttää vihjeen + kirjaimen
4. Karttaa voi liikuttaa sormella ja zoomata nipistämällä tai +/− napeilla
5. Kun kaikki pisteet on kerätty, pelaaja siirtyy loppupisteeseen
6. Loppunäytöllä pelaaja muodostaa sanan kerätyistä kirjaimista ja ratkaisee arvoituksen

---

## Vaatimukset

- WordPress-sivu **HTTPS**-osoitteessa (http ei toimi GPS:n kanssa)
- Pelaajalla GPS-lupa selaimessa
- Pelialue ulkona (GPS-tarkkuus sisällä on heikko)

---

## Tiedostot

| Tiedosto | Kuvaus |
|---|---|
| `game.html` | Koko peli — kopioidaan WordPress Custom HTML -lohkoon |
| `game.gpx` | Pelisisältö ja asetukset — ladataan palvelimelle |
| `kartta.png` | Karttakuva — ladataan palvelimelle |

Pelin asetukset (pisteet, kirjaimet, vihjeet, karttakuva) ovat **GPX-tiedostossa**, ei `game.html`:ssä. Tämä tarkoittaa että pelisisältöä voi muuttaa ilman `game.html`-tiedoston muokkaamista.

---

## Asennus

### 1. Karttakuvan valmistelu

Paras tapa on viedä kartta OpenStreetMapin Export-toiminnolla:

1. Avaa [openstreetmap.org](https://www.openstreetmap.org)
2. Siirry pelialueelle
3. Klikkaa **Export** → valitse alue → **Export**
4. Tallenna kuva nimellä `kartta.png`
5. Kopioi talteen alueen nurkkien koordinaatit (tarvitaan vaiheessa 3)

### 2. Koordinaattien selvittäminen

Tarvitset koordinaatit jokaiselle pisteelle. Helpoin tapa:

**Google Mapsissa:**
1. Avaa [maps.google.com](https://maps.google.com)
2. Klikkaa haluttua kohtaa kartalla
3. Koordinaatit näkyvät alareunassa muodossa `64.1234, 25.5678` (lat, lon)

**Tarvittavat koordinaatit:**
- Karttakuvan **vasen yläkulma** (pohjoinen + länsi) → GPX `maxlat` ja `minlon`
- Karttakuvan **oikea alakulma** (etelä + itä) → GPX `minlat` ja `maxlon`
- Jokaisen keräilypisteen sijainti
- Loppupisteen sijainti

### 3. GPX-tiedoston muokkaus

Avaa `game.gpx` tekstieditorissa ja muokkaa seuraavat kohdat:

**Kartan kulmat** (`<bounds>`-tagi):
```xml
<bounds minlat="64.896895" minlon="25.513276"
        maxlat="64.909018" maxlon="25.544147"/>
```
- `minlat` = eteläisin leveysaste (oikea alakulma)
- `maxlat` = pohjoisin leveysaste (vasen yläkulma)
- `minlon` = läntisin pituusaste (vasen yläkulma)
- `maxlon` = itisin pituusaste (oikea alakulma)

**Yleiset asetukset** (`<extensions>`-tagi metadatassa):
```xml
<game:mapImage>https://sinun-sivusi.fi/kartta.png</game:mapImage>
<game:geofenceRadius>15</game:geofenceRadius>
<game:gpsAccuracyLimit>30</game:gpsAccuracyLimit>
<game:checkAnswer>true</game:checkAnswer>
<game:puzzleInstruction>Muodosta sana kerätyistä kirjaimista.</game:puzzleInstruction>
<game:puzzleAnswer>KOTI</game:puzzleAnswer>
```

**Loppupiste** (`type=end`):
```xml
<wpt lat="64.900809" lon="25.527019">
  <name>Pääskyläntie 2</name>
  <type>end</type>
  <desc>Teksti joka näkyy loppunäytöllä.</desc>
</wpt>
```

**Keräilypisteet** (`type=collect`, yksi per kirjain):
```xml
<wpt lat="64.905434" lon="25.520382">
  <name>Koskela</name>
  <type>collect</type>
  <extensions>
    <game:id>1</game:id>
    <game:letter>K</game:letter>
    <game:hint>Vihjeteksti joka näkyy popup-ikkunassa.</game:hint>
    <!-- Valinnainen: kuva näytetään popupissa vihjetekstin yläpuolella -->
    <game:image>https://sinun-sivusi.fi/kuva1.jpg</game:image>
    <!-- Valinnainen: kustomoitu ikoni kartalla ja alapalkissa (esim. emoji) -->
    <game:icon>🏠</game:icon>
  </extensions>
</wpt>
```

`game:id` määrittää kirjainten järjestyksen loppunäytöllä (1, 2, 3...). Pisteet voi kerätä missä järjestyksessä tahansa. `game:image` ja `game:icon` ovat valinnaisia — jos `game:icon` puuttuu, näytetään `?`-merkki; jos `game:image` puuttuu, popup näyttää vain vihjetekstin.

### 4. Tiedostojen lataus palvelimelle

Lataa nämä tiedostot palvelimelle (esim. WordPress Media tai FTP):

| Tiedosto | Mihin |
|---|---|
| `kartta.png` | Palvelimen juureen tai Media-kirjastoon |
| `game.gpx` | Palvelimen juureen tai Media-kirjastoon |

Varmista että `game:mapImage`-osoite GPX-tiedostossa (tai `gpxInline`-lohkossa) vastaa tiedoston sijaintia.

### 5. Upotus WordPressiin

1. Avaa WordPress-editori (Gutenberg)
2. Lisää lohko: **"/"** → kirjoita "HTML" → valitse **"Mukautettu HTML"**
3. Kopioi koko `game.html`-tiedoston sisältö lohkoon
4. Tallenna / julkaise sivu
5. Testaa puhelimella HTTPS-osoitteessa

---

## GPX-asetukset

| Kenttä | Tyyppi | Oletus | Kuvaus |
|---|---|---|---|
| `<name>` | string | – | Pelin otsikko |
| `<bounds>` | attribuutit | – | Karttakuvan nurkat (minlat, maxlat, minlon, maxlon) |
| `game:mapImage` | URL | – | Karttakuvan osoite |
| `game:geofenceRadius` | number | `15` | Etäisyys metreinä pisteen aktivoitumiseen |
| `game:gpsAccuracyLimit` | number | `30` | GPS-tarkkuusraja metreinä |
| `game:checkAnswer` | boolean | `true` | Tarkistetaanko vastaus (`true`/`false`) |
| `game:puzzleInstruction` | string | – | Ohjeteksti loppunäytöllä |
| `game:puzzleAnswer` | string | – | Oikea vastaus (kirjainkoolla ei väliä) |
| `game:hint` | string | – | Vihjeteksti keräilypisteen popupissa |
| `game:image` | URL | – | (Valinnainen) Kuva keräilypisteen popupissa vihjetekstin yläpuolella |
| `game:icon` | string | `?` | (Valinnainen) Kustomoitu ikoni kartalla ja alapalkissa (esim. emoji) |

Lisäksi `game.html`:ssä on minimaalinen konfiguraatio. Oletuksena GPX upotetaan suoraan tiedostoon (`gpxInline`), mutta sen voi myös ladata palvelimelta:

```javascript
const CONFIG = {
  // Vaihtoehto 1 (oletus): GPX upotettu suoraan — yksitiedostoinen, ei CORS-ongelmia
  gpxInline: `<?xml ...>`,

  // Vaihtoehto 2: lataa GPX URL:sta (käytä kun GPX on samalla palvelimella)
  // gpxUrl: "https://sinun-sivusi.fi/game.gpx",
};
```

---

## Testaus

**Debug-tilassa** (ilman fyysistä liikettä):
1. Lisää `?debug` sivun URL-osoitteen loppuun (esim. `http://localhost:8000/game.html?debug`)
2. Avaa peli selaimessa (voidaan testata myös ilman HTTPS:ää)
3. Napauta karttaa → pelaajan sijainti siirtyy napautettuun kohtaan
4. Geofence-säteet näkyvät kartalla katkoviivaympyröinä

Debug-tila ei vaadi muutoksia `game.html`-tiedostoon.

**Lokaalisti** (ei tarvitse palvelinta):
```bash
uv run python -m http.server 8000
```
Avaa selaimessa: `http://localhost:8000/game.html`

**Oikealla GPS:llä:**
- Testaa aina HTTPS-osoitteessa
- Ulkona, hyvässä GPS-signaalissa
- Tarkista että pisteet täsmäävät karttakuvaan

---

## Ongelmatilanteet

| Ongelma | Syy | Ratkaisu |
|---|---|---|
| "Sijaintilupa vaaditaan" | GPS-lupa estetty | Salli sijainti selaimen asetuksista |
| Karttaa ei näy | Väärä URL `game.gpx`:ssä | Tarkista `game:mapImage`-osoite |
| Pisteet ovat väärässä kohdassa | `<bounds>`-koordinaatit väärin | Tarkista karttakuvan kulmat |
| Peli ei lataa asetuksia | `game.gpx` ei löydy | Tarkista `gpxUrl` `game.html`:ssä; CORS-ongelma jos eri palvelimella |
| Karttaa ei pysty vierittämään | Selain estää touch-tapahtumat | Varmista HTTPS; kokeile toisella selaimella |
| Peli ei toimi WordPressissä | `<script>`-tagit riisuttu | Kokeile Code Snippets -pluginia tai omaa sivupohjaa |
| GPS-signaali heikko | Sisätila tai huono sää | Siirry ulos, odota signaalia |
| Edistyminen häviää | localStorage tyhjennetty | Normaalia – peli alkaa alusta |
| Peli jumittuu kesken | Aloita alusta -nappi | Paina ↺-nappia otsikossa tai "Aloita alusta" loppunäytöllä |
