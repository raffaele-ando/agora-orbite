# Agorà

Sito statico: il marchio **Agorà** al centro e, in orbita attorno, i loghi degli
atenei italiani e le bandiere di regioni e capoluoghi. Nessun framework, nessuna
dipendenza da CDN.

```
index.html
assets/css/style.css
assets/js/intro.js        apertura: la porta del brandmark scopre il sito
assets/js/orbits.js       motore delle orbite (vanilla JS, ~9 KB)
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

## L'apertura

All'ingresso si attraversa un **tunnel di porte**. Il marchio al centro di
Agorà è una pila di cinque archi concentrici, cioè cinque porte: su fondo crema
ognuna arriva addosso — nasce piccola al centro, si allarga accelerando ed esce
dallo schermo — e la successiva è già in arrivo dietro di lei. Poi si è fuori:
il sito.

Le cinque porte hanno **la stessa forma**, sono lo stesso oggetto a distanze
diverse, ed è questo che le fa leggere come un tunnel. Usare i cinque archi
diversi del marchio rovesciava la prospettiva: nel marchio il tratto è sempre
spesso circa 7,9 unità, quindi l'arco esterno lo ha sottile rispetto al proprio
raggio e quello interno spesso, e la porta più vicina finiva per sembrare la più
lontana. La forma usata è l'arco più interno, il vano vero del marchio, quello
con le gambe lunghe.

La porta più lontana — l'ultima partita — è quella che porta la finestra sul
sito: è il fondo del tunnel, e cresce mentre ci viene incontro finché non siamo
fuori. Il bordo della finestra coincide sempre col suo arco nero, che lo
nasconde: non si vede mai un taglio nel vuoto. Il fondo non è un crema piatto ma
la stessa sfumatura della scena, così nemmeno il velo si legge come un salto di
tono: resta visibile solo il nero delle porte.

Il rapporto di scala fra una porta e quella dietro di lei è circa 1,55: il
tunnel tiene in scena tutte e cinque le porte. Durata 2,1 s; un tocco, un tasto
o uno scroll la accorciano, e la pagina torna interattiva appena la finestra ha
coperto lo schermo, senza aspettare la fine.

La geometria non è ridisegnata a occhio: è **misurata** dal logotipo originale
con precisione subpixel e riprodotta con archi di cerchio esatti — i numeri e il
metodo sono in `assets/img/brandmark.md`, il vettore in
`assets/img/brandmark.svg`. L'animazione genera i tracciati da quella
geometria, quindi a qualunque scala la porta è quella del logo.

Senza JavaScript, o con `prefers-reduced-motion: reduce`, l'apertura non esiste
affatto: la pagina si presenta già aperta.

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
  regime (misurati a 1920×1080 con densità 2×: mediana e 95° percentile a
  16,7 ms). Un solo ciclo `requestAnimationFrame` che scrive esclusivamente
  `transform` e `opacity`, senza letture del layout.
- I quattro tracciati delle orbite stanno in **un solo SVG**, senza
  `will-change`. Come quattro elementi con bordo ellittico e `will-change`
  erano quattro texture GPU grandi come lo schermo: bastava un velo sopra per
  far scendere il frame rate a 20 fps.
- L'animazione si ferma quando la scheda non è visibile.
- `prefers-reduced-motion: reduce` → scena statica, tutti i token al loro posto.
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
