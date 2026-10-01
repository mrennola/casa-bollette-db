# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 22 — aggiornata al 01/10/2026

**Data:** 2026-10-01
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici

**Novità v22:**
1. **Raccomandazione contatore elettrico domotico** per monitoraggio bidirezionale prelievo/immissione a quadro generale: **Shelly Pro EM-50** (101,90€).
2. Confrontate e scartate due alternative (Shelly Pro 3EM-3CT63, Shelly EM Gen3) con motivazione.
3. Evento registrato in `cronologia_eventi` (rowid 19) — decisione di acquisto non ancora presa dall'utente, resta una pendenza aperta "bassa urgenza, alto interesse".
4. Nessuna modifica ai dati di consumo/produzione in questa sessione (handoff puramente di continuità, nessun nuovo dato caricato).

---

## 1. Obiettivo del progetto

Invariato: ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo.

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v22

| Tabella | Righe | Note v22 |
|---|---:|---|
| `cronologia_eventi` | **19** (era 18) | + 1 riga (01/10/2026, categoria `strumentazione`): raccomandazione contatore Shelly Pro EM-50 |
| Le altre tabelle | — | invariate da v21 |

---

## 4. NOVITÀ v22 — Contatore domotico per monitoraggio bidirezionale

Richiesta dell'utente: individuare il miglior contatore elettrico domotico per monitorare **sia il prelievo sia l'immissione** tra fotovoltaico e quadro generale, in sostituzione/affiancamento al Sonoff POWCT/eWeLink attuale.

### Opzioni confrontate (prezzi reali verificati, ottobre 2026)

| Modello | Prezzo | Fasi/canali | Accuratezza | Storico locale | Connettività | Note |
|---|---:|---|---|---|---|---|
| **Shelly Pro EM-50** ✅ raccomandato | 101,90€ | Monofase, 2 canali CT | Classe B ±1% | 60 gg a 1 min | Ethernet/Wi-Fi, Modbus/MQTT/HTTP/WS/RPC | Relè integrato; adatto esattamente a 2 punti di misura (generale + FV) |
| Shelly Pro 3EM-3CT63 | 130,90€ | Trifase, 3 canali CT | Classe B ±1% | 60 gg a 1 min | Ethernet/Wi-Fi, stesse interfacce | Capacità trifase non necessaria per utenza monofase — costo extra senza beneficio |
| Shelly EM Gen3 | 58,90€ | Monofase, 2 canali CT | ±2% | 10 gg | Solo Wi-Fi/Bluetooth, no Ethernet | Economico ma meno storico, meno precisione, meno stabilità di connessione |

### Raccomandazione

**Shelly Pro EM-50**, perché:
- Monofase con esattamente 2 canali CT, il numero di punti di misura necessari (linea generale + linea FV), senza pagare la capacità trifase del modello Pro 3EM che non serve.
- Precisione classe B ±1%, storico locale 60 giorni a risoluzione 1 minuto — sufficiente a rifare in autonomia simulazioni come quella fatta su 26-29/09/2026, su qualsiasi finestra recente, senza aspettare export manuali.
- Accesso locale nativo (Modbus/MQTT/HTTP/WebSocket/RPC) e integrazione Home Assistant, senza dipendenza dal cloud eWeLink attuale.
- Relè integrato, utile per eventuale automazione futura dei carichi.

### Installazione prevista

1. **CT1 sulla linea generale**, prima del punto di immissione FV → lettura prelievo/immissione a 4 quadranti.
2. **CT2 sulla linea di uscita dell'inverter EVT800** → lettura produzione lorda.

Con questi due punti si ottiene in tempo reale tutta la formula del progetto (prelievo, produzione, immissione/autoconsumo per differenza), senza passare per bolletta o export Tapo.

**Caveat dalla FAQ Shelly**: il totale "consumo" mostrato di default nell'app per un singolo CT bidirezionale può richiedere un piccolo script per nettare correttamente import/export — la lettura grezza a 4 quadranti resta comunque accurata; con Home Assistant si risolve con un sensore template.

