# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 13 — aggiornata al 27/08/2026

**Data:** 2026-08-27
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v13:**
1. **Bolletta Sorgenia luglio 2026 ricevuta e ingerita** (621,1 kWh, 214,57€ + 9€ canone = 223,57€) — chiusura definitiva di luglio, con causa dell'aumento (climatizzazione + ondata di calore) già identificata in sessione precedente e ora tradotta in conto economico.
2. **Produzione FV agosto quasi completata**: Tapo 01-27/08 (27/31 gg, giorno 27 parziale), inverter diretto 01-25/08 (25/31 gg). Bias inverter/Tapo confermato ancora a +7,8%.
3. **Prelievo/immissione agosto completato** via nuovo export eWeLink storico (19/02→27/08), colmando il buco 05/08-27/08. Riconciliazione preliminare 01-26/08 fatta.
4. **Periodo vacanza 17-22/08 identificato e confermato dall'utente**: prelievo -80,5% rispetto alla media del mese, immissione in aumento (FV normale, semplicemente non autoconsumato). Registrato in `cronologia_eventi`.
5. **Chiarito lo stato della sostituzione pannelli bifacciali Megasol** (annunciata 04/08): confermato dall'utente che il lavoro **non è ancora stato eseguito**. Verificato anche sui dati (nessun salto nei picchi di potenza giornalieri, sempre 310-400W/pannello, range identico ai vecchi Dahai).

---

## 1. Obiettivo del progetto

Ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo.

**Domanda guida:**
```
Quanto del risparmio deriva dal fotovoltaico/autoconsumo
e quanto dal cambio fornitore?
```

Distinzione metodologica sempre obbligatoria:
- **Effetto kWh** = riduzione del prelievo dalla rete (fotovoltaico/autoconsumo)
- **Effetto tariffa** = risparmio/aggravio dal cambio fornitore o dalle oscillazioni PUN
- **Effetto piscina** = da giugno 2026, nuovo carico diurno
- **Effetto climatizzazione** = carico aggiuntivo correlato alle temperature, dominante da metà giugno a luglio
- **Canone TV** = sempre escluso dai confronti energetici

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v13

| Tabella | Righe | Copertura |
|---|---:|---|
| `bollette` | **10** (era 9) | Mar 2025 → **Luglio 2026** |
| `produzione_fv` | 6 | Mar 2026 → Ago 2026 (parziale) |
| `produzione_giornaliera` | **147** | 23/03 → 27/08/2026 |
| `letture_inverter_orarie` | **94.662** | 09/05 → 25/08/2026 |
| `letture_quadro` | **5.930** | 23/12/2025 → **27/08/2026 15:00** |
| `verifica_bias_tapo_inverter` | 88 | 09/05 → 04/08/2026 |
| `riconciliazione_mensile` | **9** | Giugno (definitivo) + **Luglio (definitivo, v13)** + Agosto (preliminare) |
| `cronologia_eventi` | **16** | + vacanza 17-22/08 + sostituzione pannelli (annunciata, non eseguita) |
| `meteo_giornaliero` | 65 | 01/06 → 04/08/2026 (fermo, nessun aggiornamento in questa sessione) |
| `meteo_orario` | 1.560 | 01/06 → 04/08/2026 |
| `fornitori_storico` | **4** | + Sorgenia Luglio 2026 |
| `pannelli_fv_specifiche` | 2 | Dahai (attuale) + Megasol (annunciato, non installato) |

---

## 4. NOVITÀ v13 — Chiusura definitiva luglio 2026

### Bolletta Sorgenia luglio (fattura V012606612873, emessa 17/08/2026)

| Voce | Valore |
|---|---:|
| Consumo fatturato | 621,1 kWh (F1=186,8 / F2=173,4 / F3=260,9) |
| Totale bolletta (no canone) | 214,57 € |
| Canone TV | 9,00 € |
| **Totale da pagare** | **223,57 €** |
| Prezzo medio quota consumi | 0,265706 €/kWh |

**Confronto col dato Sonoff preliminare (597,45 kWh):** bolletta ufficiale +23,65 kWh (+3,96%) — stesso pattern già visto a giugno (bolletta sempre leggermente sopra il dato Sonoff/eWeLink).

