---
status: Iterating
owner: Leader
started: 2026-04-29
last_iteration: 2026-04-29
frozen_at: —
qa_rounds: 1
codename: TALOS
tagline: Scaler 500k
---

# PROJECT-RAW — Vision del Progetto: TALOS (Scaler 500k)

> **Documento governato da [ADR-0012](docs/decisions/ADR-0012-project-vision-capture.md).**
> Status corrente: `Iterating` — il Leader ha esposto la prima bozza completa il 2026-04-29; trascrizione integrale + raccolta lacune in corso.
>
> **Nota del Leader (verbatim, 2026-04-29):** *"Nessuna lacuna lasciata aperta. Il dominio funzionale e matematico è stato definito con rigore assoluto."*
>
> **Nota di Claude (ADR-0012 regola "Lacune Mai Completate"):** rispettando la disciplina del documento, sono state comunque marcate **24 lacune o ambiguità** — alcune critiche (formula VGP non specificata; il "monarca decisore" del sistema), altre di forma (apparente refuso R-08 ↔ R-09; sezione 15 saltata). La regola obbliga a marcare anche lacune che il Leader può ritenere chiare: chiusura nei round Q&A successivi.
>
> **Pipeline (ADR-0012):** Draft → **Iterating (qui)** → Frozen → proposta scomposizione (in chat) → validazione Leader → ADR di stack + ROADMAP.

---

## 1. Cosa è

**Talos non è un software gestionale.** È un **Hedge Fund algoritmico automatizzato applicato al modello FBA Wholesale High-Ticket**.

È un **motore finanziario e di compounding vettoriale** che:
1. Acquisisce listini fornitori,
2. Li epura dai rischi hardware tramite NLP,
3. Calcola l'allocazione perfetta di un budget liquido.

**Natura strettamente Stateless:** il bot non ha memoria degli stati temporali o logistici (scaglionamenti); riceve un budget di sessione e un listino fresco, restituendo un'analisi **"Just-in-Time"** ottimizzata per il momento esatto dell'esecuzione.

> [LACUNA L01 — IMPORTANTE]: la natura "Stateless" è dichiarata qui ma le sezioni 5 (Storico ordini, Plugin Panchina) e R-03 (ORDER-DRIVEN MEMORY) implicano persistenza. **Conferma necessaria:** lo "stateless" si riferisce all'analisi di sessione (input = listino fresco + budget; nessuna dipendenza da sessioni precedenti), mentre lo Storico Ordini e la Panchina sono persistenze di archivio/lookup mai usate come input causale dell'analisi corrente. Corretto?

---

## 2. Perché (Obiettivo di Business)

Per **scalare il capitale circolante da `x` (da definire) a 500k** annullando il **"Cash Drag"** e il **"Rischio Hardware"**.

L'essere umano non è in grado di calcolare a mente il **Bin Packing** su migliaia di referenze rispettando i vincoli logistici dei multipli di 5. La macchina sì.

> [LACUNA L02 — IMPORTANTE]: il Leader ha dichiarato esplicitamente "x da definire" come capitale di partenza. **Domanda:** qual è il valore corrente di partenza? Va inteso come "budget di apertura del primo scaglione di sessione" o come "capitale totale a disposizione su cui Talos opera ciclo dopo ciclo via R-07"? Influenza i criteri di completamento dell'MVP.

---

## 3. Per chi

### 3.1 Utenti target

Un **singolo utente:** il **CFO/Hedge Fund Manager** dell'operazione e-commerce. L'interfaccia è un **"cruscotto militare"**.

### 3.2 Stakeholder

| Stakeholder | Cosa riceve |
|---|---|
| **L'Erario / Commercialista** | Riceve dati in **Reverse Charge senza che Talos generi XML**. |
| **Fornitori** | Ricevono **ordini logistici inattaccabili in multipli di 5**. |

