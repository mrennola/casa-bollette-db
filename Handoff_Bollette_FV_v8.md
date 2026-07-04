# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 8 — aggiornata al 04/07/2026

**Data:** 2026-07-04
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v8:**
1. **Bolletta Plenitude Mar-Apr 2026 confermata da PDF originale** (era `stima` dalla v3). Verificati tutti i totali; **segnalata e documentata** l'impossibilità di verificare lo split F2/F3 separato dal PDF, perché la bolletta Plenitude è bioraria (F1/F23 aggregate).
2. **Anomalia 09/05/2026 risolta.** Importate 294 righe orarie dal "Rapporto giornaliero" EnverView del 09/05. Causa identificata: non un errore di misura, ma una sottocopertura oraria del log inverter (14:12–19:08) coincidente col changeover EVT560→EVT800. Il giorno resta non comparabile tra le fonti, ma ora con motivazione definitiva invece che "da verificare".
3. **Bolletta Sorgenia maggio 2026 riverificata da PDF** — nessuna discrepanza, dato già corretto in DB dalla sessione precedente.
4. **Nuova informazione tecnica**: l'offerta Sorgenia (Next Energy Sunlight) è a **prezzo variabile indicizzato al PUN**, non fissa — il prezzo medio si muoverà mese su mese. Da tenere presente nei confronti futuri (giugno, luglio).

---

## 1. Obiettivo del progetto

Ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici,
confrontando bollette, produzione fotovoltaica, dati contatore e cambiamenti tecnici.

**Domanda guida:**
```
Quanto del risparmio deriva dal fotovoltaico/autoconsumo
e quanto dal cambio fornitore?
```

Distinzione metodologica sempre obbligatoria:
- **Effetto kWh** = riduzione del prelievo dalla rete (fotovoltaico/autoconsumo)
- **Effetto tariffa** = risparmio dal cambio fornitore
- **Effetto piscina** = da giugno 2026, nuovo carico diurno che sposta produzione da immissione ad autoconsumo
- **Canone TV** = sempre escluso dai confronti energetici

Domanda di controllo su ogni risparmio osservato:
```
È risparmio da prezzo più basso o da meno kWh prelevati?
```

---

## 2. Nota privacy (regola di progetto, invariata e vincolante)

I dati identificativi — intestatario, indirizzo di fornitura, POD, codice fiscale — **non
vengono mai memorizzati né inclusi in repo, handoff, DB o output di alcun tipo**, anche
quando forniti esplicitamente dall'utente nelle istruzioni di progetto o nei PDF caricati
(bollette). Questa regola è stata rispettata in modo continuo anche in questa sessione,
nonostante le bollette PDF caricate (Plenitude e Sorgenia) contenessero nome, indirizzo
completo, codice fiscale e POD in chiaro.

Dati tecnici dell'utenza (non identificativi, riportabili):

| Campo | Dato |
|---|---|
| Distributore | Areti (Lazio) |
| Tipologia cliente | Domestico residente |
| Potenza impegnata | 4,5 kW |
| Potenza disponibile | 5,0 kW |
| Tensione | 220 V |
| Misuratore | 2G |

---

## 3. Database e infrastruttura dati

### Repository GitHub
- **URL pubblico:** `https://github.com/mrennola/casa-bollette-db`
- **Visibilità:** pubblico (nessun dato identificativo)
- **File principale:** `reqa_bollette.db` (SQLite)
- **Ultimo commit:** `beb8314` (04/07/2026) — import righe orarie 09/05 + fix anomalia

### Come accedere in sessione futura
```bash
git clone --depth 1 https://github.com/mrennola/casa-bollette-db.git
cd casa-bollette-db
# poi python3 + sqlite3 per le query (row_factory = sqlite3.Row)
```
Nessun token necessario per la **lettura**. Per **scrivere** aggiornamenti serve il token
`claude-casa-bollette` (fine-grained, Contents Read/Write, scadenza 28/09/2026), fornito in
chat come file `token.txt` (mai incollato in chiaro). **Nota di sicurezza (invariata da v6/v7):**
il token è stato riutilizzato più volte in sessioni successive — consigliata rigenerazione
su GitHub appena possibile.