**Stato:** solo raccomandazione, nessun acquisto ancora effettuato dall'utente. Registrato in `cronologia_eventi` come pendenza a bassa urgenza.

---

## 5. Dati invariati da v21

Tutto il resto — bollette, dati inverter/Tapo, simulazione stagionale 2026, caratterizzazione carichi stagionali, verifica margine corrente EVT800, ricerca accumulo — invariato. Vedi v21 per il dettaglio completo.

---

## 6. Pendenze aperte (aggiornate v22)

| Dato mancante / decisione | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **giugno 2026** — conto economico | Sorgenia | Alta | Invariato — unico mese pieno con piscina di cui manca ancora il lato € |
| Decisione acquisto contatore domotico (Shelly Pro EM-50) | — | Bassa | Raccomandazione data in questa sessione, decisione non ancora presa |
| Data accensione pompa di calore (riscaldamento) | Utente | Media | Necessaria per validare la proiezione di immissione quasi-zero nov-dic 2026 |
| Bolletta Plenitude **marzo-aprile 2026** (PDF originale) | Plenitude | Media | Ancora `stima` per lo split F2/F3 mensile |
| Verifica procedura/costi Areti-GSE per eventuale upgrade >800W | Areti/GSE | Bassa | Solo se l'utente decide di procedere — dati normativi non ancora verificati su fonti 2026 attuali |
| Monitoraggio margine di corrente EVT800 nei mesi ad alta irradianza | EnverView | Media | Nessun segnale di criticità nei giorni osservati finora |
| Bolletta Sorgenia **settembre 2026** | Sorgenia (attesa metà ottobre) | Bassa | Mese ancora in corso |
| Push commit v21 (`f6d41ac`) su GitHub | — | — | Ancora in attesa di conferma push manuale dall'utente (vedi sezione 7) |

**Azione a più alto impatto per la prossima sessione:** ingest bolletta Sorgenia giugno per chiudere l'ultima pendenza economica aperta.

---

## 7. Nota operativa — push GitHub ancora in sospeso

Il commit `f6d41ac` (handoff v21) risultava **non ancora pushato** su `origin/main` all'inizio di questa sessione (proxy di rete della sessione blocca il push, errore consueto "access denied by the git proxy"). Questo handoff v22 viene committato localmente nello stesso modo; i file aggiornati (`reqa_bollette.db`, `Handoff_Bollette_FV_v22.md`) vengono inviati all'utente per il push manuale via GitHub Desktop, secondo la procedura già documentata in `CLAUDE.md`.

---

## 8. Regole operative del progetto (invariate)

Tutte le regole precedenti restano valide. Vedi `CLAUDE.md` nel repo per le note operative su push GitHub e convenzioni.

---

## 9. Conclusione attuale del progetto

Nessuna novità sui dati di consumo/produzione in questa sessione. L'unico avanzamento è la chiusura della richiesta sul contatore domotico, con una raccomandazione concreta (Shelly Pro EM-50) basata su prezzi e specifiche reali verificate, confrontata con le alternative dirette. La decisione di acquisto resta all'utente; se e quando installato, il nuovo contatore permetterà di sostituire/affiancare il Sonoff POWCT attuale con dati a risoluzione più fine (1 minuto vs orario) e accesso locale diretto, utile per tutte le analisi orarie già impostate in questo progetto (es. simulazioni di capacità batteria).

Resta invariata la conclusione di fondo del progetto: il calo della bolletta è spiegato principalmente dal minor prelievo dovuto al fotovoltaico/autoconsumo (soprattutto in F1), non dal cambio fornitore, che ha un effetto secondario.

---
*Fine handoff v22. Prossima azione consigliata: (1) ingest bolletta Sorgenia giugno per l'ultima pendenza economica aperta; (2) se l'utente procede con l'acquisto del contatore, aggiornare l'handoff con la data di installazione e confrontare i primi dati con Sonoff POWCT; (3) attendere e registrare la data di accensione della pompa di calore per validare la proiezione invernale; (4) confermare il push manuale di questo commit e del precedente (v21) via GitHub Desktop.*