> [LACUNA L03 — IMPORTANTE]: "L'Erario/Commercialista riceve dati in Reverse Charge senza che Talos generi XML." **Domanda:** concretamente, cosa esce da Talos per il commercialista? Un export CSV/Excel della cronologia ordini? Un PDF report? Un'email automatizzata? Niente di automatico (solo storico interno consultabile)? Va capito perché tocca il bordo "out-of-scope fiscale" (sezione 7).

---

## 4. Cosa fa

### 4.1 Funzionalità chiave — Il Motore Fisico

#### 4.1.1 Fetch informazioni

Avviene tramite:
- **API di Keepa** (canale primario), oppure
- **Scansione di file JungleScout** in formato `csv` o `xlsx` (fallback).

Se si procede con il file di JungleScout, è necessario fare un **lookup del prodotto su Amazon** in base alla disponibilità mostrata nei listini fornitore somministrati.

> [LACUNA L08 — CRITICA]: il "lookup del prodotto su Amazon" come avviene tecnicamente? Scraping di amazon.it (Selenium / requests+bs4)? Product Advertising API di Amazon (PA-API 5)? Solo by ASIN o anche by titolo+EAN? **Bloccante per la scelta dello stack di networking + per i ToS Amazon.**

> [LACUNA L21 — CRITICA]: API Keepa: quale subscription level necessaria? Quali campi specifici servono (BuyBox corrente + storico, BSR, Fee FBA, Referral Fee, dimensioni/peso)? Stima costo mensile? Affidabilità dei limiti di rate?

#### 4.1.2 Estrattore di entità (Regex specializzato)

Idea plausibile per ottenere sempre match perfetti:

> Un **"estrattore di entità"** (basato su espressioni regolari o Regex) **specializzato esclusivamente su smartphone Samsung**. Il suo compito è prendere una stringa di testo disordinata (come il titolo di un listino o di Amazon) e strutturarla in dati precisi.

**Cosa estrae:**
- modello base (es. `S24`, `Z FOLD`)
- spazio di archiviazione in GB/TB (supportando termini internazionali come `SPEICHER` o `GO`)
- RAM
- connettività (`4G` o `5G`)
- colore (normalizzandolo in macro-famiglie)
- versione **Enterprise** (sì/no)

**La validazione ("Match Sicuro"):** una funzione confronta i dati estratti dal fornitore con quelli di Amazon applicando regole di business severissime. Esempio: se la memoria (ROM) differisce, o se uno dei due lati non la specifica, il sistema **blocca il match definendolo "AMBIGUO"**.

**L'eccezione intelligente — Whitelist 5G:** include una whitelist di modelli nati nativamente come 5G (es. serie S22-S26 e Z Fold/Flip). Se il fornitore non scrive "5G" per un Galaxy S24, lo script sa che è un'omissione innocua (**asimmetria benigna**) e fa passare il match.

> [LACUNA L06 — CRITICA]: "specializzato esclusivamente su smartphone Samsung". **Domanda:** Talos opera solo su Samsung anche in MVP, o l'MVP scarta a monte tutti i non-Samsung dal listino? E in roadmap futura: estensione ad altri brand (Apple, Xiaomi, Google, Huawei)? Il design del modulo Estrattore deve essere generalizzabile o monolitico Samsung?

#### 4.1.3 Filtro NLP (Kill-Switch)

**Smembramento stringhe testuali per RAM / ROM / Rete / Edizione.**

> [LACUNA L07 — IMPORTANTE]: "Filtro NLP (Kill-Switch)" e "Estrattore di entità (Regex)" del 4.1.2 — **sono due step diversi o lo stesso step con due nomi?** Dalla descrizione sembrano sovrapposti (estrazione → smembramento per validazione). Confermare l'architettura: 1 modulo o 2?

#### 4.1.4 VGP Score Monarchy (Calcolatore)

L'algoritmo di ranking è l'**unico ed esclusivo decisore**. L'analisi è **totalmente cieca**: ignora se un ASIN è un flagship o un accessorio; decide unicamente in base al **massimo coefficiente matematico di guadagno (VGP Score)**.