### Struttura del database (invariata nel numero di tabelle da v7, dati aggiornati)

| Tabella | Righe | Contenuto | Copertura |
|---|---:|---|---|
| `bollette` | 8 | Plenitude/Sorgenia, F1/F2/F3, importi | Mar 2025 → Mag 2026 |
| `produzione_fv` | 5 | Produzione FV mensile (Tapo + inverter) | Mar 2026 → Lug 2026 (parziale) |
| `produzione_giornaliera` | 92 | Serie giornaliera FV (Tapo + inverter) | 23/03 → 03/07/2026 |
| `letture_inverter_orarie` | **49.378** (era 49.084) | Letture orarie per singolo microinverter | **09/05 (parziale, 14:12) → 03/07/2026, continua** |
| `letture_quadro` | 4.629 | Sonoff/eWeLink: prelievo + immissione | 23/12/2025 → 04/07/2026 10:00 |
| `cronologia_eventi` | 10 | Date eventi tecnici | 05/2025 → 06/2026 |
| `fornitori_storico` | 3 | Prezzi medi comparati | Mag25 / Mar26 / Mag26 |
| `verifica_bias_tapo_inverter` | 53 | Confronto giornaliero Tapo vs inverter | 09/05 → 30/06/2026 |
| `riconciliazione_mensile` | 4 | Storico versioni calcolo chiusura piscina giugno | v4-v6 (superato) + v7 (corretto) |
| `pannelli_fv_specifiche` | 1 | Scheda tecnica pannello Dahai | — |

Schema `letture_inverter_orarie`: `sn`, `ts`, `vdc`, `vac`, `potenza_w`, `freq_hz`,
`temperatura_c`, `energia_tot_kwh` (contatore cumulato lifetime), `fonte_file`.
Deduplicazione su `(sn, ts)` — sicuro ricaricare file già presenti.

Aggregazione mensile: `substr(ts,1,7)`; giornaliera: `substr(ts,1,10)`.

---

## 4. Cronologia tecnica rilevante (invariata)

| Data | Evento | Impatto atteso |
|---|---|---|
| Mag-Giu 2025 | Periodo pre-FV anno precedente | Benchmark storico |
| Gen-Feb 2026 | Periodo benchmark pre-FV | Riferimento pulito senza FV |
| 23/24 mar 2026 | Installazione iniziale FV plug & play | Marzo mese misto |
| 04 apr 2026 | Riposizionamento/ottimizzazione pannelli | Da qui il calo F1 è più leggibile |
| 30 apr 2026 | Fine contratto/fattura Plenitude | Chiusura vecchio fornitore |
| 01 mag 2026 | Decorrenza Sorgenia | Inizio nuovo fornitore |
| **9 mag 2026 (14:12 circa)** | **Inizio changeover inverter EVT560→EVT800** | **Causa identificata dell'anomalia 09/05 (§7)** |
| 9/17 mag 2026 | Completamento cambio inverter → Envertech EVT800 | Maggiore potenza massima FV |
| 18 mag 2026 | Sonoff legge bene supply/immissione | Da qui immissione attendibile |
| 30 mag 2026 | Installazione piscina | Nuovo carico elettrico diurno |
| Da giu 2026 | Pompa piscina attiva tutto il mese | Aumenta consumo reale e autoconsumo FV |
| 04 lug 2026 | Sessione v8: conferma bollette, risoluzione anomalia | — |

---

## 5. Fotovoltaico — configurazione (invariata da v7)

| Componente | Dato |
|---|---|
| Nome sistema (EnverView) | Horti FV |
| Tipologia | Plug & play |
| Moduli | 2 × 400 W = 800 Wp (S/N microinverter: 30578336, 30578337) |
| Marca/modello pannelli | Dahai Solar DHM54T35-400/MR |
| Inverter iniziale | Envertech EVT560 (~0,56 kW) |
| Inverter attuale | Envertech EVT800 (dal ~13/05/2026), potenza max continua 800 W |
| Accumulo | No |
| Misura produzione — fonte 1 | Tapo / EnverView (foglio "Anno"/"Mese") |
| Misura produzione — fonte 2 | Export diretto inverter (mensile + orario) |
| Misura quadro | Sonoff POWCT / eWeLink |

