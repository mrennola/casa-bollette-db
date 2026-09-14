# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 16 — aggiornata al 14/09/2026

**Data:** 2026-09-14
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v16:**
1. **Agosto 2026 chiuso energeticamente** (fonte inverter diretto, mese completo 31/31 giorni) — colmati gli ultimi 2 giorni mancanti (26-27/08) con 2 export EnverView "Daily Report".
2. Nuova riga in `riconciliazione_mensile`: **agosto, fonte inverter diretto, versione "definitivo 01-31/08 (v15)"**.
3. Chiarito un secondo caso di delta anomalo (27/08, +42% inverter vs Tapo): **non è produzione reale**, è un artefatto — il valore Tapo di quel giorno era parziale (export a metà giornata, già segnalato in v13).

---

## 1. Obiettivo del progetto

Ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo. (invariato)

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v16

| Tabella | Righe | Copertura |
|---|---:|---|
| `bollette` | 10 | invariato |
| `produzione_fv` | 6 | invariato in numero; **riga agosto aggiornata** (mese completo lato inverter) |
| `produzione_giornaliera` | 165 | invariato in numero — **26-27/08 ora con doppia fonte** |
| `letture_inverter_orarie` | **109.182** (era 107.586) | 09/05 → 13/09/2026, **ora continua senza buchi** |
| `letture_quadro` | 6.360 | invariato |
| `verifica_bias_tapo_inverter` | **96** (era 94) | + 26/08, 27/08 |
| `riconciliazione_mensile` | **12** (era 9) | + **agosto, fonte inverter, definitivo** |
| `cronologia_eventi` | 16 | invariato |
| `meteo_giornaliero` / `meteo_orario` | 65 / 1.560 | invariato — fermi al 04/08 |
| `fornitori_storico` | 4 | invariato |
| `pannelli_fv_specifiche` | 2 | invariato |

---

## 4. NOVITÀ v16 — Chiusura energetica di agosto 2026 (fonte inverter)

Con gli ultimi 2 giorni (26-27/08), l'inverter diretto copre **tutto il mese di agosto per la prima volta**:

| | Valore |
|---|---:|
| Prelievo rete (eWeLink, 31/31 gg) | 566,10 kWh |
| Produzione FV (inverter diretto, 31/31 gg) | **124,69 kWh** |
| Immissione (eWeLink, 31/31 gg) | 7,39 kWh |
| Autoconsumo FV | 117,30 kWh |
| **Consumo reale casa** | **683,40 kWh** |

```
Autoconsumo FV     = 124,69 - 7,39 = 117,30 kWh
Consumo reale casa = 566,10 + 117,30 = 683,40 kWh
```

Salvato in `riconciliazione_mensile` come versione `definitivo 01-31/08 (v15)`, fonte "Inverter diretto". La fonte Tapo per lo stesso mese resta invece parziale (27/31 gg, mancano ancora 28-31/08) e la sua riga di riconciliazione (v13, preliminare 01-26/08) non è stata toccata — non comparabile 1:1 col dato inverter finché non si completa.

**Nota di lettura importante:** questo dato include il periodo vacanza 17-22/08 (prelievo -80,5% vs media del mese, casa vuota, già in `cronologia_eventi`), che abbassa artificialmente la media mensile rispetto a un mese con la casa sempre occupata. Va tenuto presente confrontando agosto con altri mesi.

### Confronto con i mesi precedenti (fonte prelievo, sempre eWeLink/bolletta)

| Mese | Prelievo | Nota |
|---|---:|---|
| Giugno 2026 | 344,50 kWh (bolletta) | — |
| Luglio 2026 | 621,10 kWh (bolletta) | picco climatizzazione |
| **Agosto 2026** | **566,10 kWh** (eWeLink, bolletta non ancora emessa) | -8,9% vs luglio, ma include 6 giorni di vacanza |

---

## 5. NOVITÀ v16 — Secondo caso di delta "anomalo" chiarito (27/08)

Il delta Tapo/inverter del 27/08 risultava +42,29% (Tapo 2,45 kWh vs inverter 3,49 kWh) — molto oltre il bias sistematico ~7%. Causa identificata, **non è un'anomalia di produzione**: il valore Tapo di quel giorno era già segnalato in v13 come parziale (export effettuato alle 15:07, giorno ancora in corso). L'inverter, invece, copre l'intera giornata. Annotato in `verifica_bias_tapo_inverter` per non confondere questo caso con una vera anomalia di produzione come il 10/09/2026 (dove invece le due fonti concordavano su un calo reale).

**Riepilogo dei due casi anomali nella tabella `verifica_bias_tapo_inverter`, per chiarezza:**

| Data | Tipo | Causa |
|---|---|---|
| 09/05/2026 | Fonti in disaccordo, segno opposto | Probabile transizione cambio inverter EVT800 — mai risolta |
| **27/08/2026** | Delta molto alto, stesso segno | **Tapo parziale (export a metà giornata)** — non è un'anomalia di produzione |
| 10/09/2026 | Entrambe le fonti basse, concordanti | Calo di produzione reale, causa (meteo?) non verificabile per mancanza di dati meteo |

---

## 6. Dati invariati da v15

Buco Tapo 28-31/08 ancora aperto (nessun file disponibile), scheda EVT800, bollette, cronologia tecnica, stato pannelli Megasol — tutto invariato. Vedi v14/v15 per il dettaglio.

---

## 7. Pendenze aperte (aggiornate v16)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Produzione FV **Tapo 28-31/08** | Tapo, foglio Mese | Bassa-Media | Non più bloccante: agosto è già chiuso lato inverter. Utile solo per completare anche la serie Tapo e ricalcolare il bias su tutto il mese |
| Bolletta Sorgenia **agosto 2026** | Sorgenia (attesa metà settembre) | **Alta** | Ora il confronto avrà un dato energetico (consumo reale 683,40 kWh) già pronto per il conto economico |
| Dati meteo oltre il 04/08 | Open-Meteo (via browser utente) | Media | Utile per l'anomalia del 10/09 e per correlazione temperatura/prelievo agosto-settembre |
| Analisi economica sostituzione pannelli/inverter | — | Bassa | invariato |
| Rigenerare token GitHub | — | Consigliata | invariato |

**Azione a più alto impatto per la prossima sessione:** ingest bolletta Sorgenia agosto appena emessa — ora il lato energetico è già completo e pronto per il confronto economico.

---

## 8. Regole operative del progetto (invariate)

Tutte le regole precedenti restano valide e sono state applicate senza eccezioni.

---

## 9. Conclusione attuale del progetto

Agosto 2026 è ora **chiuso dal lato energetico**: prelievo, immissione e produzione FV (fonte inverter) coprono tutti e 31 i giorni del mese. Il consumo reale della casa è stimato in 683,40 kWh, con un autoconsumo FV di 117,30 kWh — un dato che va letto tenendo conto dei 6 giorni di vacanza a metà mese, che abbassano la media rispetto a un mese pienamente occupato. Resta aperto solo il lato economico (bolletta Sorgenia di agosto, attesa a metà settembre) e il completamento della serie Tapo (non più bloccante).

Chiarito anche un secondo caso di delta anomalo (27/08): a differenza del 10/09 (calo di produzione reale, confermato su due fonti), il 27/08 era solo un artefatto di tempistica dell'export Tapo — utile distinzione per non confondere i due tipi di segnale in futuro.

---
*Fine handoff v16. Prossima azione consigliata: (1) ingest bolletta Sorgenia agosto appena emessa; (2) opzionale — completare la serie Tapo 28-31/08; (3) recuperare dati meteo per verificare la causa dell'anomalia del 10/09.*
