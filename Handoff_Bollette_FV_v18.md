# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 18 — aggiornata al 29/09/2026

**Data:** 2026-09-29
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v18:**
1. **Agosto 2026 chiuso anche economicamente**: bolletta Sorgenia agosto ingerita (567,7 kWh, 210,00€ totale). Era l'ultima pendenza ad alta priorità segnalata in v16/v17.
2. **`letture_quadro` esteso**: da 14/09 13:00 a 29/09 21:00 (+368 letture orarie), via nuovo export storico eWeLink completo (24/03–29/09/2026).
3. **`produzione_giornaliera` completata 14-29/09**: corretto il valore parziale del 14/09 (2,06→3,41 kWh) e aggiunti 15-29/09 da nuovo export Tapo.
4. **Sostituzione pannelli Dahai→Megasol eseguita** (annunciata in v13, 04/08): fermo impianto 22-25/09, reinstallazione 26/09 ore 13:00. Primo segnale preliminare: **+56,4%** di produzione media giornaliera nei 3 giorni puliti post-sostituzione vs i 7 giorni pre-sostituzione — molto oltre il solo +10% di potenza nominale, da verificare con più dati.
5. Nuova riga `riconciliazione_mensile`: agosto, chiusura con **prelievo da bolletta ufficiale** (567,7 kWh) invece del dato eWeLink preliminare.
6. Nuova riga `fornitori_storico` e `cronologia_eventi` (sostituzione pannelli).

---

## 1. Obiettivo del progetto

Invariato: ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo.

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v18

| Tabella | Righe | Note v18 |
|---|---:|---|
| `bollette` | **11** (era 10) | + Agosto 2026 Sorgenia (confermato) |
| `letture_quadro` | **6.728** (era 6.360) | + 368 letture, copertura ora 23/12/2025 → **29/09/2026 21:00** |
| `produzione_giornaliera` | 165 | 14/09 corretto, **15-29/09 aggiunti** |
| `riconciliazione_mensile` | **14** (era 13) | + agosto, fonte "Tapo (bolletta ufficiale)", definitivo |
| `cronologia_eventi` | **17** (era 16) | + sostituzione pannelli eseguita |
| `fornitori_storico` | **5** (era 4) | + Sorgenia agosto 2026 |
| Le altre tabelle | — | invariate da v17 |

---

## 4. NOVITÀ v18 — Agosto 2026 chiuso anche economicamente

### Bolletta Sorgenia agosto (fattura V012607655422, emessa 22/09/2026)

| Voce | Valore |
|---|---:|
| Periodo | Agosto 2026 |
| Consumo fatturato | 567,7 kWh (F1=146,8 / F2=142,8 / F3=278,1) |
| Prezzo medio quota consumi | 0,271094 €/kWh |
| Totale bolletta (energia) | 201,00 € |
| Canone TV | 9,00 € |
| **Totale da pagare** | **210,00 €** |
| Picco potenza prelevata | 4,0 kW su 4,5 impegnati |

Verifica incrociata con eWeLink (566,10 kWh, mese completo 31/31 gg): delta -1,6 kWh (-0,28%) — scarto minimo, molto più piccolo di giugno (-1,33%) e luglio (-3,96%).

### Chiusura energetico-economica definitiva

```
Prelievo rete (bolletta, confermato)  = 567,7 kWh
Produzione FV (Tapo, confermato)      = 115,823 kWh
Immissione (eWeLink, confermato)      = 7,39 kWh

Autoconsumo FV     = 115,823 - 7,39  = 108,43 kWh
Consumo reale casa = 567,7 + 108,43  = 676,13 kWh
```

Confrontabile con le due righe già presenti (fonte eWeLink invece di bolletta): Tapo/eWeLink 674,53 kWh (v17), inverter diretto/eWeLink 683,40 kWh (v15-16). Le tre stime differiscono per pochi punti percentuali, spiegati dal bias sistematico Tapo/inverter e dal piccolo scarto bolletta/eWeLink.

### Effetto prezzo — segnale rilevante

```
Prezzo agosto (0,271094 €/kWh) vs prezzo maggio (0,201421 €/kWh) = +34,6%
```

Non è un effetto cambio-fornitore (stesso contratto Sorgenia, prezzo indicizzato PUN): PUN F1/F2/F3 di agosto (0,1745/0,2044/0,1717 €/kWh) più alto di maggio, più dispacciamento. Stimando a parità di consumo di agosto col prezzo di maggio: 567,7 × 0,201421 = 114,33 € contro i 153,90 € realmente fatturati per la quota consumi → **≈39,6 € del costo di agosto sono dovuti al solo aumento di prezzo, non ai kWh**.

---

## 5. NOVITÀ v18 — Estensione dati Sonoff/eWeLink e Tapo fino al 29/09

Nuovo export storico eWeLink (24/03–29/09/2026 UTC, 4.536 righe) ha colmato il buco tra 14/09 13:00 e 29/09 21:00. Verifica di completezza sui mesi già chiusi (giugno, luglio, agosto): tutti confermati, nessuna discrepanza rilevante rispetto ai valori già in DB (giugno: 340,03 vs 339,92 kWh già confermato, +0,03% — rumore di arrotondamento).