Produzione mensile — Tapo vs Inverter diretto (invariato da v7):

| Mese | Tapo (foglio Anno) | Inverter diretto | Delta | Stato |
|---|---:|---:|---:|---|
| Marzo 2026 | 23,836 kWh | — | — | confermato (solo Tapo) |
| Aprile 2026 | 101,507 kWh | — | — | confermato (solo Tapo) |
| Maggio 2026 | 115,205 kWh | 92,45 kWh (09-31/05) | +8,0% (pulito) | confermato |
| Giugno 2026 | 124,482 kWh | 134,81 kWh (completo) | +8,3% | confermato |
| Luglio 2026 | — | 12,65 kWh (01-03/07) | — | da-aggiornare (parziale) |

---

## 6. NOVITÀ v8 — Conferma bolletta Plenitude Mar-Apr 2026 (fattura 2619841892)

### Verifica da PDF originale (emesso 09/05/2026)

| Campo | Valore DB (v7, `stima`) | Valore PDF (confermato) | Esito |
|---|---:|---:|---|
| Periodo | 01/03/2026–30/04/2026 | 01/03/2026–30/04/2026 | ✓ match |
| Consumo totale | 813 kWh | 813 kWh | ✓ match |
| Totale luce | 253,69 € | 253,69 € | ✓ match |
| Canone TV | 9,00 € | 9,00 € | ✓ match |
| Totale da pagare | 262,69 € | 262,69 € | ✓ match |
| Prezzo medio quota consumi | 0,204600 €/kWh | 0,204600 €/kWh | ✓ match |
| Quota fissa (2 mesi) | 14,025 €/mese | 14,025 €/mese | ✓ match |
| Quota potenza (2 mesi) | 8,89 €/mese | 8,89 €/mese | ✓ match |
| F1 (bimestre aggregato) | 217 kWh | 217 kWh | ✓ match |

### Attenzione — limite di verificabilità dello split F2/F3

La bolletta Plenitude è **bioraria**: il PDF originale riporta le letture aggregate come
**F1 = 217 kWh** e **F23 = 596 kWh** sul bimestre. **Non esiste una separazione F2/F3 nel
documento fiscale.** Lo split dettagliato già presente in DB (f2_kwh=257, f3_kwh=339, e il
dettaglio mensile Mar F1=151/F2=162/F3=187, Apr F1=66/F2=95/F3=152) **non è verificabile da
questo PDF** e resta una stima di derivazione esterna (probabile fonte: app Plenitude, che
può disaggregare F2/F3 anche se la bolletta fiscale non lo fa).

Lo storico consumi mensili totali a pag. 4 del PDF **conferma** invece i valori mensili
aggregati: Marzo = 500 kWh, Aprile = 313 kWh, coerenti col dato già presente.

### Azione svolta
- **Stato aggiornato**: `stima` → **`confermato`** per tutti i campi verificabili dal PDF
- **Nota DB aggiornata** con avvertenza esplicita sullo split F2/F3 non verificabile
- Commit `aa5dde3`

**Decisione ancora aperta:** se serve uno split F2/F3 realmente confermato per marzo/aprile
2026, va reperito dall'app Plenitude (Area Personale → Elementi di dettaglio) o accettato
come stima permanente, dato che il tipo di bolletta non lo espone.

---

## 7. NOVITÀ v8 — Anomalia 09/05/2026: risolta

### Il problema (aperto da v5, mai chiuso fino a v7)

| Data | Tapo (DB) | Inverter diretto (DB) | Delta |
|---|---:|---:|---:|
| 09/05/2026 | 4,021 kWh | 1,010 kWh | -3,011 kWh (-75%) |