> [LACUNA L04 — CRITICA / BLOCCANTE]: la **formula del VGP Score non è stata definita** in nessun punto della bozza. È il "monarca decisore" del sistema, il cuore di Talos. Le formule 1-5 della sezione 6.3 coprono Cash Inflow, Cash Profit, Compounding, Quantità Target e Lotti, ma **non esiste formula `VGP = f(...)`. **Domanda:** qual è la formula esatta del VGP Score? Quali variabili coinvolge (Cash_Profit, Velocity, ROI, Volume Q_m, Costo)? Senza questa formula non è specificabile né testabile l'MVP.

#### 4.1.5 Reattività Vettoriale

Ricalcolo in tempo reale al mutare dello **slider Velocity Target**.

> [LACUNA L05 — IMPORTANTE]: lo "slider Velocity Target" è il controllo UI che modifica il numeratore della formula 4 (`Qty_Target = Q_m * (Velocity / 30)`)? Range del slider (es. 7-30 giorni)? Valore di default (la formula 4 fissa hardcoded a 15)? Granularità (1 giorno? 7? continuo?)?

#### 4.1.6 Allocatore Tetris

Funzione di saturazione del **budget di sessione**. Segue l'ordine decrescente di VGP per riempire il budget fornito dall'utente, **senza alcuna distinzione qualitativa tra i prodotti**. (Vedi R-06 per il dettaglio.)

#### 4.1.7 Formazione carrello

Generazione della lista della spesa sui criteri stabiliti. **Ogni ASIN di prodotto incasellato deve contenere un elemento ipertestuale cliccabile che punta direttamente alla rispettiva pagina prodotto su Amazon.it** (tramite ASIN). Scopo: debug per validazione manuale dei match e assegnazione dello stato di "match validato o meno".

#### 4.1.8 Esportatore Carrello

Generazione della lista della spesa **scaricabile, pronta per il fornitore**, se l'utente clicca il pulsante "ordina".

#### 4.1.9 Plugin Panchina (Archivio Opportunità)

