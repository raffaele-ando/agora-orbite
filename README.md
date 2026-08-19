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
assets/loghi/             loghi degli atenei (originali + versioni corrette)
assets/loghi/fonti/       marchi scaricati, sorgente delle versioni corrette
strumenti/                rigenera le versioni corrette dei loghi
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

### Il fondo d'attesa

Il velo lo mette in pagina il selettore, che è uno script differito: per un
centinaio di millisecondi la pagina esisterebbe senza niente sopra, e si
vedrebbe il fallback — loghi e bandiere incolonnati — prima che l'apertura
parta. (Nelle versioni di un solo design il problema non c'era: il velo stava
scritto in `index.html`.)

Perciò `index.html` porta un `<div class="ground">` che copre lo schermo dal
primo disegno, e il blocco di script nell'`<head>` — che gira prima che il corpo
sia dipinto — gli dà il colore giusto leggendo la scelta da `localStorage`: nero
per «Marchio», crema per gli altri tre, perché il colore sbagliato sarebbe un
lampo di crema prima del nero. Il selettore lo rimuove **nello stesso
fotogramma** in cui il velo disegna la prima volta: `run()` ha già chiesto il
suo `requestAnimationFrame`, quindi quello del selettore arriva subito dopo, e
davanti non c'è mai né il fallback né un fondo pieno al posto del sito. Se il
selettore non arrivasse affatto — script bloccato, errore — un timeout di 8 s
toglie il fondo comunque, così la pagina non resta coperta.

Verificato sui fotogrammi veri del compositore (screencast CDP, non screenshot):
nelle otto combinazioni di quattro design × due viewport nessun fotogramma
mostra i chip del fallback, mentre la versione di prima ne mostra due.

## Loghi

I file caricati dall'utente (`project/uploads/loghi_universita/`) sono il punto
di partenza e restano nel repo intatti. Cinque però non andavano bene, e per
tre di loro il chip era stato fatto nero per rimediare. Ora **i chip sono tutti
bianchi** e i cinque file sono stati sostituiti o corretti. Le versioni nuove
stanno accanto agli originali, non al loro posto, e `strumenti/loghi-positivi.js`
le rigenera: rilanciandolo si riottengono gli stessi byte.

### Cosa non andava, e cosa c'è ora