Unico giorno con segno di delta opposto a tutti gli altri (che mostrano sempre inverter >
Tapo, +5%/+9%). Nessuna spiegazione conclusiva era stata trovata in v5, v6, v7.

### La scoperta (v8)

È stato caricato il **"Rapporto giornaliero" EnverView per il 09/05/2026** — il file
sorgente originale da cui derivava il dato "inverter=1,01 kWh" già in DB. L'ispezione ha
rivelato:

- Il campo Overview **"Today's Energy: 1.01 KWh"** = esattamente il valore già in DB
- Ma il **log dettagliato orario copre solo dalle 14:12:47 alle 19:08:15** — non l'intera
  giornata
- Somma reale dell'energia cumulata nella finestra coperta:
  **1,55 kWh (SN 30578336) + 1,82 kWh (SN 30578337) = 3,37 kWh**

### Interpretazione definitiva

```
Il valore "inverter=1.01 kWh" non era un errore di misura:
era il campo "Today's Energy" di EnverView, che risultava troncato
perché il log dettagliato parte solo dalle 14:12 — coincidente con
la finestra di changeover EVT560->EVT800 (9-17 maggio, §4).

La somma reale nella finestra coperta (3.37 kWh) è quasi il triplo
del valore registrato come "inverter" e più vicina (ma ancora sotto)
al dato Tapo di 4.021 kWh (presumibilmente giornata intera).

Conclusione: non e' un disaccordo tra le due fonti di misura.
E' una differenza di copertura temporale, causata dal cambio hardware
avvenuto proprio in quel giorno. Il 09/05/2026 resta un giorno
NON COMPARABILE tra Tapo e inverter -- ma ora con causa nota,
non piu' "anomalia da verificare".
```

### Azione svolta
- **294 righe orarie importate** in `letture_inverter_orarie` (14:12–19:08, entrambi i
  pannelli) — la copertura oraria continua del dataset ora parte dal 09/05 pomeriggio
  invece che dal 10/05
- `produzione_giornaliera`: campo `copertura_ore` corretto da 24 (errato) a 5 (reale) per
  il 09/05; nota riscritta con la spiegazione definitiva
- `verifica_bias_tapo_inverter`: nota aggiornata
- Il giorno **resta correttamente escluso** dal calcolo del bias medio dell'8,2-8,3% (§9),
  perché quel calcolo richiede copertura piena su entrambe le fonti
- Commit `beb8314`

**Pendenza residua:** non risolvibile ulteriormente senza un log installatore che indichi
l'orario esatto dello swap fisico dell'inverter — ma la causa del delta è ormai chiara e
documentata, non più un'anomalia aperta.

---

## 8. NOVITÀ v8 — Riverifica bolletta Sorgenia maggio 2026 (fattura V012604790978)

Caricato il PDF completo della bolletta Sorgenia già presente in DB. **Nessuna discrepanza
trovata** — il dato era già stato inserito correttamente in una sessione precedente:

| Campo | DB | PDF | Esito |
|---|---:|---:|---|
| N° fattura / data emissione | V012604790978, 18/06/2026 | V012604790978, 18/06/2026 | ✓ |
| F1/F2/F3 | 41,7/102,6/144,3 | 41,7/102,6/144,3 | ✓ |
| Totale kWh | 288,6 | 288,6 | ✓ |
| Totale luce | 88,69 € | 88,69 € | ✓ |
| Canone TV | 18,00 € (2 mesi: mag+giu, 9€ ciascuno) | 18,00 € | ✓ |
| Totale da pagare | 106,69 € | 106,69 € | ✓ |
| Prezzo medio | 0,201421 €/kWh | 0,201421 €/kWh | ✓ |
| Quota fissa | 8,19 €/mese | 8,19 €/mese | ✓ |
| Quota potenza | 8,89 €/mese | 8,89 €/mese | ✓ |
| Consumo annuo storico (Gen-Dic 2025) | F1=1479/F2=1615/F3=2054/tot=5148 | F1=1.479,0/F2=1.615,0/F3=2.054,0/tot=5.148,0 | ✓ |

**Nessuna azione DB necessaria.**

