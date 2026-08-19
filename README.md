# Agorà

Sito statico: il marchio **Agorà** al centro e, in orbita attorno, i loghi degli
atenei italiani e le bandiere di regioni e capoluoghi. Nessun framework, nessuna
dipendenza da CDN.

```
index.html
assets/css/style.css
assets/js/orbits.js       motore delle orbite (vanilla JS, ~9 KB)
assets/js/intro.js        selettore: sceglie un'apertura, la mette in scena
assets/js/aperture/       le quattro aperture, una per file, come al loro commit
assets/img/agora-logo.png
assets/img/brandmark.svg  la "o" di Agorà come vettore autonomo
assets/img/brandmark.md   geometria misurata del brandmark, con i numeri
assets/loghi/             loghi degli atenei, originali, ottimizzati con svgo
project/                  bundle originale di Claude Design (sorgente del design)
chats/                    trascrizioni della progettazione
```

Per vederlo basta un server statico qualsiasi dalla radice del progetto:

```
python3 -m http.server 8000     # poi apri http://localhost:8000
```

## Come funziona la scena

Ogni token (chip con logo o bandiera) percorre un'ellisse inclinata di −12°. Le
ellissi sono concentriche, hanno la stessa inclinazione e "respirano" in fase:
i raggi si allontanano e si avvicinano insieme, quindi i tracciati non si
incrociano mai e le distanze relative restano costanti.

Il numero di anelli, i raggi, la dimensione dei chip e quanti token stanno su
ciascun anello vengono **ricalcolati a ogni resize** partendo dal viewport: la
stessa scena regge il desktop orizzontale e il telefono verticale senza rimpicciolire
tutto. Su desktop 1440×900 vengono fuori 3 orbite e 33 posizioni; su un telefono
390×844 due orbite (atenei fuori, bandiere dentro) e 16 posizioni.

Le sovrapposizioni non sono stimate a occhio. `fitsOnRing` e `ringsCompatible`
in `assets/js/orbits.js` verificano numericamente che due riquadri non si
incontrino mai, campionando tutte le posizioni relative possibili; il layout
scarta i raggi che non superano la verifica. Un'orbita che reggerebbe meno di
tre token viene eliminata.

Dove non c'è spazio per tutti — tipicamente su telefono — ogni posizione ospita
**più token a turno**: lo scambio avviene nel punto più lontano dell'orbita,
dove opacità e scala sono al minimo, con una breve dissolvenza. Così tutti e 13
gli atenei compaiono anche su uno schermo da 320 px.

Il velo dietro al marchio riprende esattamente lo stesso sfondo della scena e lo
sfuma con una maschera: i token che passano dietro spariscono prima di toccare
la scritta, e il velo stesso resta invisibile.

## Le aperture

Le aperture provate durante la progettazione sono **tutte in pagina**, e una
barra in alto passa da una all'altra. Non sono riscritture: il corpo di ogni
file in `assets/js/aperture/` è il codice del commit in cui quel design era in
produzione, riportato senza modifiche, e il markup del velo che si aspetta sta
nei `<template data-aperture="…">` di `index.html`, anch'esso identico
all'originale. `assets/js/intro.js` non anima nulla: sceglie un design, gli mette
in pagina il suo markup, lo avvia e costruisce la barra. Il cambio ricarica la
pagina — è esattamente quello che vede chi entra — e così il ciclo di animazione
del design precedente non resta a girare a vuoto. La scelta si ricorda in
`localStorage`; alla prima visita parte **Tunnel · 5 archi**.

| barra | commit | com'è |
| --- | --- | --- |
| **Marchio** | `271f3f4` | Fondo scuro, archi crema. Il marchio cresce fino alla sua misura, resta un istante, poi il vano della sua porta si allarga e scopre il sito. |
| **Sequenza** | `c37e886` | Fondo crema, porte nere. Le cinque porte partono una dopo l'altra, dalla più esterna alla più interna; l'ultima scopre il sito. |
| **Tunnel · 1 porta** | `8d11425` | Un tunnel di porte tutte della stessa forma: il vano del marchio ripetuto a distanze diverse. |
| **Tunnel · 5 archi** | `77bcd1d` | Il tunnel fatto da tutti e cinque gli archi del marchio, con la profondità ordinata perché la prospettiva funzioni. |

Che siano davvero le aperture di allora e non delle imitazioni è verificato in
quattro modi indipendenti: lo SHA-256 del corpo di ogni file coincide con quello
dell'`intro.js` del suo commit; lo SHA-256 di ogni `<template>` coincide con
quello del markup di allora; sostituendo `requestAnimationFrame` con una coda a
tempo deterministico, la geometria generata (tracciati, opacità, gradiente,
`viewBox`) è **identica in tutti i 157 fotogrammi**, a 900×600 e a 390×844, per
tutte e quattro; e gli screenshot presi a quel tempo deterministico coincidono
(differenza media 0,000%; restano dai 14 ai 29 pixel su 837 000, tutti
nell'antialiasing di un logo della pagina sotto, nessuno nell'apertura).

### Come funziona quella predefinita

All'ingresso si attraversa un **tunnel fatto dai cinque archi del marchio**. Su
fondo crema ogni porta arriva addosso — nasce piccola al centro, si allarga
accelerando ed esce dallo schermo — e le altre sono già in arrivo dietro di lei.
Poi si è fuori: il sito.