Nuovo export Tapo (foglio "Mese", copertura 01/07–29/09) ha permesso di:
- correggere il valore parziale del 14/09 (2,0576 → **3,411617 kWh**, il vecchio dato era un export a metà giornata);
- aggiungere 15 nuovi giorni (15-29/09) a `produzione_giornaliera`.

---

## 6. NOVITÀ v18 — Sostituzione pannelli eseguita: primo segnale (+56%, preliminare)

Fermo impianto confermato nella serie Tapo: produzione 0,00 kWh/gg dal 22 al 25/09 incluso. Reinstallazione il 26/09 ore 13:00 (2,45 kWh, giorno parziale).

| Periodo | Giorni puliti | Pannelli | Produzione media/gg |
|---|---:|---|---:|
| Pre-sostituzione (15-21/09) | 7 | Dahai 2×400Wp | 3,28 kWh/gg |
| **Post-sostituzione (27-29/09)** | 3 | **Megasol 2×440Wp bifacciali** | **5,13 kWh/gg** |

```
Incremento = +56,4%
```

Molto superiore al solo guadagno di potenza nominale (+10%, 800→880Wp). Compatibile con un effetto bifacciale rilevante, ma **dato preliminare** (solo 3 giorni puliti) — non isola ancora l'effetto pannello da un'eventuale differenza di irradianza tra le due finestre di 7 e 3 giorni. Da confermare con più settimane di dati. Il vincolo noto sull'inverter EVT800 (margine di corrente Impp stretto con guadagno bifacciale, già documentato in `pannelli_fv_specifiche`) resta da monitorare sui dati orari, non ancora disponibili per il periodo post-sostituzione.

---

## 7. Dati invariati da v17

Bollette maggio-luglio, scheda EVT800, cronologia tecnica precedente, casi anomali 09/05 e 10/09 — tutto invariato. Vedi v17 per il dettaglio.

---

## 8. Pendenze aperte (aggiornate v18)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **giugno 2026** — conto economico | Sorgenia | Alta | L'unico mese pieno con piscina di cui manca ancora il lato € (solo kWh disponibili) |
| Bolletta Plenitude **marzo-aprile 2026** (PDF originale) | Plenitude | Media | Ancora `stima` per lo split F2/F3 mensile |
| Più giorni di produzione post-sostituzione pannelli | Tapo/inverter | **Alta** | Solo 3 giorni puliti per il +56% osservato — servono almeno 2-3 settimane |
| Dati orari inverter post-sostituzione (per il monitoraggio del limite di corrente EVT800) | EnverView | Media | Nessun dato orario per-pannello disponibile dopo il 26/09 |
| Dati meteo oltre il 04/08 | Open-Meteo | Media | Invariato da v16-v17, utile anche per isolare l'effetto irradianza sul +56% pannelli |
| Analisi economica sostituzione pannelli/inverter | — | Bassa | invariato |
| Bolletta Sorgenia **settembre 2026** | Sorgenia (attesa metà ottobre) | Bassa | Mese ancora in corso |
| Token GitHub | — | ✅ **Rigenerato in questa sessione** | — |

**Azione a più alto impatto per la prossima sessione:** ingest bolletta Sorgenia giugno (unico mese piscina ancora senza conto economico) e/o raccogliere altri giorni di produzione FV post-sostituzione pannelli per consolidare il +56%.

---

## 9. Regole operative del progetto (invariate)

Tutte le regole precedenti restano valide e sono state applicate senza eccezioni.

---

## 10. Conclusione attuale del progetto

Con questa sessione **luglio e agosto 2026 sono ora chiusi sia dal lato energetico sia da quello economico**. Il consumo reale della casa si conferma nell'ordine di 745,6 kWh (luglio, picco climatizzazione) e 676,1 kWh (agosto, -9,3% vs luglio nonostante il caldo residuo, grazie in parte alla vacanza 17-22/08). Entrambi restano ben sopra la baseline pre-FV (gen-feb 2026, ~475 kWh/mese: +56,9% luglio, +42,3% agosto), a conferma che il fotovoltaico attutisce ma non annulla l'impatto di climatizzazione e piscina in piena estate.

L'aumento di spesa di agosto rispetto a maggio (+34,6% sul prezzo medio) è quasi interamente un effetto tariffario stagionale (indice PUN), non un effetto fornitore.

La novità più interessante emersa in questa sessione è il primo segnale, ancora preliminare, dell'effetto della sostituzione dei pannelli Dahai con i Megasol bifacciali: +56,4% di produzione media giornaliera nei primi 3 giorni, ben oltre il +10% di potenza nominale atteso. Da trattare con cautela finché non si accumulano più settimane di dati puliti.

---
*Fine handoff v18. Prossima azione consigliata: (1) ingest bolletta Sorgenia giugno per completare il conto economico dell'unico mese piscina ancora aperto; (2) raccogliere più giorni di produzione FV post-sostituzione pannelli per consolidare la stima del +56%; (3) recuperare dati meteo settembre per isolare l'effetto irradianza da quello pannello; (4) quando disponibili, dati orari inverter post-26/09 per monitorare il margine di corrente EVT800 con i nuovi pannelli bifacciali.*