### Informazione tecnica nuova emersa dal PDF: formula tariffaria Sorgenia

L'offerta "Next Energy Sunlight" è a **prezzo variabile indicizzato al PUN** (non fissa
come la precedente Plenitude "Fixa Time Luce Base"), con formula esplicita:

```
Prezzo = (Perdite di Rete + PUN Index GME) + Dispacciamento + Fee
```

Valori PUN di riferimento maggio 2026 (dal PDF): F1=0,1072 €/kWh, F2=0,1314 €/kWh,
F3=0,1208 €/kWh.

**Implicazione metodologica (nuova, v8):** il prezzo medio Sorgenia **non è costante nel
tempo** come lo era Plenitude a prezzo fisso — si muove ogni mese col PUN. Questo significa
che il confronto "beneficio cambio fornitore ≈ 7-8 €/mese" (calcolato su un solo mese,
maggio) **non va assunto costante per i mesi successivi**. Ogni nuova bolletta Sorgenia
(giugno, luglio) andrà confrontata con il proprio PUN di riferimento, non con l'ipotesi di
prezzo Sorgenia fisso.

---

## 9. Bias Tapo vs Inverter diretto (invariato da v7, ora con nota 09/05 aggiornata)

| | Valore |
|---|---:|
| Giorni con entrambe le fonti (puliti) | 52 (escluso 09/05, ora con causa nota) |
| Totale Tapo (puliti) | 209,112 kWh |
| Totale Inverter (puliti) | 226,250 kWh |
| Delta aggregato | +17,138 kWh (+8,20%) |
| Media delta giornaliero | +8,33% |
| Deviazione standard | 1,37 punti percentuali |

**Decisione ancora in sospeso (invariata da v5):** promuovere l'inverter diretto a fonte
primaria per tutte le riconciliazioni future, o mantenere entrambe le fonti sempre
affiancate. Non decisa in questa sessione.

---

## 10. Dati presenti nel DB — bollette (invariato, ora tutte confermate salvo una)

### Bollette (fasce, importi, prezzi)

| Periodo | Fornitore | F1 | F2 | F3 | Tot kWh | Luce € | €/kWh | Fissa | Pot. | Stato |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| Mar-Apr 2025 | Plenitude | 217 | 269 | 314 | 800 | 249,84 | — | — | — | confermato |
| Mag-Giu 2025 | Plenitude | 227 | 243 | 331 | 801 | 236,93 | 0,200037 | 14,01 | 9,48 | confermato |
| Lug-Ago 2025 | Plenitude | 280 | 253 | 361 | 894 | 277,93 | 0,207383 | 14,005 | 9,48 | confermato |
| Set-Ott 2025 | Plenitude | 244 | 269 | 303 | 816 | 255,34 | 0,204216 | 14,005 | 9,48 | confermato |
| Nov-Dic 2025 | Plenitude | 276 | 276 | 400 | 952 | 286,09 | 0,201155 | 14,005 | 9,48 | confermato |
| Gen-Feb 2026 (baseline) | Plenitude | 297 | 306 | 347 | 950 | 290,35 | 0,206895 | 14,025 | 8,89 | confermato |
| **Mar-Apr 2026 (FV installato)** | Plenitude | 217 | 257 | 339 | 813 | 253,69 | 0,204600 | 14,025 | 8,89 | **confermato (v8)** — split F2/F3 non verificabile dal PDF, vedi §6 |
| Maggio 2026 | Sorgenia | 41,7 | 102,6 | 144,3 | 288,6 | 88,69 | 0,201421 | 8,19 | 8,89 | confermato (riverificato v8) |

Dettaglio mensile per confronti anno-su-anno (invariato):

| Mese | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Maggio 2025 | 95 | 125 | 141 | 361 |
| Giugno 2025 | 132 | 118 | 190 | 440 |
| Marzo 2026 | 151 | 162 | 187 | 500 |
| Aprile 2026 | 66 | 95 | 152 | 313 |
| Maggio 2026 | 41,7 | 102,6 | 144,3 | 288,6 |