Sezione di **stoccaggio dinamico** per ASIN validi ma non prioritari. Ogni articolo che:
- supera il **Kill-Switch hardware**
- supera il **Veto ROI** (citato come "R-09" nel testo originale del Leader, ma vedi LACUNA L09 sull'incongruenza con R-08)
- viene escluso dal carrello finale **per esaurimento del budget di sessione**

→ viene automaticamente archiviato qui.

La Panchina funge da **"lista d'attesa"** pronta per essere pescata in caso di aumento improvviso del budget o per la sessione di acquisto successiva.

#### 4.1.10 Storico ordini

I prodotti che hanno coinvolto ordini effettuati (messi in carrello, convalidati manualmente e poi esportati tramite pulsante "ordina") **non vanno dimenticati**. Sono memorizzati in una sezione apposita chiamata **"storico ordini"**, in una **lista a griglia** con i rispettivi:
- ASIN
- descrizione
- prezzo pagato
- quantità acquistate
- data

> [LACUNA L22 — FORMA]: Storico ordini è alimentato solo dall'azione "ordina" interna a Talos, oppure è anche sincronizzato da Amazon Seller Central? Se solo interno: come si gestiscono ordini effettuati fuori da Talos (manuali, retroattivi)?

### 4.2 Scenario d'uso primario

L'utente inserisce il listino e definisce il **Budget di Sessione** (es. 6.000€ per il primo scaglione logistico). Talos:
1. Analizza i dati di Keepa,
2. Ordina per **VGP Score**,
3. Applica la logica **Tetris** per saturare i resti,
4. Genera **due output distinti**:
   - **Il Carrello Definitivo** (saturato al 99.9%).
   - **La Panchina** (tutto il resto della selezione valida, ROI > 8%).

### 4.3 Modulo Universal Ingestion (Input Multi-formato)

Talos deve **annullare i tempi morti di inserimento dati**. L'interfaccia accetta listini in qualsiasi formato:

| Categoria | Formati | Logica |
|---|---|---|
| **Strutturati** | `XLSX`, `CSV` | Importazione diretta e immediata |
| **Non-strutturati** | `PDF`, `DOCS`, `TXT`, Immagini | Layer di estrazione intelligente (**OCR avanzato o Vision**) per ricostruire la tabella dati (ASIN, Titolo, Prezzo) e normalizzarla in un formato leggibile dal motore di calcolo prima di procedere con l'analisi VGP |

> [LACUNA L18 — CRITICA]: "OCR avanzato o Vision" — quale tecnologia concretamente? Tesseract (locale, open source, gratuito), GPT-4 Vision (API a pagamento), Claude Vision (API a pagamento), AWS Textract (a pagamento)? Decisione architettonica con impatto su costi operativi, latenza, dipendenze esterne.

> [LACUNA L19 — FORMA]: "DOCS" come formato non strutturato. Significa Microsoft Word `.docx`? Google Docs export? Apple Pages?

---

## 5. Cosa NON fa (Out-of-Scope)

| Out-of-scope | Note |
|---|---|
| Gestione Contabile / Fiscale (Fatturazione SDI) | — |
| Sincronizzazione Multicanale (stile Linnworks) | — |
| Pre-Accounting (collegamento a conti bancari / QuickBooks) | — |

---

## 6. Vincoli e requisiti

### 6.1 Vincoli tecnici

| Vincolo | Specifica |
|---|---|
| **Linguaggio** | Python 3.10+, architettura modulare |
| **Database** | PostgreSQL (Zero-Trust) via Docker |
| **Governance** | Git Hooks e check `CLAUDE.md` obbligatori |

> [LACUNA L15 — IMPORTANTE]: PostgreSQL **"Zero-Trust"** — significato concreto? Row-Level Security attiva, separazione dei ruoli/utenti, no superuser nel pool applicativo, audit log? Va specificato per l'ADR di stack DB.

> [LACUNA L16 — IMPORTANTE]: ORM o raw SQL? (SQLAlchemy / asyncpg / psycopg3 raw / DuckDB?) Test framework? (pytest / unittest / hypothesis?) Type checking? (mypy strict / pyright?) Linter? (ruff / black?) Sono decisioni che la governance attuale NON copre — diventeranno ADR di stack.

### 6.2 Le Leggi di Talos (Vincoli Business)

| ID | Nome | Regola |
|---|---|---|
| **R-01** | NO SILENT DROPS | Vietato usare `.drop()`; **ogni scarto deve essere loggato**. |
| **R-02** | BUDGET DINAMICO | Budget di sessione **sempre fornito dall'utente, mai hardcoded**. |
| **R-03** | ORDER-DRIVEN MEMORY | DB aggiornato **solo alla finalizzazione del carrello**. |
| **R-04** | MANUAL OVERRIDE | ASIN `locked_in` hanno **Priorità = ∞**. |
| **R-05** | KILL-SWITCH HARDWARE | Mismatch NLP **forza VGP a 0**. |
| **R-06** | TETRIS ALLOCATION | L'allocatore scorre la classifica VGP. Se un ASIN supera il budget residuo, prosegue (`continue`) cercando item con VGP inferiore ma costo compatibile, fino a saturare il budget di sessione al **99.9%**. |
| **R-07** | VAT CREDIT COMPOUNDING | **100% del bonifico Amazon è capitale reinvestibile.** |
| **R-08** | VETO ROI MINIMO | Nonostante la monarchia del VGP, viene applicato un **filtro di sbarramento assoluto**. Qualsiasi ASIN con un ROI stimato **inferiore all'8%** viene scartato a prescindere dal punteggio VGP. Protegge il capitale da oscillazioni di prezzo Amazon o fee impreviste, garantendo un margine di sicurezza minimo. |
| **R-09** | ARCHIVIAZIONE IN PANCHINA | Nessun ASIN con ROI ≥ 8% deve essere dimenticato. Al termine del ciclo di saturazione Tetris (R-06), il bot ha l'obbligo di generare un output secondario denominato **"Panchina"**. Questo file deve contenere tutti i prodotti idonei scartati unicamente per ragioni di **capienza finanziaria**, ordinati per VGP Score decrescente. |

> [LACUNA L09 — FORMA / DA CONFERMARE]: nel testo della funzionalità 4.1.9 (Plugin Panchina) il Leader cita: *"supera il Kill-Switch hardware e il **Veto ROI (R-09)**"*. Ma in "Le Leggi" R-09 è **ARCHIVIAZIONE** e R-08 è **VETO ROI**. Refuso o intenzionale? Confermare numerazione canonica: il Veto ROI è R-08?

> [LACUNA L09b — FORMA]: R-06 nel testo originale era citato in un punto come "Tetris" e la Panchina (R-09) cita "ciclo di saturazione Tetris (R-07)". R-07 in tabella è VAT CREDIT COMPOUNDING. Probabile altro refuso: R-09 punta a R-06 (Tetris), non R-07 (VAT). Confermare.

> [LACUNA L10 — IMPORTANTE]: la soglia **8% del Veto ROI (R-08)** è **hardcoded** o configurabile dal CFO via UI? Influenza la flessibilità del cruscotto militare e la struttura del config layer.

> [LACUNA L13 — IMPORTANTE]: **Manual Override (R-04)** — come funziona concretamente l'UI? Pulsante "Lock-in" sull'ASIN nella griglia? Edit diretto del DB? Lista di ASIN locked_in modificabile? Implica un'interfaccia pre-calcolo per "fissare" l'allocazione di alcuni ASIN prima del Tetris.

### 6.3 Formule Matematiche

> Nota del Leader (verbatim): *"Il budget di input per le analisi è dinamico e riferito alla singola sessione di acquisto. Non mi fido della memoria, quindi incido le formule nel documento raw."*

#### Formula 1 — Cash Inflow

```
Cash Inflow = BuyBox − Fee_FBA − (BuyBox * Referral_Fee)
```

**Nota del Leader:** *zero scorporo IVA per via del Reverse Charge + Credito infinito*.

> [LACUNA L11 — CRITICA]: `Fee_FBA` — come è ottenuta nella formula? Hardcoded? Lookup da Keepa API (campo `feeStorage`/`feeFBA`)? Da una tabella manuale per peso/dimensione? La FBA fee dipende dalla taxonomy Amazon (small/standard/oversize) e dalla stagione (Q4 surcharges).

> [LACUNA L12 — CRITICA]: `Referral_Fee` — come è ottenuta? Hardcoded a un valore (es. 8% per "Cell Phones & Accessories")? Configurabile per categoria? Lookup automatico dal nodo categoria Amazon? La % varia significativamente per categoria.

#### Formula 2 — Cash Profit

```
Cash Profit = Cash Inflow − Costo_Fornitore
```

#### Formula 3 — Compounding T+1

```
Budget_T+1 = Budget_T + Somma(Cash_Profit)
```

#### Formula 4 — Quantità Target a 15 giorni

```
Qty_Target = Q_m * (15 / 30)
```

> Nota del Leader: *"L'algoritmo non lavora su volumi generici, ma sulla capacità reale di assorbimento della BuyBox, limitando l'esposizione a una finestra di 15 giorni."*

##### Formula 4.A — Definizione della Quota Mensile (Q_m)

La Quota Mensile è la stima delle unità che spettano all'utente in base alla **concorrenza reale**. Non è il volume totale di Amazon, ma la "fetta di torta" competitiva.

| Variabile | Definizione |
|---|---|
| `V_tot` | Vendite totali mensili stimate per l'ASIN (Dati Keepa / BSR) |
| `S_comp` | Numero di venditori competitivi in BuyBox (entro il **2%** del prezzo minimo) |

```
Q_m = V_tot / (S_comp + 1)
```

> Nota del Leader: *"il +1 rappresenta l'ingresso dell'utente nella competizione."*

##### Formula 4.B — Target a 15 Giorni (Formula 4 modificata)

Talos **dimezza la Quota Mensile** per coprire esclusivamente un **Velocity Target di 15 giorni**, liberando l'altra metà del capitale per il Tetris.

#### Formula 5 — Quantità finale (lotti del fornitore)

```
Qty_Final = Floor(Qty_Target / 5) * 5
```

**Forzatura ai lotti del fornitore con arrotondamento sempre per difetto (Floor) per proteggere il cashflow.**

#### Formula MANCANTE — VGP Score

```
VGP = ???
```

> [LACUNA L04 — CRITICA / BLOCCANTE — qui in pari posizione delle altre formule]: la formula non è stata fornita. Bloccante per qualsiasi implementazione. Vedi 4.1.4.

### 6.4 Ambiente operativo

| Layer | Tecnologia |
|---|---|
| **Backend** | Container Docker locale (PostgreSQL + Python Data Science stack) |
| **Frontend** | **Streamlit** o **Gradio** (locale) per slider dinamici reattivi e tabellari |

> [LACUNA L14 — IMPORTANTE]: **Streamlit OR Gradio** — quale dei due, o decidiamo dopo (es. dopo prototipo MVP comparato)? La scelta impatta su: ciclo di rerun (Streamlit ha rerun completo a ogni interazione, Gradio ha event-driven), gestione dello stato (sessione), facilità di tabelle reattive grandi.

### 6.5 Vincoli business / tempo

> [LACUNA L02 — già marcata]: capitale di partenza `x` da definire.

Nessun deadline temporale dichiarato esplicitamente.

---

## 7. Successo misurabile

### 7.1 Criteri di completamento

> Nota del Leader (verbatim): *"Il software è pronto quando funziona senza nessun errore, in perfetto ordine e senza rischi di allucinazioni o compromessi sbrigativi di struttura."*

> [LACUNA L20 — CRITICA per definizione di "done"]: il criterio è qualitativo. Per un MVP serve una traduzione misurabile. **Domanda:** definiamo un test di accettazione concreto? Esempi proposti (da validare): suite pytest con copertura target X%; fixture di listino noto (es. 1.000 righe Samsung) con risultato VGP+Tetris atteso byte-exact; nessun `.drop()` (R-01) verificabile via grep statico nel codice; lint zero warning. Va deciso prima del Frozen.

### 7.2 Orizzonte di successo / MVP

> Nota del Leader (verbatim): *"Si intende la versione minima funzionante del software. Non serve che sia bello, serve che sia efficace. L'obiettivo dell'MVP è dimostrare che la pipeline 'Dato Fornitore → Analisi → Carrello' produce profitto reale senza errori di calcolo."*

**Definizione MVP:** acquisizione automatica del listino fornitore, arricchimento Keepa, filtraggio hardware e generazione del primo carrello secondo i criteri stabiliti.

> Osservazione: l'MVP è circolare con LACUNA L04 (formula VGP). Senza VGP non c'è "criterio stabilito" per generare il carrello. **Risolvere L04 è prerequisito di MVP definibile.**

---

## 8. Rischi noti

### 8.1 Rischi tecnici

| Rischio | Mitigazione dichiarata dal Leader |
|---|---|
| **Computazione del Bin Packing**: il ricalcolo al variare dello slider Velocity Target potrebbe causare colli di bottiglia nella RAM se il database sfiora le **10.000 righe** | **Vettorizzazione Numpy rigorosa** |

> [LACUNA L24 — IMPORTANTE]: rischi tecnici aggiuntivi non menzionati che vorrei segnalare per validazione:
> - **Throttling Keepa API** (limite chiamate/sec, costi a tier).
> - **Rotture scraping Amazon.it** se la strada è scraping (selettori HTML cambiano).
> - **Falsi positivi/negativi OCR** per file PDF deteriorati o scan a bassa risoluzione (sezione 4.3).
> - **Ambiguità persistenti** nel matcher Samsung che bloccano il flusso (gestione dell'eccezione "AMBIGUO" del 4.1.2).
> - **Drift dei dati Keepa** (BSR estimate non sempre allineata alla realtà → V_tot sbagliato → Q_m sbagliato → Qty_Final sbagliata → cashflow reale ≠ previsto).

### 8.2 Rischi non tecnici (sezione 15 — saltata nella bozza originale)

> [LACUNA L17 — IMPORTANTE]: la bozza originale del Leader **salta dal punto 14 al punto 16** (manca il 15). Probabile sezione "Rischi non tecnici". **Domanda:** sezione saltata volutamente o lacuna involontaria? Rischi candidati da validare:
> - **ToS Amazon** (scraping vietato, automazione del cart)
> - **ToS fornitori** (export ordini automatizzato — alcuni fornitori vietano)
> - **Normative privacy / GDPR** sui dati clienti contenuti nei listini
> - **Dipendenza da Keepa** (single-vendor: cambio prezzi, sparizione, throttling)
> - **Dipendenza da JungleScout** (idem, fallback)
> - **Compliance fiscale Reverse Charge** (verificare con commercialista che il flusso sia corretto)

---

## 9. Lacune Aperte

> Sezione **live** alimentata in `Iterating`. Scende monotonicamente verso 0 (o lacune accettate consapevolmente) prima del `Frozen`.

| # | Priorità | Lacuna | Sezione | Round | Status |
|---|---|---|---|---|---|
| **L04** | **CRITICA** | Formula del **VGP Score** non definita — è il monarca decisore del sistema | 4.1.4 / 6.3 | 1 | aperta |
| **L08** | **CRITICA** | Lookup Amazon (fallback JungleScout): scraping vs PA-API? | 4.1.1 | 1 | aperta |
| **L11** | **CRITICA** | `Fee_FBA`: hardcoded / Keepa / tabella manuale? | 6.3 (F1) | 1 | aperta |
| **L12** | **CRITICA** | `Referral_Fee`: hardcoded / configurabile / lookup? | 6.3 (F1) | 1 | aperta |
| **L18** | **CRITICA** | OCR/Vision: tecnologia concreta (Tesseract / GPT-4V / Claude V / Textract)? | 4.3 | 1 | aperta |
| **L20** | **CRITICA** | Criteri di completamento misurabili (traduzione del "funziona senza errori") | 7.1 | 1 | aperta |
| **L21** | **CRITICA** | Keepa: subscription, campi, costo, rate limit | 4.1.1 | 1 | aperta |
| **L06** | **CRITICA** | Estrattore Samsung-only: scope MVP e roadmap multi-brand | 4.1.2 | 1 | aperta |
| **L01** | IMPORTANTE | Stateless vs Storico Ordini/Panchina: confermare semantica | 1 | 1 | aperta |
| **L02** | IMPORTANTE | Capitale di partenza `x`: valore concreto | 2 | 1 | aperta |
| **L03** | IMPORTANTE | Cosa esce per il commercialista (formato/canale) | 3.2 | 1 | aperta |
| **L05** | IMPORTANTE | Slider Velocity Target: range, default, granularità | 4.1.5 | 1 | aperta |
| **L07** | IMPORTANTE | Filtro NLP vs Estrattore di entità: 1 modulo o 2? | 4.1.2-3 | 1 | aperta |
| **L10** | IMPORTANTE | Soglia 8% Veto ROI: hardcoded o configurabile? | 6.2 R-08 | 1 | aperta |
| **L13** | IMPORTANTE | Manual Override (R-04): UI/UX del lock-in | 6.2 R-04 | 1 | aperta |
| **L14** | IMPORTANTE | Streamlit vs Gradio: scelta o decisione differita? | 6.4 | 1 | aperta |
| **L15** | IMPORTANTE | PostgreSQL "Zero-Trust": RLS, ruoli, audit? | 6.1 | 1 | aperta |
| **L16** | IMPORTANTE | ORM, test framework, type checker, linter | 6.1 | 1 | aperta |
| **L17** | IMPORTANTE | Sezione 15 saltata: rischi non tecnici (ToS, GDPR, single-vendor) | 8.2 | 1 | aperta |
| **L24** | IMPORTANTE | Rischi tecnici aggiuntivi (throttling, drift Keepa, OCR fail) | 8.1 | 1 | aperta |
| **L09** | FORMA | R-08 vs R-09: refuso "Veto ROI (R-09)" — riferimento corretto è R-08? | 4.1.9 / 6.2 | 1 | aperta |
| **L09b** | FORMA | R-09 cita "ciclo di saturazione Tetris (R-07)" — il Tetris è R-06 | 6.2 R-09 | 1 | aperta |
| **L19** | FORMA | "DOCS" come formato: `.docx`? | 4.3 | 1 | aperta |
| **L22** | FORMA | Storico ordini: solo interno o sync da Seller Central? | 4.1.10 | 1 | aperta |

**Totale lacune aperte:** 24 (8 critiche, 12 importanti, 4 di forma — L09b conta separatamente da L09)

---

## 10. Q&A Log

> Cronologia append-only di tutte le domande poste da Claude e delle risposte del Leader durante `Iterating`. Mai cancellata. Ogni voce ha round, data, riferimento, esito.

### Round 1 — 2026-04-29 — Esposizione iniziale

| # | Tipo | Contenuto |
|---|---|---|
| 1.0 | Esposizione del Leader | Il Leader ha esposto la bozza completa di TALOS in un singolo blocco testuale strutturato in 14+1 sezioni (1-14, 16; sezione 15 omessa). Status del file passato `Draft → Iterating`. |
| 1.1 | Trascrizione | Claude ha trascritto verbatim il contenuto mappandolo sulle 11 sezioni del template ADR-0012, con sotto-sezioni 4.1.x, 6.x e 8.x per preservare la granularità della bozza. |
| 1.2 | Raccolta lacune | Claude ha identificato **24 lacune** in violazione apparente della dichiarazione del Leader ("nessuna lacuna aperta"), giustificate dalla regola "Lacune Mai Completate" di ADR-0012. Categorizzate in critiche/importanti/forma. |
| 1.3 | Lacune chiuse | 0. Tutte aperte, da chiudere nei round successivi. |

**Esito Round 1:** trascrizione integrale fatta, 24 lacune da risolvere. Il prossimo round è la prima tornata di Q&A (Round 2): il Leader risponde alle critiche prioritariamente.

---

## 11. Refs

> Riferimenti esterni (link, documenti, ispirazioni) che il Leader vuole conservare come contesto.

[da raccogliere — il Leader non ha citato riferimenti espliciti nella prima esposizione. Domande da porre nei round successivi:
- Dataset di esempio per il listino fornitore (un file reale anonimizzato)?
- Riferimenti Keepa API doc?
- Schema BSR/BuyBox di Amazon che il Leader vuole assumere come canonico?
- Riferimenti "Cash Drag", "VGP", "Hedge Fund applicato a FBA" — letteratura, articoli, sue note interne?]

---

## Cronologia Stati

| Data | Status | Evento |
|---|---|---|
| 2026-04-29 | Draft | File creato (CHG-2026-04-29-003, ratifica ADR-0012) — pronto per esposizione |
| 2026-04-29 | **Iterating** | Esposizione iniziale del Leader; trascrizione verbatim; 24 lacune raccolte (CHG-2026-04-29-004) |
