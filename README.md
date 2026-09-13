# Realm Gantt Studio

Tool di pianificazione **Gantt** per lo staff e il dev team di **Realm Reborn**.
Un solo file HTML, nessun server, nessun database, nessuna dipendenza esterna.

Asse temporale **continuo**: si zooma senza scatti dalla vista annuale fino al **quarto d'ora**,
per pianificare sia la roadmap a mesi sia le finestre operative notturne a ore.

![versione](https://img.shields.io/badge/versione-2.0-0070E0) ![licenza](https://img.shields.io/badge/uso-interno-E8B84B)

---

## Metterlo online (GitHub Pages)

1. Crea un repository, es. `realm-roadmap`.
2. Carica `index.html` nella root.
3. **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`**.
4. Dopo ~1 minuto il tool è raggiungibile su
   `https://<utente>.github.io/realm-roadmap/`

### Roadmap condivisa e sempre aggiornata

1. Nel tool: **Dati → Esporta → JSON**.
2. Committa il file nel repo con nome `gantt.json`.
3. Condividi il link `https://<utente>.github.io/realm-roadmap/?src=gantt.json`
   — chi lo apre vede sempre la versione committata.
4. Per aggiornare: modifica nel tool, riesporta il JSON, ricommittalo.

> `?src=` funziona solo via http/https (GitHub Pages, un web server locale).
> Aprendo `index.html` con doppio click il browser blocca la lettura del file per policy CORS: in quel caso usa **Dati → Importa**.

### Condivisione istantanea senza repo

**Dati → Link condivisibile** genera un URL che contiene l'intera roadmap compressa nell'hash.
Chi lo apre la vede in **sola lettura**, senza installare niente — comodo da incollare su Discord.
Se il link supera i ~7.500 caratteri conviene passare al metodo `?src=`.

---

## Zoom dinamico

L'asse non ha livelli fissi: è una scala continua in **pixel per ora**, da `0,012`
(circa 0,3 px al giorno, vista pluriennale) fino a `260` (oltre 4 px al minuto).
Intestazione, griglia e passo di trascinamento si riadattano da soli mentre zoomi.

| Gesto | Effetto |
|---|---|
| `Ctrl` / `Cmd` + rotellina | Zoom continuo **ancorato al punto sotto il cursore** |
| Pinch a due dita | Zoom continuo su touch e trackpad |
| `+` / `−` (pulsanti o tastiera) | Zoom a passi del 35% sul centro della vista |
| `⤢` oppure `0` | Adatta l'intera roadmap alla finestra |
| Preset **Minuti · Ore · Giorni · Settimane · Mesi · Anno** | Scorciatoie sulle scale più usate |
| `Shift` + rotellina | Scorrimento orizzontale |

Le unità dell'asse vengono scelte in automatico: quarti d'ora → mezz'ore → ore (1/3/6/12) →
giorni → settimane → mesi → trimestri, sempre con celle abbastanza larghe da restare leggibili.
La scala corrente è scritta nella barra di stato.

## Attività a giornate e attività a ore

Ogni attività può essere pianificata **a giornate** (default) o **a ore**, spuntando
*Pianificazione a ore* nella scheda.

- A giornate: date `YYYY-MM-DD`, il giorno di fine è incluso, il trascinamento va a passo di **1 giorno** a qualsiasi zoom.
- A ore: date `YYYY-MM-DDTHH:mm`, trascinamento a passo variabile (1 ora → 30 → 15 → 5 minuti man mano che zoomi).
- Nelle viste a ore la fascia **21:00 → 07:00** è ombreggiata, utile per manutenzioni, deploy ed eventi in-game.
- Le due modalità convivono nello stesso progetto e nello stesso file.

## Cosa fa

| Area | Funzioni |
|---|---|
| Pianificazione | Fasi collassabili, attività, milestone, dipendenze con frecce (rosse se il predecessore sfora) |
| Editing | Drag per spostare, maniglie per allungare, doppio click per la scheda completa, undo/redo |
| Team | Membri con colore e ruolo, assegnazione multipla, pannello **Carico del team** in giorni lavorativi |
| Viste | Zoom continuo anno → quarto d'ora, linea "adesso" con orario, weekend e notti evidenziati, tema scuro e chiaro |
| Filtri | Testo, membro, categoria, stato, solo in ritardo, solo milestone |
| Export | JSON, PNG, SVG, CSV (Excel), Markdown con blocco **Mermaid** che GitHub renderizza da solo, stampa/PDF |
| Multi-progetto | Più roadmap nello stesso file, duplicabili |

## Scorciatoie

| Tasto | Azione |
|---|---|
| `N` | Nuova attività |
| `+` / `−` | Zoom avanti / indietro |
| `0` | Adatta alla finestra |
| `T` | Vai a oggi |
| `Ctrl` + `Z` / `Ctrl` + `Shift` + `Z` | Annulla / ripeti |
| `Ctrl` + `S` | Esporta JSON |
| `Canc` | Elimina l'attività selezionata |
| `Ctrl` + rotellina | Zoom sul cursore |
| `Esc` | Chiudi finestre |

---

## Formato dati (`gantt.json`)

```jsonc
{
  "versione": 2,
  "progettoAttivo": "demo",
  "progetti": [{
    "id": "demo",
    "nome": "Roadmap Realm Reborn",
    "descrizione": "...",
    "owner": "Direzione tecnica",
    "versione": "v1.0",
    "team":       [{ "id": "u1", "nome": "Lead Dev", "ruolo": "...", "colore": "#0070E0" }],
    "categorie":  [{ "id": "eco", "nome": "Economia", "colore": "#E8B84B" }],
    "fasi":       [{ "id": "fa1", "nome": "Fase 1", "colore": "#0070E0", "chiusa": false }],
    "attivita": [{
      "id": "t01",
      "nome": "Audit risorse e resmon completo",
      "faseId": "fa1",
      "categoria": "inf",
      "assegnatari": ["u1", "u2"],
      "inizio": "2026-09-13",          // YYYY-MM-DD (giornate)
      "fine": "2026-09-17",             // oppure "2026-09-14T03:00" per le attività a ore
      "progresso": 60,                  // 0-100
      "priorita": "alta",               // bassa | media | alta | critica
      "stato": "in-corso",              // da-fare | in-corso | review | fatto | bloccato
      "milestone": false,
      "dipendenze": ["t00"],            // id che devono finire prima
      "note": "..."
    }]
  }]
}
```

Il file può essere scritto a mano o generato da script: campi mancanti vengono normalizzati all'apertura.

`inizio` e `fine` accettano due formati: `YYYY-MM-DD` per le attività a giornate (fine inclusa)
e `YYYY-MM-DDTHH:mm` per quelle a ore. Se una delle due porta l'orario e l'altra no, l'apertura
le allinea automaticamente. I file salvati con la versione 1 (solo date) si caricano senza modifiche.

---

## Note tecniche

- Dati salvati in `localStorage` del browser: **locali al singolo PC**. Il JSON nel repo è la fonte di verità condivisa.
- Nessuna chiamata di rete: funziona anche offline e dietro proxy.
- Testato su Chromium/Chrome, Firefox ed Edge recenti.
- Asse e griglia sono **virtualizzati**: anche al massimo zoom (canvas da centinaia di migliaia di pixel) vengono disegnate solo le celle visibili, così lo zoom resta fluido.
