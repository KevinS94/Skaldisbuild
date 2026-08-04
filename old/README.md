# SKALDIS BUILD Configurator

Deze versie bevat:

- een top-down editor met snapping op rasterranden en rastervakken;
- een gekoppelde 3D-weergave;
- een live stuklijst;
- lokaal opslaan en openen van ontwerpen als JSON;
- inklapbare assetpacks in de linkerzijbalk;
- een aparte bibliotheek voor de persoonlijke fysieke collectie;
- een grijze placeholder wanneer een GLB-model ontbreekt.

## Starten

Start bij voorkeur een lokale webserver in de map van het project:

```bash
python -m http.server 8000
```

Open daarna:

```text
http://localhost:8000
```

De 3D-bibliotheek Three.js wordt via jsDelivr geladen. Hiervoor is een internetverbinding nodig.

## Packs in de configurator

De linkerzijbalk bevat drie inklapbare categorieën:

1. **Private collection**
   - toont alleen assets waarvan in de bibliotheek minstens één stuk werd geregistreerd;
   - toont per asset het aantal in bezit en het aantal dat momenteel in het ontwerp staat;
   - de collectie wordt lokaal in de browser opgeslagen.

2. **Castle pack**
   - bevat alle huidige structuurelementen en props;
   - de assets blijven hier onbeperkt beschikbaar om ontwerpen te maken.

3. **Dungeon pack**
   - is momenteel een lege placeholder;
   - assets met `pack: 'dungeon'` verschijnen hier automatisch.

## Bibliotheek

Klik bovenaan op **Bibliotheek**.

Per asset staat het aantal standaard op `0`. Gebruik de min- en plusknoppen of typ rechtstreeks een aantal. De waarden worden opgeslagen in `localStorage` onder:

```text
skaldis-build-private-collection-v1
```

Deze persoonlijke collectie is losgekoppeld van een opgeslagen ontwerp. Een ontwerpbestand bevat alleen de geplaatste onderdelen op het raster.

## Benodigde GLB-bestandsnamen

Plaats de modellen in de map `models`:

```text
models/
├── wall-straight-1.glb
├── wall-straight-2.glb
├── corner-1x1.glb
├── corner-2x2.glb
├── t-piece-1.glb
├── t-piece-2.glb
├── round-corner-1.glb
├── round-corner-2.glb
├── door-1.glb
├── door-2.glb
├── chest.glb
├── campfire.glb
└── action-figure.glb
```

Wanneer een bestand ontbreekt, blijft het onderdeel zichtbaar als een grijze placeholder.

## Referentiefiguur

De prop **Referentiefiguur** gebruikt:

```text
./models/action-figure.glb
```

Zonder model verschijnt een eenvoudige grijze humanoïde figuur op een ronde base. De webweergave schaalt een extern model tot maximaal ongeveer `1,25` rastereenheden hoog, ongeveer `31,75 mm` wanneer één rastereenheid `25,4 mm` voorstelt.

## Eigen modellen voorbereiden

Aanbevolen modeloriëntatie:

- lokale X-as: lengte;
- lokale Y-as: hoogte;
- lokale Z-as: diepte;
- onderkant van de geometrie rond Y = 0;
- geometrie zo dicht mogelijk rond het object-origin.

De configurator normaliseert de bounding box, schaalt het model en plaatst het in een aparte placement group op de juiste rasterpositie.

## Een asset aan Dungeon Pack toevoegen

Voeg in `PARTS` een nieuw onderdeel toe en gebruik:

```js
pack: 'dungeon'
```

Voorbeeld van een randstuk:

```js
'dungeon-wall-1': {
  name: 'Dungeon muur 1',
  pack: 'dungeon',
  category: 'structure',
  anchorType: 'edge',
  footprintEdges: ['H:0:0'],
  modelUrl: './models/dungeon-wall-1.glb',
  placeholder: 'straight'
}
```

Voorbeeld van een prop op een vak:

```js
'dungeon-altar': {
  name: 'Dungeon altaar',
  pack: 'dungeon',
  category: 'prop',
  anchorType: 'cell',
  footprintCells: ['0:0'],
  modelUrl: './models/dungeon-altar.glb',
  placeholder: 'chest'
}
```

Het onderdeel verschijnt vervolgens automatisch:

- onder Dungeon Pack in de configurator;
- onder Dungeon Pack in de bibliotheek;
- in Private collection zodra het aantal groter dan nul is.

## Projectbestanden

**Ontwerp opslaan** downloadt een JSON-bestand met:

- rasterinstellingen;
- onderdeel-ID;
- rasterpositie;
- rotatie.

**Ontwerp openen** valideert het bestand op onbekende onderdelen, overlap, rastergrenzen en geldige rotaties.
