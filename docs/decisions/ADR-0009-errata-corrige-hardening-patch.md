---
id: ADR-0009
title: Errata Corrige & Hardening Patch
date: 2026-04-29
status: Active
deciders: Leader
category: meta
supersedes: —
superseded_by: —
---

## Contesto

ADR-0001 stabilisce che "un ADR `Active` non viene mai modificato retroattivamente": ogni revisione richiede un nuovo ADR con `supersedes`. Questa regola è corretta per **decisioni di sostanza**, ma diventa un anti-pattern in due casi specifici:

1. **Refusi e incoerenze testuali** — un typo, un riferimento errato, un nome di branch sbagliato (`master` invece di `main`), un elenco numerico incompleto. Creare un ADR di supersessione per un refuso introduce rumore architetturale e nasconde correzioni triviali in atti di apparente gravità decisionale.

2. **Sezioni obsolete in ADR per il resto validi** — quando un ADR `Active` contiene una sezione superata da un ADR posteriore (es. ADR-0004 elenca un "Flusso di Re-Briefing" oggi sostituito dalla sequenza in ADR-0008), il resto dell'ADR è ancora normativo. Superseder l'ADR intero per modificare una sezione produce duplicazione documentale e perdita di tracciabilità storica.

L'audit di hardening della sessione 2026-04-29 ha rilevato:
- ADR-0003 cita la branch `master`; la branch reale del repo è `main` (refuso M1).
- CHG-2026-04-29-001 nel testo elenca "ADR 0001–0007" mentre nel frontmatter elenca correttamente fino a ADR-0008 (refuso M2).
- ADR-0004 sezione "Flusso di Re-Briefing" contraddice CLAUDE.md e ADR-0008 (sezione obsoleta B1).
- FILE-ADR-MAP marca `.gitattributes` "triviale; nessun ADR" mentre è materiale per ADR-0006 (refuso M5).

Serve un meccanismo formale per applicare correzioni mirate senza supersessione, mantenendo la disciplina di ADR-0001 per le decisioni reali.

## Decisione

### Definizione: Errata Corrige

Una **errata corrige** è una modifica a un ADR `Active` che soddisfa **tutte** le seguenti condizioni:

1. Non altera la decisione presa, le sue conseguenze, o i suoi test di conformità.
2. Corregge un refuso testuale (typo, nome errato, link rotto, elenco numerico) o un riferimento incoerente con altri ADR `Active` o con lo stato verificato del repository.
3. Lascia inalterate le sezioni `## Decisione` e `## Conseguenze` nel loro contenuto normativo.

Le errata corrige sono ammesse senza supersessione. Sono applicate via commit `docs(decisions):` e tracciate sia nel frontmatter dell'ADR modificato sia in una sezione `## Errata` aggiunta in coda all'ADR.

### Definizione: Hardening Patch

Una **hardening patch** è una modifica a un ADR `Active` che marca esplicitamente come **obsoleta** una sezione specifica, **senza** modificare il testo originale, e indirizza il lettore al testo corrente in un ADR posteriore.

La sezione obsoleta riceve un blocco di intestazione del tipo:

```markdown
> **⚠ Sezione superseduta da ADR-NNNN — non normativa.**
> Il flusso/regola corrente è definito in [ADR-NNNN sezione X](ADR-NNNN-slug.md#sezione-x).
> Testo originale conservato sotto per ragioni storiche.
```

Le hardening patch sono ammesse quando:
- La sezione da marcare contraddice un ADR `Active` posteriore.
- Il resto dell'ADR rimane normativo e non richiede revisione.
- L'ADR posteriore esiste già o viene introdotto contestualmente.

Le hardening patch sono applicate via commit `docs(decisions):` e registrate nella sezione `## Errata` dell'ADR modificato e nel frontmatter come `hardening_patches:`.

### Frontmatter Esteso

Il frontmatter degli ADR sottoposti a errata corrige o hardening patch riceve due campi opzionali:

```yaml
errata:
  - date: YYYY-MM-DD
    chg: CHG-YYYY-MM-DD-NNN
    summary: "breve descrizione del refuso corretto"
hardening_patches:
  - date: YYYY-MM-DD
    chg: CHG-YYYY-MM-DD-NNN
    section: "Titolo della sezione marcata obsoleta"
    superseded_by: ADR-NNNN
```