### Riconciliazione definitiva (sostituisce la stima preliminare)

| | Tapo | Inverter diretto |
|---|---:|---:|
| Produzione FV | 127,78 kWh | 137,92 kWh |
| Immissione | 3,31 kWh | 3,31 kWh |
| Autoconsumo FV | 124,47 kWh | 134,61 kWh |
| **Consumo reale casa** | **745,57 kWh** | **755,71 kWh** |
| Δ vs luglio 2025 (stima 468 kWh) | +59,3% | +61,5% |
| Δ prelievo rete vs giugno 2026 (bolletta) | +80,3% | +80,3% |
| Δ F1 vs giugno 2026 | **+174,7%** | +174,7% |

**Lettura:** l'immissione (3,31 kWh sull'intero mese) conferma che il FV ha autoconsumato praticamente tutta la propria produzione — non ha smesso di funzionare, è stato sommerso dal carico piscina+climatizzazione. Il balzo di F1 (+175%, la fascia diurna che il FV dovrebbe comprimere) è il segnale più diretto che il carico ha superato la capacità dell'impianto (800 Wp, senza accumulo).

### Effetto tariffa vs effetto volume (decomposizione)

Il prezzo medio dell'energia è salito da 0,218578 €/kWh (giugno) a 0,265706 €/kWh (luglio, +21,6%), causato principalmente da:
- PUN in salita su tutte le fasce (F1 +22,6%, F2 +11,7%, F3 +19,7%)
- Dispacciamento quasi raddoppiato (0,0199 → 0,0385 €/kWh)

Quote fisse Sorgenia invariate (17,07 vs 17,08 €/mese) — non è un aumento deciso dal fornitore, ma il contratto **indicizzato PUN** che trasferisce l'oscillazione di mercato (legata anche alla stessa ondata di caldo che ha fatto salire i consumi).

Decomposizione approssimata dell'aumento di costo energia (giugno→luglio, Δ=+105,60€): ~57% effetto volume, ~15% effetto tariffa puro, ~12% interazione, ~15% residuo (IVA/accise su base più alta).

**Causa dell'aumento, confermata (da sessione precedente):** uso intensivo di climatizzazione (split 12.000 BTU salone + multisplit 3 camere) durante un luglio con temperatura media 28,4°C (dato Settebagni), correlazione r=0,642 con il prelievo.

---

## 5. NOVITÀ v13 — Agosto 2026, stato attuale (mese ancora aperto)

### Produzione FV

| Fonte | Copertura | kWh |
|---|---|---:|
| Tapo (foglio Anno/Mese) | 01-27/08 (27/31 gg, 27 parziale) | 101,25 |
| Inverter diretto | 01-25/08 (25/31 gg) | 102,61 |

Bias inverter vs Tapo sul periodo comune (01-25/08): **+7,8%**, coerente col bias sistematico +8,2% già documentato da maggio a luglio.

### Prelievo/immissione (eWeLink, completo 01-27/08 15:00)

Import da nuovo export storico (19/02→27/08/2026, 4.536 righe), che ha colmato il buco 05/08-27/08 rimasto dalla sessione precedente. Nessuna correzione di fuso necessaria (timestamp già coerenti col DB esistente).

### Riconciliazione preliminare (01-26/08, 26 giorni completi)

| | Valore |
|---|---:|
| Prelievo rete | 450,39 kWh |
| Produzione FV (Tapo) | 98,80 kWh |
| Immissione | 7,06 kWh |
| Autoconsumo FV | 91,74 kWh |
| **Consumo reale casa** | **542,13 kWh** |
| Media prelievo/giorno | 17,32 kWh/g |

Confronto medie giornaliere di prelievo: giugno 11,48 kWh/g → **luglio 20,04 kWh/g** → **agosto (01-26) 17,32 kWh/g** — segnale di attenuazione post-picco di luglio, ma dato ancora preliminare (mese aperto, bolletta non emessa).

### Periodo vacanza 17-22/08 (confermato dall'utente)

| Periodo | Prelievo medio orario |
|---|---:|
| 01-16/08 (casa occupata) | 0,893 kWh/h |
| **17/08 00:00 - 22/08 14:00 (vacanza)** | **0,175 kWh/h (-80,5%)** |