**L'ordine di profondità non è arbitrario.** Nel marchio il tratto è sempre
spesso circa 7,9 unità, quindi l'arco interno lo ha spesso rispetto al proprio
raggio (7,9 su 36) e quello esterno sottile (7,9 su 140). Poiché un tratto
spesso si legge come "vicino", la porta più vicina deve essere l'arco più interno
e la più lontana l'arco più esterno: così raggio e spessore calano insieme con la
distanza. Con l'ordine opposto la prospettiva risulta rovesciata e la porta
grande sembra la più lontana.

Tutte le porte nascono con lo stesso raggio in pixel e crescono alla stessa
velocità relativa: è la scala di ciascun arco a essere ricavata dal proprio
raggio, non il contrario. Così l'annidamento resta uniforme come in prospettiva
(rapporto circa 1,6 fra una porta e quella dietro) e ogni porta conserva le
proporzioni che ha nel logo.

Il sito si vede **in fondo al tunnel**: la finestra segue il vano della porta più
lontana, quindi i contenuti compaiono al centro, inquadrati dalle porte più
vicine, e crescono finché non si è usciti. Il bordo è sfumato invece che netto —
il vano dell'arco più esterno è una fessura bassa e larga, e un ritaglio secco su
quella forma lascerebbe un bordo visibile dove i chip vengono tagliati. La
trasparenza sta negli stop del gradiente del velo, non in una maschera SVG: una
maschera a schermo intero costa un ridisegno per fotogramma e dimezza il frame
rate.

Durata 2,1 s; un tocco, un tasto o uno scroll la accorciano (con un click
l'apertura finisce in circa 0,8 s), e la pagina torna interattiva appena la
finestra ha coperto lo schermo, senza aspettare la fine.

La geometria non è ridisegnata a occhio: è **misurata** dal logotipo originale
con precisione subpixel e riprodotta con archi di cerchio esatti — i numeri e il
metodo sono in `assets/img/brandmark.md`, il vettore in
`assets/img/brandmark.svg`.

Senza JavaScript, o con `prefers-reduced-motion: reduce`, l'apertura non esiste
affatto: la pagina si presenta già aperta. In quel caso non compare nemmeno la
barra, perché non c'è niente da mettere in scena.

## Loghi

Vengono usati **i file originali caricati dall'utente** (`project/uploads/loghi_universita/`),
senza alcuna modifica ai colori: sono passati solo da `svgo` (che rimuove metadati
e accorcia i tracciati) e pesano complessivamente 441 KB invece di 918 KB.

LUISS, La Statale e IULM sono marchi bianchi: sul chip chiaro sarebbero
invisibili. Invece di ricolorarli, **il contenitore diventa nero** (`.token--dark`)
— si adatta il chip, non il marchio.

## Bandiere

Le bandiere di regioni e capoluoghi restano su URL di Wikimedia Commons: non
esistono raccolte locali (né su npm né su GitHub) delle bandiere regionali
italiane. Se una non carica, il token viene rimosso dal giro e le posizioni si
ridistribuiscono, senza icone rotte.

Per averle in locale servono i file: mettili in `assets/bandiere/` e sostituisci
gli `src` in `index.html`.

## Accessibilità e prestazioni

- 60 fps stabili anche con CPU rallentata 6×, sia durante l'apertura sia in
  regime, con tutte e quattro le aperture (mediana a 16,7 ms; 95° percentile a
  16,8 ms passati i primi 300 ms). Un solo ciclo `requestAnimationFrame` che
  scrive esclusivamente `transform` e `opacity`, senza letture del layout.
- Resta **un solo fotogramma lungo** al primo disegno — circa 0,6 s su un
  telefono con CPU rallentata 6× — speso in parsing e decodifica delle immagini.
  C'era già prima del selettore, con gli stessi valori: i quattro file delle
  aperture (37 KB in tutto) non lo peggiorano in modo misurabile.
- I quattro tracciati delle orbite stanno in **un solo SVG**, senza
  `will-change`. Come quattro elementi con bordo ellittico e `will-change`
  erano quattro texture GPU grandi come lo schermo: bastava un velo sopra per
  far scendere il frame rate a 20 fps.
- L'animazione si ferma quando la scheda non è visibile.
- `prefers-reduced-motion: reduce` → scena statica, tutti i token al loro posto,
  nessuna apertura e nessuna barra.
- La barra del selettore va a capo su telefono verticale invece di scorrere di
  lato: a 320 px tutti e cinque i tasti restano visibili e cliccabili (26 px di
  altezza minima).
- Senza JavaScript la pagina resta leggibile: marchio in alto e loghi incolonnati sotto.
- 95 KB al primo caricamento su telefono (i token non ancora in scena sono lazy).

Se il server supporta gzip o brotli, i loghi SVG scendono da 441 KB a circa 110 KB:
vale la pena attivarli.

---

# Bundle originale (Claude Design)

`project/` e `chats/` sono l'export di Claude Design da cui è nata questa
implementazione e restano invariati come riferimento. `project/Agorà Orbite.dc.html`
è il prototipo: caricava React e ReactDOM da CDN più 69 KB di runtime, ed è la
ragione dei rallentamenti segnalati durante la progettazione.