Questi campi sono **append-only**: ogni nuova errata o patch aggiunge una voce, mai sostituisce.

### Sezione `## Errata` (Append-Only)

In coda all'ADR, **dopo la sezione `## Rollback`**, gli ADR modificati ricevono:

```markdown
## Errata

### YYYY-MM-DD — CHG-YYYY-MM-DD-NNN
- **Tipo:** errata corrige | hardening patch
- **Modifica:** descrizione precisa di cosa è stato cambiato e dove
- **Motivo:** refuso / incoerenza con ADR-NNNN / contraddizione con stato del repo
- **Sostanza alterata:** No (errata) | Sezione X marcata obsoleta (hardening patch)
```

Ogni voce è permanente. La rimozione di una voce è proibita.

### Cosa NON è una Errata

Le seguenti modifiche **richiedono supersessione completa** via nuovo ADR (regola originale di ADR-0001):

- Cambio della decisione presa (`## Decisione` modificata nel suo contenuto normativo)
- Aggiunta di nuove regole vincolanti
- Cambio dei test di conformità
- Cambio delle conseguenze operative
- Cambio della categoria o dello status

Se in dubbio: **trattare come supersessione**. La soglia per qualificare una modifica come errata è alta.

### Hook Pre-Commit (Aggiornamento Necessario)

Il `pre-commit` hook (ADR-0006) deve essere aggiornato per riconoscere che un commit `docs(decisions):` che modifica un ADR `Active` è ammesso se:
- Il commit modifica solo la sezione `## Errata` e/o aggiunge campi `errata:`/`hardening_patches:` nel frontmatter, e/o aggiunge un blocco "Sezione superseduta" in cima a una sezione esistente, **oppure**
- Il commit aggiunge un nuovo ADR di supersessione (caso normale di ADR-0001).

L'enforcement di questa distinzione è demandato a verifica manuale pre-commit (sufficientemente raro da non richiedere automazione meccanica). Aggiornamento del hook tracciato in CHG-2026-04-29-002.

## Conseguenze

- I refusi vengono corretti senza inquinare il registro ADR con supersessioni inutili.
- Le sezioni obsolete sono marcate esplicitamente, eliminando il rischio che un futuro Claude le legga come normative.
- La storia di ogni modifica resta tracciabile dalla sezione `## Errata` di ogni ADR.
- ADR-0001 rimane intatto: la regola di non-modifica retroattiva per le decisioni di sostanza è preservata.
- Le errata corrige NON sono uno strumento per aggirare il processo decisionale; il loro scope è strettamente limitato ai casi sopra elencati.

## Test di Conformità

| Scenario | Comportamento Atteso |
|---|---|
| ADR con typo `master`/`main` | Errata corrige con sezione `## Errata` aggiornata |
| ADR con sezione contraddetta da ADR posteriore | Hardening patch con blocco "Sezione superseduta" |
| ADR con regola vincolante modificata | Supersessione completa (ADR-0001 originale) |
| Tentativo di errata su `## Decisione` | RIFIUTATO — richiede supersessione |
| Errata applicata senza voce in `## Errata` | RIFIUTATO in code review |

Verifica: per ogni ADR con `errata:` o `hardening_patches:` nel frontmatter, deve esistere una corrispondente voce nella sezione `## Errata`.

## Cross-References

- ADR correlati: ADR-0001 (regola originale di non-modifica), ADR-0004 (subisce hardening patch in CHG-2026-04-29-002), ADR-0006 (hook da aggiornare)
- Governa: meccanismo di modifica degli ADR `Active`, sezione `## Errata`, campi `errata:` e `hardening_patches:` nel frontmatter
- Impatta: ADR-0003, ADR-0004, ADR-0008 (potenziali soggetti a errata/patch in questa sessione e future), `scripts/hooks/pre-commit`
- Test: verifica strutturale manuale pre-commit
- Commits: [da aggiornare post-commit]

## Rollback

Se il meccanismo di errata corrige produce abusi (modifiche di sostanza camuffate da errata), si crea un ADR di supersessione che ripristina la regola stretta di ADR-0001 e converte tutte le errata pregresse in supersessioni retroattive.