| logo | il problema | come è stato risolto |
| --- | --- | --- |
| **UniTo** | era il **marchio vecchio**: sigillo nero e nome in nero. Dal giugno 2022 UniTo ha un marchio nuovo — sigillo rosso, nome in grigio con «UNI» e «TO» in rosso | `Unito-nuovo.svg`, dal marchio nuovo (export Illustrator, id `LOGO_UNITO_VERTICALE`, classi dichiarate `.st0{fill:#EA0029}` e `.st1{fill:#54565A}`). Nel file d'origine metà marchio non si vedeva: il `viewBox` era largo quanto il solo sigillo e un `clipPath` tagliava allo stesso riquadro. Misurato con `getBBox`, il disegno intero è 1334,6 × 543,2 |
| **LUISS** | era **bianco** (la versione per l'header scuro del sito) e con la cornice sbagliata: `width`/`height` dicevano 176×72 mentre il `viewBox` è 348×64, così nel chip risultava piccolo | `LUISS-ufficiale.svg`: colore **blu #003A70**, quello del file ufficiale `Luiss_ML_POS_COL_RGB.png` (POS COL = positivo colore); non il nero. Il disegno è lo stesso: rese alla stessa altezza, le due sagome si sovrappongono al **90,5 %** |
| **UniGe** | uno **scudo nero pieno**: il file usa le classi `st0`/`st1`/`st2` senza dichiararle, quindi tutto cadeva sul nero di default | `Unige-colore.svg`: il logotipo resta il vettore del file, al posto del disegno monocromatico va lo **stemma a colori** nella stessa casella (x −0,1 y 3,1 · 94,7 × 117, misurata con `getBBox`). I colori sono quelli del manuale: blu UniGe **#002677**, giallo **#F4DA40**, rosso **#C8102E**, grigio **#333333** |
| **La Statale** | negativo bianco, più le ombre di un mockup Sketch che su bianco si vedevano come un alone grigio | `Unimi-positivo.svg`: 31 riempimenti da `#FFF` a `#1D1D1B`, via i due `<use fill="#000">` con filtro e il filtro esterno. Che il positivo sia scuro lo conferma il sigillo ufficiale trovato su GitHub, nero su trasparente |
| **IULM** | disco bianco con «IULM» viola dentro e «università iulm» in bianco fuori: su bianco restava solo «IULM» | `IULM-positivo.png`: la scritta fuori dal disco passa al **viola del marchio stesso** (`#312783`, letto dal file); il disco diventa trasparente e l'immagine si ritaglia sul contenuto (1488×228 invece di 1600×836), altrimenti il chip dimensionava il logo su un disco invisibile |

### Gli altri otto

Polimi, Bocconi, Cattolica, Polito, Bicocca, Sapienza, UniSR e Humanitas
**restano i file originali, senza un pixel toccato**. Portano già i propri colori
dichiarati — #102C53, #0046AD, #00325C, #822433, #008FC4 + #C72635 + #F9C940,
#007953 — e vengono dagli header dei siti degli atenei, quindi sono la versione
in uso. Due verifiche fatte su tutti e tredici:

- **cornici coerenti**: `width`/`height` contro `viewBox`, perché il difetto di
  LUISS non fosse anche altrove. Nessun altro caso.
- **classi orfane**: `Polito.svg` (`cls-1`) e `Unito.svg` (`st0`, `st1`) usano
  classi non dichiarate come UniGe. Verificato disegnando le varianti: lì il
  nero di default è **giusto** — nel logo Polito il marchio è nero, e nel vecchio
  UniTo mettere `st1` bianco spezzava il logotipo in «UNI / TO». Il difetto
  visibile era solo quello di UniGe.

Su Bicocca resta un limite: il file è un raster in scala di grigi con
ventidue tonalità di grigio da compressione, non un vettore. Non ne ho trovato
uno migliore autentico (vedi sotto).

### Perché non i file scaricati dai siti ufficiali

L'uscita di rete di questo ambiente passa da un proxy che consente **solo GitHub
e il registro npm**. `upload.wikimedia.org`, `commons.wikimedia.org`, Brandfetch
e i siti degli atenei rispondono `403`, e lo stesso vale per il recupero di
pagine tramite gli strumenti dell'assistente: la ricerca sul web funziona (ed è
da lì che vengono i colori del manuale UniGe e la notizia del marchio UniTo
nuovo), ma i file no. Quello che si è potuto scaricare viene da repository
GitHub, ed è tenuto in `assets/loghi/fonti/`:

| file | da dove | cosa è |
| --- | --- | --- |
| `fonti/unito-marchio.svg` | `eduardz1/UniTO-typst-template` | il marchio UniTo nuovo, vettoriale, con i colori dichiarati |
| `fonti/unige-stemma.png` | `barbaLab/master-thesis-template` | lo stemma UniGe a colori, 786×968, palette identica a quella del manuale |

Serviti anche come riscontro, senza finire nel sito: `Luiss_ML_POS_COL_RGB.png`
(in `gianmarchioni/Luiss_thesis_template`) per il blu LUISS, e il sigillo della
Statale a 1200 px (in `GiacomoBotti/UniMI_template_pack`) per il nero della
Statale. Di IULM, Bicocca e Cattolica non esiste nulla di autentico raggiungibile.

I tredici file in pagina pesano 433 KB, 186 KB con gzip.

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
- Resta **un fotogramma lungo** al primo disegno, speso in parsing e decodifica:
  su un telefono con CPU rallentata 6× sta fra 33 e 233 ms, senza differenze
  sistematiche fra il selettore e le versioni di un solo design — i quattro file
  delle aperture (37 KB in tutto) non si sentono.
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
- Al primo caricamento arrivano **50 KB su telefono** e 148 KB su desktop (fino
  al momento in cui parte l'apertura): i token non ancora in scena sono `lazy`.
  Quando tutti e 33 sono passati davanti all'occhio si arriva a circa 630 KB,
  di cui 547 KB di immagini.

I 13 file dei loghi in pagina pesano 433 KB, che con gzip diventano 186 KB: se
il server sa comprimere, vale la pena attivarlo. Gli SVG sono la parte che ci
guadagna — i due PNG (Bicocca e IULM) sono già compressi.

---

# Bundle originale (Claude Design)

`project/` e `chats/` sono l'export di Claude Design da cui è nata questa
implementazione e restano invariati come riferimento. `project/Agorà Orbite.dc.html`
è il prototipo: caricava React e ReactDOM da CDN più 69 KB di runtime, ed è la
ragione dei rallentamenti segnalati durante la progettazione.