**Nota:** il dettaglio mensile F1/F2/F3 di marzo/aprile 2026 in questa tabella riepilogativa
resta quello già noto, ma va ricordato che per marzo-aprile lo split F2/F3 non è confermabile
dal PDF originale (§6) — i totali mensili (500/313) sono invece confermati.

### Letture Sonoff/eWeLink — aggregato mensile (invariato da v7)

| Mese | Prelievo kWh | Immissione kWh | Note |
|---|---:|---:|---|
| Dic 2025 | 154,95 | 0,00 | parziale (dal 23/12) |
| Gen 2026 | 501,20 | 0,00 | = bolletta 502 kWh ✓ |
| Feb 2026 | 447,50 | 0,00 | = bolletta 448 kWh ✓ |
| Mar 2026 | 504,34 | 0,00 | FV a fine mese |
| Apr 2026 | 314,17 | 0,00 | pannelli riposizionati |
| Mag 2026 | 291,82 | 13,46 | immissione leggibile dal 18/05 |
| Giu 2026 | 339,92 | 6,74 | mese completo, corretto in v7 |
| Lug 2026 (parziale) | 48,49 | 0,78 | 01-04/07, 83 letture |

---

## 11. Confronti principali consolidati (invariati da v7)

### Maggio 2025 vs maggio 2026

| | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Maggio 2025 | 95 | 125 | 141 | 361 |
| Maggio 2026 | 41,7 | 102,6 | 144,3 | 288,6 |
| Differenza | −53,3 | −22,4 | +3,3 | −72,4 |
| Variazione | **−56,1%** | −17,9% | +2,3% | **−20,1%** |

### Cambio fornitore — effetto stimato (da rivalutare mese su mese, vedi §8)

```
Beneficio cambio fornitore (maggio) ≈ 7-8 €/mese  (soprattutto quota fissa più bassa)
Beneficio FV/minor prelievo         ≈ 45-50 €/mese (causa PRINCIPALE)
```
**Attenzione (v8):** con Sorgenia a prezzo variabile PUN, questa stima va ricalcolata per
ogni mese futuro (giugno, luglio), non estrapolata da maggio.

### Chiusura contabile giugno 2026 (piscina) — invariata da v7

| | Tapo (corretto) | Inverter diretto (corretto) |
|---|---:|---:|
| Consumo reale casa | 457,66 kWh | 467,99 kWh |
| Δ vs giugno 2025 (440 kWh) | +4,01% | **+6,36%** |
| Δ prelievo rete vs giugno 2025 | -22,75% | -22,75% |

---

## 12. Formule di riconciliazione (usare sempre)

```
Autoconsumo FV     = Produzione FV - Immissione
Consumo reale casa = Prelievo rete + Produzione FV - Immissione
```

Specificare sempre la fonte di Produzione FV usata (Tapo vs inverter diretto), dato lo
scarto sistematico dell'8,2-8,3%.

Verifica incrociata Sonoff vs bolletta:
- Gen 2026: Sonoff 501,2 / Bolletta 502 → −0,8 kWh (~0,2%) ✓
- Feb 2026: Sonoff 447,5 / Bolletta 448 → −0,5 kWh (~0,1%) ✓

Verifica incrociata produzione FV: foglio "Anno" vs somma foglio "Mese" → match esatto.
Verifica incrociata energia integrata da potenza oraria vs contatore mensile → scarto <1,1%.

**Buona pratica (v7, confermata v8):** verificare sempre completezza oraria (giorni×24)
prima di considerare un mese "chiuso" nei dati eWeLink/Sonoff.

**Buona pratica nuova (v8):** quando si riceve un file "Rapporto giornaliero" EnverView,
controllare sempre l'intervallo di copertura oraria effettivo del log dettagliato, non
fidarsi del solo campo di riepilogo "Today's Energy" — può essere troncato in giornate con
eventi hardware (v. §7).

---

