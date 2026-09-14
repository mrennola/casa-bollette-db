# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 17 — aggiornata al 14/09/2026

**Data:** 2026-09-14
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v17:**
1. **Agosto 2026 chiuso energeticamente su ENTRAMBE le fonti** (Tapo e inverter diretto, 31/31 giorni ciascuna) — ultimo tassello: screenshot del calendario mensile Tapo (l'utente non riusciva a scaricare l'export del mese precedente).
2. **Corretto il valore Tapo del 27/08**: era 2,45 kWh (parziale, export a metà giornata, già segnalato in v13); il valore definitivo è **3,279 kWh**. Il delta anomalo +42% segnalato in v16 è quindi risolto: non era una vera anomalia.
3. Nuova riga in `riconciliazione_mensile`: agosto, fonte Tapo, `definitivo 01-31/08 (v17)` — ora disponibile accanto alla riga fonte inverter già chiusa in v16.

---

## 1. Obiettivo del progetto

Ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo. (invariato)

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v17

| Tabella | Righe | Copertura |
|---|---:|---|
| `produzione_fv` | 6 | **riga agosto: stato passato da `da-aggiornare` a `confermato`** — entrambe le fonti coprono il mese intero |
| `produzione_giornaliera` | 165 | invariato in numero — **27-31/08 ora con Tapo aggiunto/corretto** |
| `verifica_bias_tapo_inverter` | **101** (era 96) | + 27, 28, 29, 30, 31/08 |
| `riconciliazione_mensile` | **13** (era 12) | + **agosto, fonte Tapo, definitivo (v17)** |
| Le altre tabelle | — | invariate da v16 |

---

## 4. NOVITÀ v17 — Agosto: entrambe le fonti ora complete (31/31 giorni)

| Fonte | Prod. FV | Autoconsumo | Consumo reale casa |
|---|---:|---:|---:|
| **Tapo** (v17, definitivo) | 115,823 kWh | 108,43 kWh | **674,53 kWh** |
| **Inverter diretto** (v15, definitivo) | 124,690 kWh | 117,30 kWh | **683,40 kWh** |

```
Prelievo rete (comune alle due fonti) = 566,10 kWh
Immissione (comune alle due fonti)    = 7,39 kWh
```

Le due stime del consumo reale differiscono di 8,87 kWh (+1,3%), interamente spiegati dal bias sistematico Tapo/inverter (~7-8%) già documentato da maggio. Nessuna delle due è "sbagliata" — riflettono lo scarto noto tra le due misure. Per un valore singolo di riferimento, resta valida la pratica già in uso nei mesi precedenti: **l'inverter diretto come fonte primaria** (misura hardware di primo livello), il Tapo come controllo secondario.

---

## 5. NOVITÀ v17 — Correzione 27/08 e chiusura del caso "anomalia risolta"

Il valore Tapo del 27/08 in DB era 2,452694 kWh, già segnalato fin da v13 come **parziale** (export effettuato alle 15:07, giorno ancora in corso). Lo screenshot del calendario mensile Tapo fornito dall'utente riporta il valore definitivo: **3,279 kWh**.

```
Bias vs inverter (3,49 kWh) = +6,43%  →  coerente col pattern sistematico
```

Questo chiude definitivamente il caso segnalato in v16 (delta +42%, lì già identificato come artefatto di tempistica e non come vera anomalia di produzione). Nessuna azione ulteriore necessaria su questo giorno.

**Riepilogo aggiornato dei casi anomali/particolari nella serie Tapo/inverter:**

| Data | Tipo | Stato |
|---|---|---|
| 09/05/2026 | Fonti in disaccordo, segno opposto | Mai risolta — probabile transizione cambio inverter |
| 27/08/2026 | Delta alto, stesso segno | **Risolto in v17** — era un valore Tapo parziale, ora corretto |
| 10/09/2026 | Entrambe le fonti basse, concordanti | Ancora aperta — calo di produzione reale, causa (meteo?) non verificabile |

---

## 6. Verifica di coerenza sui dati già presenti (01-26/08)

Lo screenshot copre tutti i 31 giorni di agosto; i valori per 01-26/08 sono stati confrontati con quelli già in DB (inseriti in sessioni precedenti da altri export Tapo) — **coincidenza esatta su tutti i 26 giorni**, nessuna discrepanza. Buon segnale di affidabilità incrociata sulla fonte Tapo.

---

## 7. Dati invariati da v16

Bollette, scheda EVT800, cronologia tecnica, stato pannelli Megasol — tutto invariato.

---

## 8. Pendenze aperte (aggiornate v17)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **agosto 2026** | Sorgenia (attesa metà settembre) | **Alta** | Ora il lato energetico è chiuso su entrambe le fonti — resta solo il conto economico |
| Dati meteo oltre il 04/08 | Open-Meteo (via browser utente) | Media | Utile per l'anomalia del 10/09 (ancora aperta) e per correlazione temperatura/prelievo agosto-settembre |
| Analisi economica sostituzione pannelli/inverter | — | Bassa | invariato |
| Rigenerare token GitHub | — | Consigliata | invariato |

**Azione a più alto impatto per la prossima sessione:** ingest bolletta Sorgenia agosto appena emessa — il lato energetico è ormai completo su entrambe le fonti disponibili.

---

## 9. Regole operative del progetto (invariate)

Tutte le regole precedenti restano valide e sono state applicate senza eccezioni.

---

## 10. Conclusione attuale del progetto

Agosto 2026 è ora **chiuso energeticamente su entrambe le fonti di produzione FV** (Tapo e inverter diretto, 31/31 giorni ciascuna), grazie a uno screenshot del calendario mensile Tapo fornito dall'utente in assenza della possibilità di esportare il mese precedente dall'app. Questo ha anche permesso di correggere un valore parziale (27/08) e chiudere in modo pulito il relativo caso segnalato in v16. Il consumo reale della casa ad agosto si attesta tra 674,5 kWh (fonte Tapo) e 683,4 kWh (fonte inverter), con la differenza interamente spiegata dal bias sistematico noto tra le due misure.

Resta aperto solo il lato economico (bolletta Sorgenia di agosto) e l'anomalia isolata del 10/09, non più legata a dubbi di affidabilità della fonte Tapo ma a una causa di produzione ancora da chiarire.

---
*Fine handoff v17. Prossima azione consigliata: (1) ingest bolletta Sorgenia agosto appena emessa; (2) recuperare dati meteo per verificare la causa dell'anomalia del 10/09.*