Rientro visibile ora per ora nel pomeriggio del 22/08 (prelievo torna a >1 kWh/h dalle 15:00). Immissione in aumento nello stesso periodo — la produzione FV era normale, semplicemente non autoconsumata in assenza di occupanti. Utile come periodo di controllo per isolare il "consumo base minimo" della casa (frigorifero, standby) senza AC né piscina attiva.

Registrato in `cronologia_eventi`, note aggiornate su `riconciliazione_mensile` e sui giorni interessati in `produzione_giornaliera`.

---

## 6. NOVITÀ v13 — Sostituzione pannelli bifacciali: chiarito lo stato

L'annuncio del 04/08 (sostituzione Dahai 400W → Megasol bifacciali 440W, inverter EVT800 invariato) **non è ancora stato eseguito** — confermato dall'utente il 27/08 ("non ancora, lavoro rimandato/in corso").

Verifica indipendente sui dati: nessun salto nei picchi di potenza giornalieri per pannello nell'intero periodo 04-25/08 (sempre 310-400W, range identico ai vecchi Dahai; un pannello da 440W bifacciale mostrerebbe picchi più alti). Header "Capacity: 0.80 KWp" invariato su tutti i report EnverView del periodo. Impianto quindi ancora **Dahai 400W/800Wp** per tutti i dati analizzati finora.

Nota tecnica già in `pannelli_fv_specifiche` da sessione precedente: il guadagno bifacciale, anche minimo, porterebbe la corrente Impp oltre il limite di 14A continui dell'EVT800 (margine attuale solo 3,2%) — punto di attenzione per quando l'installazione avverrà davvero, da monitorare sui dati orari post-installazione.

---

## 7. Dati presenti nel DB — bollette (aggiornato v13)

| Periodo | Fornitore | F1 | F2 | F3 | Tot kWh | Luce € | €/kWh | Stato |
|---|---|---:|---:|---:|---:|---:|---:|---|
| Mar-Apr 2025 | Plenitude | 217 | 269 | 314 | 800 | 249,84 | — | confermato |
| Mag-Giu 2025 | Plenitude | 227 | 243 | 331 | 801 | 236,93 | 0,200037 | confermato |
| Lug-Ago 2025 | Plenitude | 280 | 253 | 361 | 894 | 277,93 | 0,207383 | confermato |
| Set-Ott 2025 | Plenitude | 244 | 269 | 303 | 816 | 255,34 | 0,204216 | confermato |
| Nov-Dic 2025 | Plenitude | 276 | 276 | 400 | 952 | 286,09 | 0,201155 | confermato |
| Gen-Feb 2026 (baseline) | Plenitude | 297 | 306 | 347 | 950 | 290,35 | 0,206895 | confermato |
| Mar-Apr 2026 | Plenitude | 217 | 257 | 339 | 813 | 253,69 | 0,204600 | confermato |
| Maggio 2026 | Sorgenia | 41,7 | 102,6 | 144,3 | 288,6 | 88,69 | 0,201421 | confermato |
| Giugno 2026 | Sorgenia | 68,0 | 102,9 | 173,6 | 344,5 | 108,97 | 0,218578 | confermato |
| **Luglio 2026** | **Sorgenia** | **186,8** | **173,4** | **260,9** | **621,1** | **214,57** | **0,265706** | **confermato (v13)** |

Dettaglio storico mensile invariato dalle versioni precedenti (vedi v12 §"Dati presenti nel DB" per il dettaglio Mar-Ott 2025, non ripetuto qui).

---

## 8. Confronti principali consolidati

### Riepilogo trimestre estivo 2026 (bollette ufficiali)

| Mese | Prelievo | Δ vs mese prec. | Δ F1 vs mese prec. |
|---|---:|---:|---:|
| Giugno | 344,5 kWh | — | — |
| Luglio | 621,1 kWh | **+80,3%** | **+174,7%** |
| Agosto (preliminare, 01-26) | ~450 kWh (proiezione mese intero più alta, dato parziale) | in attenuazione | da confermare a bolletta |

### Maggio 2025 vs maggio 2026 (invariato)

```
F1: -56,1%  |  F3: +2,3%  |  Totale: -20,1%
```

