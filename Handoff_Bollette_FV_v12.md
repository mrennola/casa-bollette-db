# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 12 — aggiornata al 04/08/2026

**Data:** 2026-08-04
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v12 (continuazione sessione v10/v11, stesso giorno):**
1. **Nuova tabella `meteo_giornaliero`**: 17 righe con temperature giornaliere di Roma
   (01-17/07/2026), fonte web (weatherandclimate.eu, stazione Roma centro — non esiste
   stazione dedicata a Settebagni, usata come proxy la più vicina disponibile).
2. **Correlazione quantificata tra temperatura e prelievo elettrico**: r=0,71 tra
   temperatura media giornaliera e prelievo (luglio 1-17); ogni grado in più ≈ +4 kWh/giorno
   di prelievo stimato.
3. **Scoperta contestuale**: un'ondata di calore ha colpito Roma anche a **giugno**
   (17-29/06, +5,3°C sopra la norma, picco 40,1°C il 29/06) — non solo a luglio. Il prelievo
   medio giornaliero è salito del **+75,7%** proprio in quella finestra, spiegando parte
   dell'aumento di F1/F3 già osservato a giugno (bolletta ufficiale).
4. Nuovo evento in `cronologia_eventi` per l'ondata di calore di giugno.

---

## 1. Obiettivo del progetto

Ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici,
confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e,
da questa versione, **dati meteo**.

**Domanda guida:**
```
Quanto del risparmio deriva dal fotovoltaico/autoconsumo
e quanto dal cambio fornitore?
```

Distinzione metodologica sempre obbligatoria:
- **Effetto kWh** = riduzione del prelievo dalla rete (fotovoltaico/autoconsumo)
- **Effetto tariffa** = risparmio dal cambio fornitore
- **Effetto piscina** = da giugno 2026, nuovo carico diurno
- **Effetto climatizzazione (nuovo, v12)** = carico aggiuntivo correlato alle temperature
- **Canone TV** = sempre escluso dai confronti energetici

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati. I dati meteo introdotti in v12 sono dati pubblici
aggregati per la città di Roma (nessuna stazione dedicata a Settebagni disponibile), non
identificano la persona né l'indirizzo esatto.

Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v12

| Tabella | Righe | Copertura |
|---|---:|---|
| `bollette` | 9 | Mar 2025 → Giugno 2026 |
| `produzione_fv` | 6 | Mar 2026 → Ago 2026 |
| `produzione_giornaliera` | 124 | 23/03 → 04/08/2026 |
| `letture_inverter_orarie` | 76.928 | 09/05 → 04/08/2026 |
| `letture_quadro` | 5.374 | 23/12/2025 → 04/08/2026 11:00 |
| `verifica_bias_tapo_inverter` | 88 | 09/05 → 04/08/2026 |
| `riconciliazione_mensile` | 8 | Giugno (definitivo) + Luglio (preliminare) |
| `cronologia_eventi` | **12** (era 10) | + climatizzazione luglio + ondata calore giugno |
| **`meteo_giornaliero`** (nuova, v12) | **17** | **01-17/07/2026, temperature Roma** |
| `pannelli_fv_specifiche` | 1 | — |

Schema `meteo_giornaliero`: `data` (PK), `temp_min_c`, `temp_media_c`, `temp_max_c`,
`deviazione_norma_c` (scostamento dalla media climatica del periodo), `precipitazione_mm`,
`fonte`, `note`.

