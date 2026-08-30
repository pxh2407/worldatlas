# World Atlas Explorer

Atlante interattivo del mondo (build **Vite/React**, router **wouter**), originariamente creato su Manus
(`https://worldatlas-dat5nx5u.manus.space/`) ed estratto come sito statico.

- **Online:** https://pxh2407.github.io/worldatlas/ (GitHub Pages, branch `main`)
- **Repo:** `pxh2407/worldatlas` · **Cartella locale:** `Desktop\CLAUDE\worldatlas`

## Struttura

```
index.html          pagina + tutte le personalizzazioni (script inline + <style>)
assets/index-*.js   bundle React compilato (con alcune patch, vedi sotto)
assets/index-*.css  CSS del bundle
manus-storage/      4 immagini (hero, grid, detail, mark)
favicon.ico
```

## Dati (caricati a runtime, NON nel repo)

L'app scarica i dati da dataset pubblici su GitHub — serve connessione internet:

- **Confini/mappa:** `datasets/geo-countries` (GeoJSON)
- **Dati Paesi:** `mledoze/countries` (nomi multilingua, capitali, valute, ecc.)
- **Popolazione:** NON è nei dataset → è una tabella statica `POP` (codice ISO3 → abitanti,
  fonte World Bank 2024 + integrazioni) **incorporata in `index.html`**.

## Personalizzazioni applicate

Quasi tutto è fatto **senza toccare il bundle**, tramite lo script inline e il `<style>` in `index.html`.
Le poche patch al bundle sono elencate nella sezione seguente.

- **Percorsi relativi** (`./assets/`, `./manus-storage/`) per funzionare in sottocartella.
- **Interruttore lingua IT/EN** (pulsante in alto a destra, default italiano; preferenza in
  `localStorage.wa_lang`). In IT intercetta `window.fetch` e localizza i dati; in EN un traduttore
  DOM (`MutationObserver` + dizionario) traduce le scritte fisse dell'interfaccia.
- **Continenti** completi (mappa `window.__waCont`) → il filtro per continente funziona per tutti i Paesi.
- **Popolazione** mostrata nel dossier, formattata (es. "59 Mln", "1,4 Mld").
- **Due blocchi dati nel dossier** iniettati via `MutationObserver` (osserva `childList`+`characterData`, l'app
  aggiorna il testo in-place) leggendo il codice in `.dossier-code`. Funzionano in IT ed EN; saltano ciò che manca.
  - **DEMOGRAFIA** (blocco `.wa-demo`): densità (calcolata al volo da popolazione/superficie, `window.__waDens`),
    aspettativa di vita (Banca Mondiale), ISU con categoria (ONU) — tabella `DEMO` (ISO3 → `[vita anni, ISU 0-1]`).
    Didascalia "Fonti: Banca Mondiale, ONU".
  - **ECONOMIA** (blocco `.wa-econ`): PIL, PIL pro capite, crescita PIL, inflazione, debito pubblico (% PIL),
    debito totale (valore assoluto = PIL × debito%/100, in $), deficit/PIL —
    tabella `ECON` (ISO3 → `[PIL mld $, pro capite $, debito % PIL, saldo % PIL, crescita %, inflazione %]`, fonte
    **FMI/WEO** ~2025, 197 Paesi). Deficit con segno e parola ("−3,1% (deficit)", "+1,3% (avanzo)"); didascalia
    "Stime FMI · anno" (2025 salvo eccezioni in `YR`: Eritrea 2019, Siria 2010, Sri Lanka 2024, Palestina 2024).
  - I Paesi non coperti dal FMI (Cuba, Corea del Nord, micro-territori) mostrano solo il blocco demografico.
    Tutti i dati sono tabelle statiche incorporate in `index.html` (nessuna chiamata a internet in più).
- **Bandiere vere** (`.wa-flag`): immagine SVG sopra il nome del Paese, iniettata leggendo `window.__waFlag`
  (cca3 → cca2 minuscolo, costruita in `loadCountries`). File locali in `flags/<cca2>.svg` (249 bandiere da
  flagcdn, ~4 MB) **committati nel repo** e inclusi anche nella versione offline → funzionano senza internet.
  Scelta locale (non emoji) perché Windows non disegna le bandiere-emoji.
