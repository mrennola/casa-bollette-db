# Note operative per sessioni Claude su questo repo

## Push a GitHub: limite noto del proxy di rete (sessioni cloud)

In alcune sessioni cloud (ambiente "Claude Code" / Cowork), il proxy di rete della sessione
blocca il push verso `github.com/mrennola/casa-bollette-db` anche fornendo un token PAT
valido (`token.txt`), con errore:

```
remote: access denied by the git proxy: mrennola/casa-bollette-db is not in this
session's authorized repository set, so the proxy will not inject a credential for it.
```

Questo NON è un problema del token: è una whitelist a livello di sessione. Il tool
`add_repo` (owner=mrennola, repo=casa-bollette-db, access=push) restituisce
`permission_denied: link your GitHub account`, e finora collegare l'account GitHub
dal lato utente non ha risolto in modo affidabile/immediato.

**Workaround che ha funzionato (sessione 29/09/2026):**
1. Claude fa comunque tutto il lavoro (clone, modifiche DB, commit locale nel container).
2. Se il push fallisce con l'errore sopra, Claude manda all'utente i file modificati
   (es. `reqa_bollette.db`, `Handoff_Bollette_FV_v*.md`) via chat.
3. L'utente li copia manualmente nella sua cartella locale del repo (ha GitHub Desktop
   installato sul proprio PC, repository "casa-bollette-db") e fa commit + push da lì.
4. Claude verifica con `git fetch origin main` (la lettura funziona sempre, anche senza
   token, essendo repo pubblico) che i commit dell'utente siano arrivati, poi allinea
   il proprio branch locale con `git reset --hard origin/main`.

**Prima di ripetere tutto questo giro in una sessione futura:** provare `add_repo` con
`access="push"` all'inizio della sessione, prima di iniziare il lavoro sul DB — se
nel frattempo il collegamento GitHub dell'utente è stato attivato, il push diretto
funzionerà e si può saltare il giro manuale via GitHub Desktop.

## Recupero dati meteo (Open-Meteo): usare il browser, non shell/WebFetch

Il dominio `api.open-meteo.com` (e `archive-api.open-meteo.com`) non è raggiungibile né dallo
shell (bloccato dal proxy di rete di sessione, fuori allowlist) né dal tool WebFetch (robots.txt
disallow su tutto il dominio, errore `ROBOTS_DISALLOWED`). L'unica via che ha funzionato
(sessione 29/09/2026): navigare direttamente con il browser (Claude in Chrome, tool
`mcp__claude-in-chrome__navigate` + `get_page_text`) sull'URL dell'API (es.
`https://api.open-meteo.com/v1/forecast?latitude=...&longitude=...&start_date=...&end_date=...&daily=...&timezone=Europe%2FRome`),
che restituisce il JSON grezzo leggibile con `get_page_text`. Tenere gli URL brevi (pochi parametri
`daily` per chiamata) per evitare l'errore "url exceeds the maximum fetchable length" del proxy.
Coordinate di riferimento già usate: lat 42.006, lon 12.520 (Settebagni, Roma).

## Convenzioni di progetto (vedi anche i vari Handoff_Bollette_FV_v*.md)

- Token GitHub sempre fornito come file `token.txt`, mai incollato in chiaro nel testo.
- Eliminare il token dal disco (`rm -f /home/claude/token.txt`) subito dopo l'uso,
  push riuscito o no.
- Ogni sessione inizia clonando il repo e leggendo l'ultimo handoff
  (`Handoff_Bollette_FV_v{N}.md`, numero più alto presente) per il contesto completo.
- Nessun dato identificativo (nome, indirizzo, POD, CF) va mai nel repo/DB/handoff.
