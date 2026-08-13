# Prijscalculator — aanbeveling (research/prototype)

Status: onderzoek/prototype. Niets hiervan is gekoppeld aan de live `index.html`.

## Het probleem met de huidige calculator

De huidige "calculator" op de site is nep: 4 losse tekstvelden en een hardcoded `€ 39,–` die nooit
verandert, ongeacht wat je invult. Er is geen backend (statische site, GitHub Pages) en geen budget
voor betaalde API's.

## Mentaal model: twee heel verschillende gevallen

### 1. De 4 vaste steden (Amsterdam, Rotterdam, Utrecht, Almere)

Dit is **geen rekenprobleem — het is een opzoeking.** Er zijn vooraf afgesproken vaste prijzen
(€39 / €69 / €59 / €49). Zodra de klant een van deze 4 kiest, toon je direct de bijbehorende prijs
uit een simpele lookup-tabel. Geen afstand, geen route, geen API, geen rekenwerk nodig — en het zou
zelfs verkeerd zijn om hier een "echte" afstandsberekening op los te laten, want de prijs ligt al
vast en hoeft niet te kloppen met de daadwerkelijke km's. Elke vorm van live-routing hiervoor is
over-engineering.

**Aanbeveling: bevestigd.** Bouw dit als een lookup-tabel (object/dictionary in JS), niet als een
berekening. Zie `CITY_PRICES` in het prototype.

### 2. Overige bestemmingen

Hier geldt het formule-tarief: **€2,15/km + €0,40/min**, altijd voor het traject Schiphol ↔ adres
(één been — de bestaande tarieven zijn ook allemaal Schiphol-gebonden, dus de calculator vraagt
alleen naar het andere adres, niet naar twee adressen).

Drie opties onderzocht:

**a. Handmatige km/min-invoer**
Twee getalvelden, klant of Michell vult de geschatte afstand en reistijd in (bv. afgelezen van
Google Maps). Nul dependencies, werkt altijd, geen netwerkverzoek, geen falen mogelijk. Nadeel: de
klant moet zelf de afstand opzoeken, wat drempel toevoegt.

**b. Automatisch schatten via een gratis geocoding/routing-API**
Onderzocht: OpenRouteService (gratis tier), OSRM public demo server, Nominatim (OpenStreetMap
geocoding), Mapbox gratis tier.

- **OSRM demo server** (`router.project-osrm.org`) — gratis, geen key nodig, maar het
  [usage policy](https://github.com/Project-OSRM/osrm-backend/wiki/API%20Usage%20Policy) zegt
  expliciet: bedoeld als demo, niet voor productie, max 1 request/sec, toegang kan zonder
  waarschuwing worden ingetrokken. Ongeschikt om op te bouwen, ook al is het bedrijf klein.
- **Nominatim** (OpenStreetMap geocoding) — gratis, maar het
  [usage policy](https://operations.osmfoundation.org/policies/nominatim/) staat max 1 request/sec
  toe, vereist een geldige User-Agent/Referer, verplicht client-side caching, en draait op
  gedoneerde servers met zeer beperkte capaciteit. Prima voor incidenteel handmatig testen, niet iets
  om een commerciële site structureel op te laten leunen.
- **OpenRouteService (ORS)** — gratis tier met een geregistreerde API-key: ~2.500 requests/dag,
  40.000/maand, 40 gelijktijdige requests. Dit is de enige van de drie die expliciet bedoeld is voor
  dit soort gebruik (geocoding + directions, met een formeel free-tier contract i.p.v. "misbruik
  van een demo-server"). Voor één chauffeur/één auto met misschien een paar tientallen offerte-
  aanvragen per dag is dit ruim voldoende, voor jaren vooruit.
  - **Kanttekening (blootstelling van de key):** dit is een statische site zonder server, dus de
    ORS-key staat noodzakelijkerwijs in client-side JS. Voor een klein bedrijf is dit een
    acceptabel risico — ORS staat toe om de key te beperken tot specifieke HTTP-referrers
    (domeinnaam) in hun dashboard, wat misbruik vanaf andere sites grotendeels blokkeert. Het is
    geen geheime sleutel die geld kan kosten zonder limiet; bij overschrijding van de gratis tier
    stopt de key simpelweg met werken (geen automatische facturering bekend op de gratis tier).

**c. Middenweg (wat het prototype implementeert)**
Handmatige invoer als **standaard, altijd werkend pad**. Automatisch schatten via ORS als
**optionele, uit-te-schakelen** toggle: de gebruiker plakt zijn eigen gratis ORS-sleutel (opgeslagen
in `localStorage`, nooit hardcoded in de code, nooit verzonden naar iets anders dan ORS zelf). Als er
geen sleutel is ingevuld, valt de flow netjes terug op de handmatige tab met een duidelijke melding.

## Aanbeveling: v1 vs. later

**v1 — nu bouwen (dit is ook wat het prototype laat zien werken):**
- 4 steden als pure lookup-tabel, instant resultaat bij selectie.
- "Andere bestemming" met **handmatige** km/min-invoer als enige verplichte pad. Geen API-key
  vereist om te functioneren. Dit is voor een static site zonder backend de meest robuuste keuze:
  niets kan breken, niets kan een rate limit raken, niets hangt af van een derde partij die blijft
  bestaan.
- Formule en tarieven als benoemde constanten in de code, makkelijk aan te passen door Michell/de
  developer als de prijzen wijzigen.

**Later — als het bedrijf groeit en het de moeite waard is:**
- Zet de optionele ORS auto-estimate standaard aan (met een eigen, referrer-beperkte key) zodra er
  genoeg volume is dat het gemak opweegt tegen het (kleine) onderhoudsrisico van een externe
  dependency.
- Overweeg een simpele rond-af-regel voor weergegeven prijzen (nu toont het prototype exacte
  centen, bv. €65,75 — dat kan prima blijven, of afgerond worden op hele/halve euro's, een
  bedrijfsbeslissing van Michell, niet een technische).
- Als volume ooit de gratis ORS-limiet nadert (2.500/dag): overstappen op een betaalde ORS-tier of
  Mapbox is een kleine wijziging, geen herontwerp — de auto-estimate-functie is al los ontworpen
  van de rest van de calculator.

## Wat er in het prototype zit

`calculator-prototype.html` — één zelfstandig bestand, geen build-stap, direct te openen in een
browser:
- `CITY_PRICES` lookup-object voor de 4 vaste steden (`lookupFixedPrice()`).
- `calcOtherPrice(km, min)` — echte berekening met `RATE_PER_KM = 2.15` en `RATE_PER_MIN = 0.40` als
  benoemde constanten, geen hardcoded uitkomst.
- Ingebouwde self-test in de browserconsole bij het laden (`[calc self-test]` regels) die de
  formule tegen een paar handmatig uitgerekende gevallen controleert.
- Optionele, uit te vouwen ORS-integratie (geocoding + directions), key alleen in `localStorage`,
  duidelijk gelabeld als optioneel, met documentatie van het gratis tier en de limieten er direct
  naast in de UI.

## Getest (handmatig, in browser)

- Klik op "Rotterdam" → toont direct `€ 69,–` (lookup, geen berekening).
- Klik op "Amsterdam" → toont direct `€ 39,–`, "andere bestemming"-paneel verdwijnt correct.
- Klik op "Andere bestemming" → handmatig 25 km + 30 min ingevuld → live resultaat
  `25 km × €2,15 + 30 min × €0,40 = € 65,75`, wat overeenkomt met de handmatige uitkomst
  (25 × 2,15 = 53,75; 30 × 0,40 = 12; totaal 65,75).
