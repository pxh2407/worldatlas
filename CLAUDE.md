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
- **Leggibilità dossier**: testi ingranditi (etichette, valori, nota) via CSS.
- **Mappa**: zoom fisso a 150%, vista che mostra tutto il nord tagliando solo il fondo dell'Antartide;
  la rotella scorre la pagina (zoom con i pulsanti +/−); manina del cursore più piccola;
  casella di ricerca nascosta.
- **Testi**: titolo "Esplora il mondo. / Scopri ogni Paese."; sopra l'elenco "Scegli un Paese dall'elenco.".

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

## Aggiornare / pubblicare

Modificare i file in `Desktop\CLAUDE\worldatlas`, poi:

```bash
git add -A && git commit -m "..." && git push
```

Il push è già autenticato (Git Credential Manager con l'account `pxh2407`).
Dopo il push, GitHub Pages si aggiorna in ~1 minuto. Per vedere subito le modifiche nel browser:
**Ctrl + F5** (ricaricamento forzato).
