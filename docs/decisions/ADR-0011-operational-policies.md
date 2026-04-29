---
id: ADR-0011
title: Operational Policies — Push, Branch, Test Validity
date: 2026-04-29
status: Active
deciders: Leader
category: process
supersedes: —
superseded_by: —
---

## Contesto

Tre policy operative sono rimaste implicite o ambigue dopo la promulgazione degli ADR fondativi 0001–0008:

1. **Push policy (P1).** ADR-0003 prescrive tag immutabili su GitHub come restore point ma non specifica quando il working branch viene pushato al remote. Se Claude esegue 5 commit certificati senza pushare, il backup remoto è stale e il restore point resta locale.

2. **Branch policy (P2).** Il sistema attuale assume commit diretti sulla branch `main`. ADR-0003 originariamente citava `master` (errata corrige in CHG-2026-04-29-002). Manca una decisione esplicita su: branch di feature? Pull request workflow? Branch protection? Quando si lavora direttamente su main vs su una feature branch?

3. **Test validity per infrastruttura (M8).** ADR-0002 elenca test "automatici e con esito positivo" come prerequisito di commit non-triviale, ma non chiarisce se i test manuali documentati (verifiche strutturali, syntax check con `bash -n`, ispezione manuale) sono validi per file di governance e infrastruttura. CHG-2026-04-29-001 ha usato de facto test manuali, sotto autorizzazione del Leader, ma senza copertura ADR esplicita. Va chiarito.

Queste policy non sono refusi né sezioni obsolete: sono **nuove decisioni** che integrano gli ADR esistenti. Vanno promulgate con un ADR proprio, non con errata corrige.

## Decisione

### Push Policy

**Regola fondamentale:** ogni commit certificato (ADR-0002 PASS + change doc + permesso Leader) viene pushato al remote `origin` immediatamente dopo il commit.

```bash
git commit ...           # locale
git push origin <branch> # immediato
```

**Motivazione:** in un contesto dove "un errore costa carissimo", la finestra fra commit locale e push è una zona di rischio (perdita filesystem, corruzione disco, lavoro non recuperabile). La chiusura immediata della finestra elimina il rischio senza overhead operativo.

**Eccezioni autorizzate dal Leader:**
- Lavoro esplicitamente off-line (Leader dichiara "non pushare fino a istruzione").
- Sequenza di commit di hardening atomici da pushare insieme a fine catena (Leader autorizza esplicitamente in apertura della sequenza).
- Rebase/cleanup pre-push su una catena di commit non ancora pushata (mai dopo il push).

In assenza di esenzione esplicita: **push immediato post-commit è il default**.

**Tag:** ogni tag annotato (checkpoint, milestone, manuale — ADR-0003) viene pushato esplicitamente:

```bash
git push origin <tag-name>
```

### Branch Policy

**Stato attuale (Fase Governance):** il progetto contiene solo file di governance e infrastruttura. La branch `main` è l'unica attiva. Tutti i commit avvengono direttamente su `main`.

**Stato futuro (Fase Codice Applicativo):** all'introduzione del primo modulo applicativo, questa policy sarà rivista con un ADR posteriore. Le opzioni preliminari includono:

- **Single-branch:** lavoro continuo su `main`, rebased rapidamente. Ammesso solo se il Leader è l'unico committer e il volume è basso.
- **Feature branch + PR:** branch `feat/<slug>` per ogni modifica non-triviale, PR su GitHub, merge solo dopo Leader approve.
- **Branch protection su main:** require linear history, require signed commits, require status checks (test gate ADR-0002).

La scelta è **rinviata** fino al primo ADR di stack tecnologico. Fino ad allora, vale la regola single-branch su `main`.

**Vincoli immediati su `main` (sempre attivi):**
- Mai `git push --force` su `main` senza autorizzazione esplicita del Leader documentata in change document.
- Mai `git reset --hard` su HEAD pubblicato (push effettuato) senza autorizzazione esplicita.
- Mai cancellazione di tag o branch remote senza autorizzazione esplicita.

### Test Validity per File di Governance e Infrastruttura

**Definizione.** Un **test manuale documentato** è un test eseguito da Claude o dal Leader che:

1. Verifica una proprietà strutturale o sintattica dell'artefatto modificato.
2. È documentato nella sezione `## Tests` del change document, con:
   - Comando o procedura eseguita (es. `bash -n scripts/hooks/pre-commit`)
   - Esito (PASS / FAIL)
   - Note se applicabile

**Ammissibilità.** I test manuali documentati sono **validi** ai fini del Test Gate (ADR-0002) **esclusivamente** per le seguenti categorie di file:

| Categoria | File / Pattern | Tipi di test ammessi |
|---|---|---|
| Hook git | `scripts/hooks/*` | Syntax check (`bash -n`), test di scenario manuale documentato |
| Setup script | `scripts/setup-hooks.sh`, simili | Syntax check, esecuzione test in clone temporaneo |
| ADR e governance | `docs/decisions/*.md`, `docs/changes/*.md` | Verifica strutturale (sezioni obbligatorie, frontmatter, link) |
| Indici | `INDEX.md`, `FILE-ADR-MAP.md` | Verifica di copertura e coerenza con i file referenziati |
| Documentazione | `CLAUDE.md`, `ROADMAP.md`, `CHANGELOG.md`, `STATUS.md` | Verifica di coerenza interna |
| Configurazione di repo | `.gitattributes`, `.gitignore` | Verifica di applicabilità (es. `git check-attr`) |

**Non ammissibilità.** I test manuali documentati **non sono validi** per:

- Codice applicativo (qualsiasi linguaggio, qualsiasi modulo eseguibile a runtime).
- Logica di business, algoritmi, trasformazioni di dati.
- Interfacce e contratti pubblici (API, schemi, eventi).

Per queste categorie, ADR-0002 nella sua forma originale rimane vincolante: serve un test automatico con esito PASS.

**Forma del Report.** Quando un commit usa test manuali documentati, il report al Leader (ADR-0002 step 4) include esplicitamente:

```
Tipo di test: manuali documentati (ADR-0011)
Comandi eseguiti: <lista>
Esito: PASS
Categoria file coperta: <da tabella sopra>
Rischi residui: <gap noti>
```

Il Leader mantiene il diritto di richiedere test automatici aggiuntivi anche per file di governance, in caso di dubbio.

## Conseguenze

- Il backup remoto è sempre allineato all'ultimo commit certificato.
- La branch policy è esplicita per la fase corrente e ha un punto di riconsiderazione chiaro (ADR di stack).
- Il Test Gate non è più ambiguo per file di governance: ammette esplicitamente test manuali documentati nelle categorie elencate, mantiene la regola stretta per il codice applicativo.
- Costo: 1 comando extra (`git push`) per commit; verbosity nel report al Leader per i test manuali (negligibile).

## Test di Conformità

| Scenario | Comportamento Atteso |
|---|---|
| Commit certificato senza push successivo | Anomalia, segnalare al Leader |
| `git push --force origin main` non autorizzato | Proibito, richiede autorizzazione esplicita |
| Modifica a hook senza test manuale documentato | Test Gate FAIL |
| Modifica a codice applicativo coperta solo da test manuale | Test Gate FAIL — serve test automatico |
| Modifica ADR coperta da verifica strutturale | Test Gate PASS (categoria ammessa) |

Verifica: log dei push (`git log origin/main..main` deve essere vuoto dopo ogni commit certificato + push).

## Cross-References

- ADR correlati: ADR-0002 (test gate, esteso da questo ADR), ADR-0003 (restore points, push integrato), ADR-0006 (hooks come categoria di file di governance)
- Governa: workflow di push, scelta di branch, definizione di test ammissibili per categoria di file
- Impatta: `CLAUDE.md` (workflow), `ROADMAP.md` (decisione futura su stack innesca branch policy v2)
- Test: verifica manuale post-sessione (catena commit/push allineata)
- Commits: [da aggiornare post-commit]

## Rollback

Se la regola di push immediato si rivela troppo rigida (errori di rete frequenti, sequenze di commit atomici interrotti), il Leader può ridefinire la policy con un ADR di supersessione. La policy di branch sarà comunque rivista all'introduzione del codice applicativo. Le regole di test validity sono modificabili solo via supersessione (sono di sostanza, non di forma).