**Nota sulla fonte meteo:** non esiste una stazione meteo dedicata a Settebagni (quartiere
citato dall'utente); i dati usano la stazione "Roma" generica di weatherandclimate.eu
(lat 41.7875, lon 12.2372, prossima al centro città) come proxy più vicino disponibile via
ricerca web. Per un'analisi più precisa in futuro si potrebbe usare la stazione Roma Urbe
(più a nord, più vicina all'area di Settebagni) se se ne trova un archivio accessibile.

---

## 4. NOVITÀ v12 — Correlazione temperatura/prelievo elettrico

### Dati meteo Roma, luglio 2026 (giorni 1-17, fonte: weatherandclimate.eu)

| Giorno | T min | T media | T max | Scostamento norma | Prelievo (kWh) |
|---:|---:|---:|---:|---:|---:|
| 1 | 21,1 | 27,3 | 32,3 | +3,8 | 18,07 |
| 2 | 19,3 | 26,0 | 31,7 | +2,4 | 12,80 |
| 3 | 21,3 | 26,3 | 32,4 | +2,7 | 12,89 |
| 4 | 21,9 | 27,4 | 33,0 | +3,7 | 14,00 |
| 5 | 21,3 | 26,6 | 30,9 | +2,8 | 11,82 |
| 6 | 19,1 | 26,1 | 32,3 | +2,2 | 8,65 |
| 7 | 20,4 | 25,7 | 30,7 | +1,7 | 12,46 |
| 8 | 19,9 | 25,8 | 30,3 | +1,8 | 13,72 |
| 9 | 20,0 | 25,9 | 31,3 | +1,8 | 19,95 |
| 10 | 21,8 | 27,6 | 33,8 | +3,4 | 16,81 |
| 11 | 21,2 | 27,2 | 32,1 | +3,0 | 20,38 |
| 12 | 20,6 | 27,0 | 32,9 | +2,7 | 20,35 |
| 13 | 21,2 | 27,5 | 32,2 | +3,1 | 25,75 |
| 14 | 22,1 | 28,6 | 34,6 | +4,2 | 21,37 |
| 15 | 20,8 | 27,7 | 32,6 | +3,2 | 20,45 |
| 16 | 22,3 | 28,1 | 32,9 | +3,5 | 30,94 |
| 17 | 25,1 | 29,3 | 34,0 | +4,7 | 24,82 |

Norma mensile luglio (media storica): 24,5°C. Tutto il periodo mostra scostamento positivo
costante (+1,7 a +4,7°C), coerente con un luglio anomalo nel complesso, con un'intensificazione
progressiva verso metà mese (14-17/07 i giorni più caldi disponibili, coincidenti con i
prelievi più alti: 20-31 kWh/giorno).

### Correlazione statistica (giorni 1-17 luglio)

| Variabile confrontata col prelievo | Coefficiente di correlazione (r) |
|---|---:|
| Temperatura media giornaliera | **0,71** (forte) |
| Temperatura massima giornaliera | 0,46 (moderata) |
| Temperatura minima (notturna) | 0,57 (moderata-forte) |

```
Regressione lineare: Prelievo stimato (kWh) = 4,02 * T_media - 90,74
Ogni +1°C di temperatura media -> +4,02 kWh/giorno di prelievo stimato
```

**Lettura:** la temperatura media giornaliera (che cattura sia il caldo diurno sia le notti
calde/tropicali) è il predittore più forte — coerente con un uso di climatizzazione esteso
sia di giorno (salone) sia probabilmente in parte anche di notte (camere da letto, multisplit).
La correlazione con la sola T massima è più debole, segno che non è solo il picco pomeridiano
a pesare ma la persistenza del caldo nell'arco della giornata/notte.

**Cautela statistica:** r=0,71 su soli 17 punti è indicativo ma non definitivo; giorni come
l'1/07 (18,07 kWh nonostante temperatura non estrema) mostrano che altri fattori (occupazione
della casa, uso variabile della piscina, ecc.) contribuiscono comunque alla varianza residua.

---

## 5. NOVITÀ v12 — Ondata di calore anche a giugno (retrospettiva)

Ricerca web ha rivelato che **anche giugno 2026** ha avuto un'ondata di calore severa,
distinta e più intensa di quella di luglio:

```
Periodo: 17-29 giugno 2026
Temperatura massima media nel periodo: ~38°C
Anomalia: +5,3°C sopra la media recente
Picco assoluto: 40,1°C il 29/06/2026 (stazione AUBAC, Collegio Romano)
Contesto: il primo semestre 2026 è risultato il più caldo mai registrato
alla stazione di Roma Ciampino; 11 giornate torride a giugno (vs 6 nel 2003)
```

### Correlazione con i dati del progetto

| Periodo giugno | Media prelievo giornaliero |
|---|---:|
| Pre-ondata (1-16/06) | 8,50 kWh/giorno |
| Durante l'ondata (17-29/06) | **14,93 kWh/giorno** |
| **Incremento** | **+75,7%** |

**Lettura:** questo spiega, almeno in parte, perché F1 e F3 di giugno erano già elevate
rispetto a maggio (§9 handoff v10) ancora prima di sapere della causa "climatizzazione"
confermata dall'utente per luglio — lo stesso fattore era probabilmente già attivo, e più
intensamente, nella seconda metà di giugno. Il giorno di picco assoluto del prelievo
giornaliero di giugno (27/06 = 25,36 kWh) cade esattamente nella finestra dell'ondata di
calore, a pochi giorni dal record assoluto di temperatura (40,1°C il 29/06).

**Implicazione per la lettura del progetto:** l'effetto "climatizzazione" non è isolato a
luglio — è presente (in modo più contenuto) già da metà giugno, sovrapposto all'effetto
piscina. Le due cause (piscina + condizionatori) insieme spiegano gran parte della crescita
dei consumi estivi 2026 rispetto al pattern osservato in primavera.

---

## 6. Riepilogo cause identificate per l'aumento dei consumi estivi 2026

| Fattore | Periodo attivo | Evidenza | Stato |
|---|---|---|---|
| Fotovoltaico/autoconsumo | Da marzo 2026 | F1 -56% maggio, -48% giugno vs anno prec. | Confermato, effetto positivo (riduce prelievo) |
| Piscina (pompa) | Da 30/05/2026 | Immissione -80,9% ore pompa attiva (v6) | Confermato, effetto negativo (aumenta consumo reale) ma in parte assorbito dal FV |
| **Climatizzazione (ondata di calore giugno)** | **17-29/06/2026** | **+75,7% prelievo medio giornaliero nella finestra** | **Confermato (v12), causa meteo oggettiva** |
| **Climatizzazione (ondata di calore luglio, riportata dall'utente)** | **Tutto luglio (+1,7/+4,7°C su norma)** | **r=0,71 correlazione T media/prelievo** | **Confermato (v11/v12), causa comportamentale + meteo** |

---

## 7. Pendenze aperte (aggiornate v12)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **luglio 2026** | Sorgenia | **Alta** | Chiusura definitiva luglio |
| Dati meteo giorni 18-31/07 e agosto | Web (weatherandclimate.eu o simile) | Media | Non recuperati in questa sessione (dati non disponibili sulla pagina consultata oltre il 17/07) |
| Dati meteo dettagliati per giugno (giorno per giorno) | Web | Bassa | Trovato solo il quadro aggregato dell'ondata (17-29/06); utile per rifinire la correlazione se servisse |
| Push di questa sessione (v10+v11+v12) su GitHub | — | **Alta** | **In attesa del token** |
| Produzione FV agosto 2026 — completare | Tapo + inverter | Media | Solo 4/31 giorni |
| Analisi economica sostituzione pannelli/inverter | — | Media | In sospeso da v7 |
| Rigenerare token GitHub | — | Consigliata | Riutilizzato in molte sessioni |

---

## 8. Regole operative del progetto (invariate + 1 nuova)

Tutte le regole precedenti (1-20, vedi v11) restano valide. Nuova regola:

21. **(Nuova, v12)** Quando si introducono dati meteo esterni, tracciare sempre la fonte e
    la stazione di rilevamento nella tabella `meteo_giornaliero` (colonna `fonte`), dato che
    non esiste una stazione meteo dedicata a Settebagni — usare sempre il proxy più vicino
    disponibile e segnalarlo esplicitamente, senza spacciarlo per dato locale esatto.

---

## 9. Conclusione attuale del progetto

Il quadro del progetto si arricchisce di un tassello importante: **i picchi di consumo
estivi 2026 (giugno e luglio) hanno ora una spiegazione quantificata**, non solo qualitativa.

```
Giugno 2025 → 440 kWh (bolletta)
Giugno 2026 → 344,5 kWh (bolletta) → -21,7% (nonostante ondata di calore + piscina)

Luglio 2025 → 468 kWh (stima da nota bolletta)
Luglio 2026 → 597,45 kWh (Sonoff, preliminare) → +27,7% (con ondata di calore prolungata)
```

Il fatto che **giugno 2026 mantenga comunque una riduzione del -21,7%** rispetto a giugno
2025 **nonostante un'ondata di calore severa** (+5,3°C, picco 40,1°C) è un segnale forte a
favore dell'efficacia del fotovoltaico. A luglio, con un caldo persistente su tutto il mese
(non solo un picco di 13 giorni) e presumibilmente un uso ancora più esteso della
climatizzazione, il piccolo impianto FV (800 Wp, senza accumulo) non basta più a compensare,
e il prelievo supera anche l'anno precedente pre-fotovoltaico.

**Causa dell'aumento di luglio, ora doppiamente confermata:**
1. Riportata dall'utente: uso intensivo di due impianti di climatizzazione (split 12000 BTU
   salone + multisplit 3 camere)
2. Confermata dai dati meteo esterni: correlazione r=0,71 tra temperatura media e prelievo,
   in un luglio con temperature costantemente 2-5°C sopra la norma stagionale

**Frontiera aperta:** bolletta Sorgenia di luglio per la chiusura economica definitiva; push
dei tre commit locali (v10, v11, v12) su GitHub.

---
*Fine handoff v12. Prossima azione consigliata: (1) push su GitHub (serve token); (2) ingest
bolletta Sorgenia luglio; (3) opzionale — completare i dati meteo per il resto di luglio e
per agosto se utile ad affinare la correlazione; (4) completare produzione FV agosto; (5)
rigenerare token GitHub.*

---

## ADDENDUM (stesso giorno, dopo v12) — Cadenza di fatturazione Sorgenia confermata

L'utente ha confermato un dettaglio operativo importante: **Sorgenia fattura mensilmente**
(una bolletta per ogni mese solare di consumo), a differenza di Plenitude che fatturava a
bimestre. Le due bollette Sorgenia ricevute finora (maggio, giugno) sono ciascuna per un
singolo mese — non bimestrali.

**Pattern di emissione osservato:**

| Periodo fatturato | Data emissione | Ritardo |
|---|---|---|
| Maggio 2026 | 18/06/2026 | ~18 giorni dopo fine mese |
| Giugno 2026 | 16/07/2026 | ~16 giorni dopo fine mese |

**Aspettativa per la prossima bolletta:** copre il **periodo luglio 2026** (non un
bimestre luglio-agosto), con emissione stimata verso **metà agosto 2026**, seguendo lo
stesso pattern di ~16-18 giorni di ritardo rispetto alla fine del mese di competenza.

Questa regola va tenuta a mente per non aspettarsi erroneamente una fattura cumulativa o
per non sollecitare prematuramente una bolletta che, per pattern storico, non è ancora
matura per l'emissione.

**Nessun nuovo dato da inserire nel DB** in questo addendum — le due bollette citate erano
già presenti (`real_sorgenia_2026_mag`, `real_sorgenia_2026_giu`). Aggiunta solo una voce
in `cronologia_eventi` (categoria `fornitore`) per tracciare la conferma della cadenza.