## 13. Pendenze aperte

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **giugno 2026** | Sorgenia (attesa metà/fine luglio) | **Alta** | Serve per fasce F1/F2/F3 e conto economico definitivo del mese piscina; ricordare che il prezzo Sorgenia è variabile (PUN), da verificare contro il PUN di giugno |
| Bolletta Sorgenia luglio 2026 | Sorgenia | Futura | — |
| Split F2/F3 confermato per Mar-Apr 2026 | App Plenitude (non nel PDF fiscale) | Bassa | Il PDF Plenitude bioraria non separa F2/F3; resta stima esterna se non reperita altrove |
| Produzione FV **luglio 2026** — completare | Tapo + inverter | Media | Solo 3-4/31 giorni disponibili |
| Dati orari inverter **marzo–08 maggio 2026** | Inverter (export giornaliero) | Bassa | Ora la copertura parte dal 09/05 pomeriggio (v8), resta da coprire fino a inizio periodo installazione |
| **Decisione**: promuovere inverter a fonte primaria? | — | Media | In sospeso da v5, non decisa in v8 |
| Rigenerare token GitHub | — | Consigliata | Riutilizzato in più sessioni consecutive |

**Azione a più alto impatto per la prossima sessione:** ingest bolletta Sorgenia giugno
appena disponibile → completa il quadro economico del primo mese pieno con piscina, tenendo
conto della natura variabile del prezzo Sorgenia (§8).

---

## 14. Regole operative del progetto

1. **Canone TV sempre escluso** dai confronti energetici/economici.
2. Periodi bimestrali sempre **normalizzati su base mensile**.
3. Distinguere sempre: dato **confermato** (PDF/export verificato) / **stima** / **da aggiornare**.
4. Non attribuire risparmio al fornitore se i prezzi medi sono simili: verificare sempre se
   deriva da meno kWh prelevati.
5. **Segnale FV affidabile** = F1 cala, F3 stabile o in lieve aumento.
6. **No PII nel repo/handoff** (POD, intestatario, indirizzo, CF restano fuori) — regola
   rispettata anche con PDF di bollette caricati contenenti questi dati in chiaro.
7. Effetto piscina quantificato energeticamente, ricalcolato più volte (v4→v7): stima finale
   +4,01%/+6,36% a seconda della fonte FV.
8. Quando si citano dati di produzione FV, specificare sempre la fonte (Tapo vs inverter
   diretto) dato lo scarto sistematico dell'8,2-8,3%.
9. Token GitHub sempre fornito come file `token.txt`, mai in chiaro; eliminarlo dal disco
   subito dopo il push quando possibile.
10. I dati orari per pannello (`letture_inverter_orarie`) usano deduplica su `(sn, ts)`.
11. Prima di considerare "chiuso" un mese in `letture_quadro`, verificare completezza oraria
    (giorni×24).
12. Quando un dato viene ricalcolato a seguito di una correzione a monte, conservare sia il
    valore precedente sia quello corretto con versioning esplicito.
13. Coefficienti tecnici riportati dall'utente da fonti non verificabili direttamente da
    Claude vanno salvati con nota di provenienza esplicita.
14. **(Nuova, v8)** Per bollette biorarie (es. Plenitude "Fixa Time"), lo split F2/F3 non è
    presente nel PDF fiscale (solo F1/F23 aggregate): qualsiasi dettaglio F2/F3 più fine
    andrà sempre marcato come proveniente da fonte esterna (es. app), non dal PDF.