- **Sfondo dossier bianco**: `aside.dossier { background:#fff }` e `aside.dossier::after { background-image:none }`
  (rimossa la texture della mappa color crema).
- **Confini della mappa = linea bianca sottile**: `.country-shape { stroke:#fff; stroke-width:0.75px;
  vector-effect:non-scaling-stroke }`. Il `non-scaling-stroke` è la chiave: prima il bordo era crema e lo
  spessore **cresceva con lo zoom** (a 150%+ le coste frastagliate diventavano una fascia bianca spessa, e nei
  Paesi piccoli il bordo copriva il colore lasciando solo un puntino). Ora è un'unica linea bianca uniforme a
  ogni ingrandimento, coste e confini interni compresi.
- **Nomi di oceani e mari** sulla mappa (`.wa-seas`): scritte SVG in blu-grigio corsivo (oceani font 11, mari 8)
  aggiunte da `addSeas()` nel gruppo SVG dei Paesi (seguono zoom/spostamenti, `pointer-events:none`). Posizioni
  calcolate con la proiezione Mercator del bundle: `x=1.445·lon+452.8`, `y=248−88.6·ln(tan(45+lat/2))` (calibrata
  sui centroidi dei Paesi). Testi IT/EN. 7 oceani (Pacifico ×2, Atlantico ×2, Indiano, Artico, Meridionale) + 5 mari.
- **Scroll alla mappa alla selezione**: quando cambi Paese, `scrollToMap()` (in `index.html`, chiamata da
  `inject()` al variare del codice, con `lastSel`) porta la `.map-frame` in vista con `scrollIntoView({block:"center"})`,
  così vedi tutta la mappa col Paese evidenziato. Sostituisce lo scroll fisso `top:260` del bundle (tolto, patch #11).
- **Freccia sul Paese selezionato** (`.wa-arrow`): quando selezioni un Paese, un piccolo triangolo stretto (contorno
  scuro `#1a2331`, interno trasparente, punta in giù) con la **punta sul bordo nord reale** del Paese vicino al centro
  orizzontale — calcolata campionando il path (`getPointAtLength`) per trovare il punto più alto della sagoma entro
  una fascia attorno al centro-X (così su forme irregolari come la Cina la punta non resta sospesa sopra il vuoto). Lo
  indica sulla mappa — così anche i Paesi minuscoli (Kosovo, Vaticano, Singapore…), che come selezione sono solo
  un puntino, si trovano subito. Iniettata in `updateArrow()` (nello script di `index.html`): trova il path
  `.country-shape.is-selected`, ne calcola il centro con `getBBox()` e ci mette sopra la freccia, come figlia
  dello stesso gruppo SVG dei Paesi (segue zoom e trascinamenti). NON tocca il bundle.
- **Leggibilità dossier**: testi ingranditi (etichette, valori, nota) via CSS.
- **Mappa**: zoom fisso a 150%, vista che mostra tutto il nord tagliando solo il fondo dell'Antartide;
  la rotella scorre la pagina (zoom con i pulsanti +/−); manina del cursore più piccola;
  casella di ricerca nascosta.
- **Testi**: titolo "Esplora il mondo. / Scopri ogni Paese."; sopra l'elenco "Scegli un Paese dall'elenco.".
- **Frecce per spostare la mappa** (`.wa-pad`): `addPad()` aggiunge in basso a destra sulla mappa un tastierino
  ↑↓←→; ogni clic sposta la mappa di **50 px fissi** (a scatti, istantaneo), alternativa rapida al trascinamento (lento sui PC datati
  perché ridisegna 258 Paesi a ogni movimento). `panStep()` simula un piccolo trascinamento (eventi Pointer sul
  `.world-map`, con `setPointerCapture` neutralizzato) così lo stato pan dell'app resta sincronizzato. Modello
  "cannocchiale": la freccia mostra quella direzione (↑=nord…). NON tocca il bundle.
- **Pulsante "Come funziona"**: `addHelp()` aggiunge in basso a **sinistra** (per non coprire le frecce del `.wa-pad`
  a destra) un pulsante (`.wa-help-btn`) che apre una finestra (`.wa-help-overlay`/`.wa-help-card`) con le istruzioni
  in 6 punti (scelta Paese, come vedere la scheda scorrendo sotto la mappa, zoom, spostamento con le frecce, lingua,
  nomi mari), IT/EN. Chiusura con ×, "Ho capito" o clic fuori.
- **Credito autore**: `addCredit()` aggiunge nell'intestazione, sotto "World Atlas", la riga "A cura di Filippo
  Nuccio Russo" (`.wa-credit`, IBM Plex Mono 8px, letter-spacing 0/word-spacing -1px, blu-ardesia #4a5462; in EN
  "Curated by ..."). Rimossa la scritta decorativa "EXPLORER / 01" (`.brand-subtitle { display:none }`).
