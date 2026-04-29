# ROADMAP

Tracker operativo del progetto. Ogni voce deve essere tracciabile a un ADR validato e ratificato dal Leader.

> **Regola vincolante:** Nessuna modifica architetturale viene registrata in questo documento prima di essere stata ratificata dal Leader come ADR in `docs/decisions/`. La validazione GitNexus è opzionale fintanto che ISS-001 è aperta (vedi STATUS.md).

---

## Obiettivi Attuali

| # | Obiettivo | ADR di riferimento | Stato |
|---|-----------|-------------------|-------|
| 1 | Inizializzazione infrastruttura dogmatica | — | Completato |
| 2 | Promulgazione ADR fondativi (0001–0004) | ADR-0001–0004 | Completato |
| 3 | Promulgazione ADR enforcement + anti-allucinazione (0005–0008) | ADR-0005–0008 | Completato |
| 4 | Hardening governance v0.5.0 — fix audit (B1–B5, M1–M9, P1–P3) | ADR-0009, ADR-0010, ADR-0011 | Completato |
| 5 | Verdetto: sistema governance a prova di bomba per fase pre-codice | tutti 0001–0011 | Pronto — in attesa di approvazione Leader del CHG-002 |
| 6 | Fork repo su PC operativo Leader + verifica `gitnexus analyze` (ISS-001) | ADR-0007 | Rinviato — bloccato da setup PC operativo |
| 7 | Definizione stack tecnologico (ISS-002) → primo ADR di architettura | Da promulgare | Bloccante per fase codice |

---

## Implementazioni in Corso

_Nessuna implementazione di codice attiva al momento. Sessione 2026-04-29: hardening governance — in attesa approvazione Leader per il commit di chiusura._

---

## Meta-Blocchi Futuri

_Decisioni architetturali future da discutere e formalizzare tramite ADR prima dell'implementazione._

| # | Tema | ADR necessario | Note |
|---|------|---------------|------|
| A | Stack tecnologico | Da promulgare | Da definire dal Leader (fonte ISS-002) |
| B | Struttura directory codice applicativo | Da promulgare | Dipende dallo stack |
| C | CI/CD pipeline | Da promulgare | Dipende dallo stack |
| D | Branch policy v2 (multi-branch / PR / branch protection) | Da promulgare | Rinviata da ADR-0011; rivedere all'introduzione del primo modulo applicativo |
| E | GitNexus operativo da PC del Leader | ADR-0007 (esistente, in attesa) | Eseguire `gitnexus analyze` post-fork |

---

## Log delle Validazioni

| Data | Modifica | ADR | Validato da |
|------|----------|-----|-------------|
| 2026-04-29 | Inizializzazione infrastruttura | — | Leader |
| 2026-04-29 | Promulgazione ADR fondativi 0001–0004 + protocolli operativi | ADR-0001–0004 | Leader |
| 2026-04-29 | Promulgazione ADR 0005–0008 + git hooks + enforcement + anti-allucinazione | ADR-0005–0008 | Leader |
| 2026-04-29 | Hardening governance v0.5.0 — ADR-0009/0010/0011 + errata + hardening patch + hook rinforzati | ADR-0009, ADR-0010, ADR-0011 | In attesa di approvazione Leader (CHG-2026-04-29-002) |