15. **(Nuova, v8)** Per bollette a prezzo variabile indicizzato (es. Sorgenia "Next Energy
    Sunlight"), il prezzo medio non è costante mese su mese: ogni confronto tariffario va
    ricalcolato sul PUN del mese specifico, non estrapolato da un mese di riferimento.
16. **(Nuova, v8)** Quando si riceve un export "Rapporto giornaliero" EnverView, verificare
    l'intervallo di copertura oraria effettivo prima di fidarsi del campo riepilogativo
    "Today's Energy" — può essere troncato in giorni con eventi hardware.

**Programma indicativo pompa piscina:**

| Fascia oraria | Stato pompa |
|---|---|
| 9/10 – 13 | Attiva |
| 13 – 16 | Pausa |
| 16 – 19 | Attiva |

---

## 15. Strumentazione

| Strumento | Dato misurato | Attendibilità |
|---|---|---|
| Sonoff POWCT / eWeLink | Prelievo orario dalla rete | Alta (validato su bollette); verificare completezza oraria |
| Sonoff POWCT / eWeLink | Immissione oraria in rete | Attendibile da ~18/05/2026 |
| Tapo (foglio Anno/Mese) | Produzione FV mensile/giornaliera | Media — bias di sottostima ~8,2-8,3% |
| Inverter diretto (export mensile/giornaliero) | Produzione FV mensile/giornaliera | Alta — misura di primo livello |
| Inverter diretto (letture orarie) | Potenza istantanea, temperatura, tensioni per pannello | Alta — verificato per integrazione (<1,1% scarto vs mensile); **attenzione a copertura troncata in giorni con eventi hardware (v8, §7)** |
| Bolletta Areti/distributore | Prelievo ufficiale fatturato | Dato definitivo |
| Etichetta pannello (foto) | Dati elettrici/meccanici nominali | Alta — letta direttamente da Claude |
| Datasheet ufficiale Dahai (coefficienti termici) | Coefficienti di temperatura | Media — riportati dall'utente, non verificati direttamente da Claude |
| **Bollette PDF originali (Plenitude/Sorgenia)** | **Totali, importi, prezzi ufficiali** | **Alta — fonte definitiva; ma attenzione ai limiti di disaggregazione fasce per bollette biorarie (v8)** |

---

## 16. Conclusione attuale del progetto

La riduzione osservata resta **coerente e verificata**, ora su una base dati più solida e
con due pendenze storiche chiuse in questa sessione.

```
Maggio 2025 → 361 kWh
Maggio 2026 → 288,6 kWh
Riduzione   → -72,4 kWh (-20,1%)
```

Dato decisivo (fascia diurna):
```
F1 maggio 2025 = 95 kWh
F1 maggio 2026 = 41,7 kWh   → -56,1%
F3 sostanzialmente invariata (+2,3%)
```

Decomposizione del risparmio (vs baseline gen-feb 2026):
- Fotovoltaico/autoconsumo → ~45-50 €/mese (causa **principale**)
- Cambio fornitore → ~7-8 €/mese (causa **secondaria**, e da ricalcolare mese su mese
  data la natura variabile del prezzo Sorgenia — nuova avvertenza v8)

**Aggiornamento v8 — due pendenze storiche chiuse:**
1. La bolletta Plenitude Mar-Apr 2026, `stima` dalla v3, è ora **confermata da PDF
   originale** — con l'avvertenza che lo split F2/F3 non è verificabile da questo tipo di
   bolletta (bioraria) e resta di fonte esterna.
2. L'anomalia 09/05/2026, aperta e mai risolta da v5 a v7, ha ora una **spiegazione
   definitiva**: sottocopertura oraria del log inverter durante il changeover hardware, non
   un errore di misura. 294 righe orarie importate, causa documentata.

La bolletta Sorgenia maggio, controllata di nuovo su PDF, si conferma priva di discrepanze
— ma il PDF rivela che l'offerta è a **prezzo variabile PUN**, informazione che va applicata
ai confronti tariffari futuri.

**Frontiera aperta (invariata):** manca ancora la bolletta Sorgenia di giugno per il conto
economico definitivo (€, fasce F1/F2/F3) del mese piscina. Resta anche da decidere se
promuovere l'inverter diretto a fonte primaria per tutte le riconciliazioni future.

---
*Fine handoff v8. Prossima azione consigliata: (1) ingest bolletta Sorgenia giugno appena
emessa, tenendo conto della sua formula a prezzo variabile PUN; (2) valutare se estendere la
copertura oraria anche a marzo-08 maggio; (3) decidere se promuovere l'inverter a fonte
primaria; (4) rigenerare il token GitHub per igiene di sicurezza; (5) se possibile, reperire
lo split F2/F3 di marzo-aprile 2026 dall'app Plenitude per upgrade da stima a confermato.*