- **Decori rimossi** (su richiesta, `display:none`): `.intro-art` (foto della vecchia mappa "PLATE 01 / ORBIS
  TERRARUM" + coordinate, nella pagina iniziale); `.dossier-image` (riquadro "FIELD NOTE / ..." in fondo al dossier);
  `aside.dossier .dossier-top` ("DOSSIER SELEZIONATO" + codice ISO) e `.dossier-compass` ("NOTA APERTA" + bussola)
  in cima al dossier. NB: il codice ISO resta nel DOM (nascosto) perché lo script lo legge per capire il Paese.
- **Nota di ripiego nascosta**: i Paesi senza descrizione curata mostravano la nota "Seleziona un Paese sulla
  mappa…"; in `inject()` la `.dossier-note` viene nascosta se inizia con quel testo (le descrizioni vere di
  Italia/Giappone/ecc. restano). Il pannello è anche nascosto del tutto nello stato vuoto (code "—").

## ⚠️ Patch al bundle da RIAPPLICARE se si riscarica il file da Manus

Se in futuro si sostituisce `assets/index-*.js` con una nuova build di Manus, riapplicare queste modifiche
(tutte le altre personalizzazioni sono in `index.html` e restano valide):

1. **Base del router:** `base:""` → `base:"/worldatlas"`
2. **Continenti:** `AE[l]||"Mondo"` → `AE[l]||(typeof window<"u"&&window.__waCont&&window.__waCont[l])||"Mondo"`
3. **Capitale (no ripiego a caso):** nella funzione `G`, `yt===wt||yt.includes(wt)||wt.includes(yt)` → `false`
4. **Rotella:** `onWheel:I=>{I.preventDefault(),` → `onWheel:I=>{if(!I.ctrlKey)return;I.preventDefault(),`
5. **Zoom default:** `[X,F]=_.useState(1)` → `[X,F]=_.useState(1.5)`
6. **Vista default:** `[P,Q]=_.useState({x:0,y:0})` → `[P,Q]=_.useState({x:-225,y:-80})`
7. **Reset vista:** `F(1),Q({x:0,y:0})` → `F(1.5),Q({x:-225,y:-80})`
8. **Didascalia zoom:** `ROTELLA PER ZOOM` → `PULSANTI + / - PER LO ZOOM`
9. **Testi:** "Traccia una rotta." → "Esplora il mondo."; "Apri un Paese." → "Scopri ogni Paese.";
   "Trova il tuo punto di partenza." → "Scegli un Paese dall'elenco."
10. **Elenco alfabetico:** nella `useMemo` della lista, dopo `...Vu(Rt)===u)})` e prima di `,[ct,c,u]`
    aggiungere `.sort((a,b)=>Pl(a).localeCompare(Pl(b)))` (ordina solo l'elenco `st`, non la mappa).
11. **Niente scroll al clic sulla lista:** nella `.country-row`, `onClick:()=>{w(I),window.scrollTo({top:260,behavior:"smooth"})}`
    → `onClick:()=>{w(I)}` (selezionare un Paese dall'elenco non fa più saltare la pagina).
12. **Passo dello zoom +/−:** i pulsanti erano `rt(X+.5)` / `rt(X-.5)` (±50% a clic) → `rt(X+.1)` / `rt(X-.1)`
    (±10% a clic, ingrandimento/riduzione più graduale).

## Aggiornare / pubblicare

Modificare i file in `Desktop\CLAUDE\worldatlas`, poi:

```bash
git add -A && git commit -m "..." && git push
```

Il push è già autenticato (Git Credential Manager con l'account `pxh2407`).
Dopo il push, GitHub Pages si aggiorna in ~1 minuto. Per vedere subito le modifiche nel browser:
**Ctrl + F5** (ricaricamento forzato).
