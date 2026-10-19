# 🌍 GeoPeli

Paikallinen GeoGuessr-tyylinen peli, jota pelataan **samassa wifissä**. Yksi tietokone (isäntä) näyttää katukuvan
videotykillä ja 2–20 pelaajaa arvaa sijainnin **puhelimellaan** kartalta. Mitään ei tarvitse asentaa pelaajien puhelimiin,
ja isäntäkoneelle riittää pelkkä zip-paketti.

## Pikaohje

1. Lataa uusin `GeoPeli-windows.zip` ([Releases](../../releases)) ja pura se.
2. Hanki ilmainen Mapillary-token (ks. [Kuvapalvelu](#kuvapalvelu)) ja liitä se tiedostoon `mapillary.txt`.
3. Tuplaklikkaa `start.bat`. Isäntänäkymä aukeaa selaimeen: heijasta se videotykille.
4. Salli Windowsin palomuurikehotteessa **Node.js yksityisessä verkossa**.
5. Jaa aulassa näkyvä osoite (📋 Kopioi tai WhatsApp-painike) pelaajille. He avaavat sen puhelimella samassa wifissä.
6. Valitse alueet ja pelitila, ja paina **Aloita peli**.

> Zipissä on mukana Noden oma `node\node.exe` (Windows 64-bit), joten mitään ei tarvitse asentaa eikä `npm install` ole tarpeen.
> Isäntänäkymä toimii vain osoitteella `http://localhost:3000/host` (samalta koneelta).

## Ominaisuudet

### Pelitilat
- **Klassinen:** vain videotykki näyttää katukuvan, pelaajat arvaavat kartalta.
- **Kaikki liikkuvat itse:** kaikki pelaajat aloittavat samasta kuvasta ja liikkuvat omalla puhelimellaan katukuvassa
  (välilehdet 👁 Katukuva / 🗺 Kartta). Aikaraja nousee tilassa oletuksena 120 sekuntiin.

### Alueet
Eurooppa, Pohjoismaat, Suomi, Pohjois-Amerikka, Etelä-Amerikka, Aasia, Afrikka ja Oseania, yksi tai useampi yhtä aikaa.
Paikka arvotaan tasaisesti kaupunkien kesken (suuria ja pieniä). **Suomi**-alueella on mukana 95 pientä paikkakuntaa
(Etelä-Pohjanmaa, Lappi, Savo, Kainuu…), joiden kattavuus on testattu.

### Pisteytys
Jokaisen kierroksen pisteet koostuvat kolmesta osasta:

| Osa | Pisteet |
|---|---|
| Etäisyys | enintään 5000: `5000 · e^(−km/1492,7)` (GeoGuessrin maailmankaava) |
| Nopeus ⚡ | enintään +1000, pienenee ajan kuluessa ja skaalautuu tarkkuuden mukaan |
| Lähin 🎯 | +500 lähimmän arvauksen tehneelle |

**Pelkkä Suomi:** pisteitä vain alle 500 km:n arvauksesta: `5000 · (1 − km/500)²`, ja kauempaa 0 p.
Säde on vakio `FINLAND_RADIUS_KM` tiedostossa `public/host.html`.

Tuloksissa näkyy oikea paikka (kaupunki, maa), kierroksen ja kokonaispisteet sekä kokonaissijoitus nuolineen (▲▼).

### Isännän hallinta
- Seuraava kierros alkaa itsestään tulosten katseluajan jälkeen (oletus 15 s, säädettävä aulassa). **⏸ Tauko** pysäyttää laskurin.
- **⏹ Lopeta peli** kesken pelin näyttää lopputulokset.
- Aulassa pelaajan nimen vieressä **✕** poistaa pelaajan (poistettu ei pääse takaisin samalla laitteella ennen palvelimen uudelleenkäynnistystä).
- Pelaajat voivat vaihtaa nimeään aulassa.
- Kierros päättyy, kun kaikki ovat arvanneet tai aika loppuu.

### Huijauksen huomautus
Jos pelaajan sivu piilotetaan tai menettää tarkennuksen kierroksen aikana (esim. toinen sovellus auki), isännälle näkyy
"⚠ poissa" tuloksissa. Se **ei vaikuta pisteisiin**. Alle 3 sekunnin poissaolot ja sivun päivitys ohitetaan.
Toista laitetta tai jaettua näyttöä peli ei voi havaita. Mapillaryn ja Googlen logolinkit (jotka avaisivat oikean paikan kartalla)
on poistettu käytöstä.

### Kartta
Pohjakartta on OpenFreeMap (MapLibre), jossa maat ja kaupungit ovat **suomeksi**, ja **tienumerot** (kilvet) sekä isot tiet
näkyvät jo kaukaa. Jos puhelimen WebGL ei toimi tai muisti on vähissä, kartta vaihtuu automaattisesti kevyeen OpenStreetMap-rasteriin.

## Kuvapalvelu

Peli tarvitsee katukuvapalvelun. Valitse toinen (tai molemmat; jos molemmat ovat käytössä, aulassa voi valita):

**Mapillary (ilmainen, ei laskutustiliä)** – suositeltu
1. Kirjaudu osoitteessa https://www.mapillary.com/dashboard/developers (ilmainen tili).
2. *Register application*, anna mikä tahansa nimi ja kopioi **Client Token** (alkaa `MLY|`).
3. Liitä se tiedostoon `mapillary.txt` (tai aseta ympäristömuuttuja `MAPILLARY_TOKEN`).

Peli valitsee kuvan automaattisesti: tavalliset kuvat vain **autolla ajetuista sarjoista** (keskinopeus yli 6,5 m/s),
jotta kävelytiet jäävät pois. 360°-kuvat kelpaavat myös hitaammista sarjoista (yli noin 3 m/s) ja yksittäisinä.
Kuvia haetaan 8 kaupungista rinnakkain, ja seuraava paikka haetaan valmiiksi taustalla edellisen kierroksen aikana.

**Google Street View** – paras kattavuus, mutta vaatii Google Cloud -projektin, jossa laskutus on käytössä
- Ota käyttöön *Maps JavaScript API*, luo API-avain ja liitä se tiedostoon `key.txt` (tai `GOOGLE_MAPS_KEY`).
- Ilman laskutusta kuvaan tulee "For development purposes only" -vesileima eikä peli ole pelattavissa.

> ⚠️ Pidä `mapillary.txt` ja `key.txt` omana tietonasi: älä lisää niitä julkiseen repoon tai jaa zipissä.
> Pelaajien puhelimet saavat tokenin "kaikki liikkuvat itse" -tilassa (katselin tarvitsee sen).

## Käyttö ilman zipiä (kehittäjille)

```bash
git clone <repo>
cd GeoPeli
# Asenna Node.js 18+ (nodejs.org), lisää mapillary.txt
node server.js
```

Avaa isäntänäkymä osoitteessa `http://localhost:3000/host`. Portin voi vaihtaa: `PORT=3100 node server.js`.
Palvelin ei käytä yhtään npm-pakettia. Kirjastot (Leaflet, MapLibre GL, MapillaryJS) ovat kansiossa `public/vendor`.

## Rakenne

```
server.js            Palvelin: pelin tila, pisteytys, SSE-päivitykset (/events), isäntä-API (vain localhost)
start.bat            Käynnistää palvelimen ja avaa isäntänäkymän (käyttää node\node.exe jos löytyy)
public/host.html     Isäntänäkymä (videotykki): aula, katukuva, tulokset
public/player.html   Pelaajan puhelinnäkymä: arvauskartta ja tulokset
public/basemap.js    Suomenkielinen pohjakartta, tienumerot ja varakartta
public/style.css     Yhteiset tyylit
public/vendor/       Kolmannen osapuolen kirjastot (paikallisesti, ei CDN:ää)
```

## Vianetsintä

| Ongelma | Ratkaisu |
|---|---|
| Puhelin ei pääse sivulle | Samassa wifissä? Salli Node.js palomuurissa (yksityinen verkko). Joissain wifeissä "asiakkaiden eristys" estää yhteyden |
| Isäntä näyttää "avain puuttuu" | Avaa isäntä osoitteella `localhost:3000/host`, ei IP-osoitteella. Tarkista `mapillary.txt` |
| Paikan haku kestää | Yleensä 1–6 s. Haku jatkuu itsestään ja Peruuta-painike keskeyttää. Mapillary voi olla hidas tiheissä kaupungeissa |
| Pelaajan ruutu jää vanhaan kierrokseen | Sivu yrittää korjata itsensä (uudelleenyhteys ja tarvittaessa lataus). Päivitys auttaa aina |
| Kartta tyhjä | Odota hetki tai päivitä. Heikolla puhelimella kartta vaihtuu rasteriin |

## Tekniikkaa

Palvelin pitää pelin tilan muistissa ja lähettää sen kaikille selaimille Server-Sent Events -yhteydellä (10 s sykkeellä).
Selaimet lähettävät toiminnot POST-pyynnöillä. Isännän toiminnot (`/api/host/*`) hyväksytään vain `localhost`-osoitteesta.
Tilaa ei tallenneta levylle, joten palvelimen uudelleenkäynnistys nollaa pelin (pelaajat liittyvät uudelleen automaattisesti).

## Kiitokset ja lähteet

- Katukuvat: [Mapillary](https://www.mapillary.com) (kuvat CC BY-SA; tekijä näkyy katselimessa), valinnaisesti Google Street View
- Kartta: [OpenFreeMap](https://openfreemap.org), [OpenMapTiles](https://openmaptiles.org), data © [OpenStreetMap](https://www.openstreetmap.org/copyright)-tekijät
- Paikannimet: [Nominatim](https://nominatim.org) (OpenStreetMap)
- Kirjastot: [Leaflet](https://leafletjs.com), [MapLibre GL JS](https://maplibre.org), [MapillaryJS](https://github.com/mapillary/mapillary-js)

Lisenssi: [MIT](LICENSE). Kolmannen osapuolen kirjastoilla (`public/vendor`) on omat lisenssinsä (BSD-2-Clause / MIT).