### Cambio fornitore (invariato)

```
Beneficio cambio fornitore     ≈ 7-8 €/mese (quota fissa più bassa)
Beneficio FV/minor prelievo    ≈ 45-50 €/mese (baseline gen-feb, causa principale)
```

---

## 9. Formule di riconciliazione (usare sempre)

```
Autoconsumo FV     = Produzione FV - Immissione
Consumo reale casa = Prelievo rete + Produzione FV - Immissione
```

Specificare sempre la fonte di Produzione FV usata (Tapo vs inverter diretto), bias sistematico +8,2%/+7,8-8,3% confermato su tutti i mesi da maggio ad agosto.

Verifica incrociata Tapo foglio Anno vs somma foglio Mese: match esatto anche in v13 (giugno, luglio confermati al millesimo contro i valori già in DB).

---

## 10. Pendenze aperte (aggiornate v13)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Produzione FV inverter diretto 26-27/08 | Export EnverView | Media | Solo 2 giorni mancanti per allineare le due fonti fino a fine agosto (parziale) |
| Prelievo/immissione oltre 27/08 15:00 | eWeLink | Media | Per chiudere agosto serve fino al 31/08 |
| Dati meteo Settebagni oltre 04/08 | Open-Meteo (via browser utente, robots.txt blocca accesso automatizzato) | Media | Utile per correlazione temperatura/prelievo su luglio-agosto completo e per confermare l'attenuazione post-picco |
| Bolletta Sorgenia agosto 2026 | Sorgenia (attesa metà settembre, pattern ~16-18gg dopo fine mese) | Alta (quando disponibile) | Chiusura definitiva di agosto, incluso l'effetto vacanza sul conto economico |
| Analisi economica sostituzione pannelli/inverter | — | Bassa (lavoro non ancora eseguito) | Riprendere quando l'installazione dei Megasol sarà effettivamente in corso/completata |
| Rigenerare token GitHub | — | Consigliata (non urgente, rischio valutato basso dall'utente) | Riutilizzato in molte sessioni consecutive |

---

## 11. Regole operative del progetto (invariate + None nuove in questa sessione)

Tutte le regole precedenti (1-21, vedi v12) restano valide e sono state applicate senza eccezioni in questa sessione (privacy, esclusione canone TV, distinzione fonte Tapo/inverter, versioning delle riconciliazioni, tracciamento provenienza dati).

---

## 12. Conclusione attuale del progetto

Il quadro di luglio è ora **chiuso e definitivo**: il salto di consumo (+80,3% di prelievo vs giugno, +59-61% di consumo reale vs luglio 2025) ha causa doppiamente confermata — climatizzazione intensiva durante un'ondata di caldo prolungata, aggravata da un effetto tariffa reale (PUN e Dispacciamento in forte salita, non deciso dal fornitore ma trasferito dal contratto indicizzato). Il fotovoltaico non ha smesso di funzionare: ha continuato ad autoconsumare quasi tutta la propria produzione (immissione residua 3,31 kWh sull'intero mese), semplicemente il carico estivo (piscina + due impianti di climatizzazione) ha superato ampiamente la capacità di un impianto da 800 Wp senza accumulo.

Agosto mostra i primi segnali di attenuazione (media prelievo giornaliero in calo rispetto a luglio), ma il dato è ancora preliminare e parzialmente "sporcato" in senso positivo da 6 giorni di vacanza (17-22/08) in cui la casa è rimasta vuota — questo va tenuto presente per non sovrastimare il miglioramento strutturale quando arriverà la bolletta definitiva.

La sostituzione dei pannelli con moduli bifacciali Megasol, annunciata a inizio agosto, non è ancora avvenuta: quando succederà, sarà il momento di riprendere l'analisi economica di payback e monitorare da vicino il vincolo di corrente stretto sull'inverter EVT800 già identificato.

---
*Fine handoff v13. Prossima azione consigliata: (1) completare produzione FV inverter 26-27/08; (2) completare eWeLink e meteo fino a fine agosto quando disponibili; (3) ingest bolletta Sorgenia agosto appena emessa (attesa metà settembre); (4) riprendere l'analisi pannelli bifacciali quando l'installazione sarà effettivamente in corso.*
