# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 9 — aggiornata al 04/07/2026

**Data:** 2026-07-04
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v9:** consolidamento completo dello stato del progetto. Nessun nuovo dato di consumo rispetto a v8 (bolletta Sorgenia giugno non ancora emessa). Aggiunta la nota metodologica sul **contratto Sorgenia** (Condizioni Generali + Modulo di Adesione esaminati): sconto fedeltà progressivo (5%→10%→15%→20%) e formula esatta del prezzo indicizzato. Confermato — tramite doppio invio identico da parte dell'utente — che non esistono discrepanze residue sulla bolletta Sorgenia maggio.

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

I dati identificativi — intestatario, indirizzo di fornitura, POD, codice fiscale, IBAN,
indirizzo email, telefono, indirizzo IP — **non vengono mai memorizzati né inclusi in repo,
handoff, DB o output di alcun tipo**, anche quando forniti esplicitamente dall'utente nelle
istruzioni di progetto o nei documenti caricati (bollette, contratti, moduli di adesione).
Questa regola è stata rispettata in modo continuo per tutta la sessione, inclusi i casi in
cui: le istruzioni progetto sono state re-inviate più volte con i dati in chiaro; le bollette
PDF (Plenitude e Sorgenia) contenevano nome, indirizzo, CF e POD; il Modulo di Adesione
Sorgenia conteneva anche IBAN e indirizzo IP.

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
- **Ultimo commit:** `1efc1ce` (04/07/2026) — handoff v8 pubblicato

### Come accedere in sessione futura
```bash
git clone --depth 1 https://github.com/mrennola/casa-bollette-db.git
cd casa-bollette-db
# poi python3 + sqlite3 per le query (row_factory = sqlite3.Row)
```
Nessun token necessario per la **lettura**. Per **scrivere** aggiornamenti serve il token
`claude-casa-bollette` (fine-grained, Contents Read/Write, scadenza 28/09/2026), fornito in
chat come file `token.txt` (mai incollato in chiaro). **Nota di sicurezza (invariata da
v6/v7/v8):** il token è stato riutilizzato più volte in sessioni successive — consigliata
rigenerazione su GitHub appena possibile.

### Struttura del database (invariata da v8)

| Tabella | Righe | Contenuto | Copertura |
|---|---:|---|---|
| `bollette` | 8 | Plenitude/Sorgenia, F1/F2/F3, importi | Mar 2025 → Mag 2026 |
| `produzione_fv` | 5 | Produzione FV mensile (Tapo + inverter) | Mar 2026 → Lug 2026 (parziale) |
| `produzione_giornaliera` | 92 | Serie giornaliera FV (Tapo + inverter) | 23/03 → 03/07/2026 |
| `letture_inverter_orarie` | 49.378 | Letture orarie per singolo microinverter | 09/05 (parziale, dalle 14:12) → 03/07/2026, continua |
| `letture_quadro` | 4.629 | Sonoff/eWeLink: prelievo + immissione | 23/12/2025 → 04/07/2026 10:00 |
| `cronologia_eventi` | 10 | Date eventi tecnici | 05/2025 → 06/2026 |
| `fornitori_storico` | 3 | Prezzi medi comparati | Mag25 / Mar26 / Mag26 |
| `verifica_bias_tapo_inverter` | 53 | Confronto giornaliero Tapo vs inverter | 09/05 → 30/06/2026 |
| `riconciliazione_mensile` | 4 | Storico versioni calcolo chiusura piscina giugno | v4-v6 (superato) + v7 (corretto) |
| `pannelli_fv_specifiche` | 1 | Scheda tecnica pannello Dahai | — |

Schema `letture_inverter_orarie`: `sn`, `ts`, `vdc`, `vac`, `potenza_w`, `freq_hz`,
`temperatura_c`, `energia_tot_kwh` (contatore cumulato lifetime), `fonte_file`.
Deduplicazione su `(sn, ts)`.

Aggregazione mensile: `substr(ts,1,7)`; giornaliera: `substr(ts,1,10)`.

---

## 4. Dati tecnici dell'utenza (no PII)

| Campo | Dato |
|---|---|
| Distributore | Areti |
| Tipologia cliente | Domestico residente |
| Potenza impegnata | 4,5 kW |
| Potenza disponibile | 5,0 kW |
| Tensione | 220 V |
| Misuratore | 2G |

---

## 5. Cronologia tecnica rilevante

| Data | Evento | Impatto atteso |
|---|---|---|
| Mag-Giu 2025 | Periodo pre-FV anno precedente | Benchmark storico |
| Gen-Feb 2026 | Periodo benchmark pre-FV | Riferimento pulito senza FV |
| 23/24 mar 2026 | Installazione iniziale FV plug & play | Marzo mese misto |
| 04 apr 2026 | Riposizionamento/ottimizzazione pannelli | Da qui il calo F1 è più leggibile |
| 30 apr 2026 | Fine contratto/fattura Plenitude | Chiusura vecchio fornitore |
| **25 mar 2026** | **Sottoscrizione contratto Sorgenia (data adesione)** | Contratto n. 1221092/WB |
| 01 mag 2026 | Decorrenza Sorgenia | Inizio nuovo fornitore |
| **9 mag 2026 (14:12 circa)** | Inizio changeover inverter EVT560→EVT800 | Causa nota dell'anomalia 09/05 (§9) |
| 9/17 mag 2026 | Completamento cambio inverter → Envertech EVT800 | Maggiore potenza massima FV |
| 18 mag 2026 | Sonoff legge bene supply/immissione | Da qui immissione attendibile |
| 30 mag 2026 | Installazione piscina | Nuovo carico elettrico diurno |
| Da giu 2026 | Pompa piscina attiva tutto il mese | Aumenta consumo reale e autoconsumo FV |
| 04 lug 2026 | Sessione v9: consolidamento handoff | — |

---

## 6. Fotovoltaico — configurazione

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

### Effetto riposizionamento pannelli (confermato, dato Tapo)
| Periodo | n giorni | Media |
|---|---:|---:|
| Pre-riposizionamento (23 mar – 6 apr) | 10 | 2,27 kWh/gg |
| Post-riposizionamento (7 apr – 1 mag) | 17 | 3,71 kWh/gg |
| **Incremento** | | **+63,9%** |

### Produzione mensile — Tapo vs Inverter diretto

| Mese | Tapo (foglio Anno) | Inverter diretto | Delta | Stato |
|---|---:|---:|---:|---|
| Marzo 2026 | 23,836 kWh | — | — | confermato (solo Tapo) |
| Aprile 2026 | 101,507 kWh | — | — | confermato (solo Tapo) |
| Maggio 2026 | 115,205 kWh | 92,45 kWh (09-31/05) | +8,0% (pulito) | confermato |
| Giugno 2026 | 124,482 kWh | 134,81 kWh (completo) | +8,3% | confermato |
| Luglio 2026 | — | 12,65 kWh (01-03/07) | — | da-aggiornare (parziale) |

### Bias sistematico Tapo vs Inverter diretto

| | Valore |
|---|---:|
| Giorni con entrambe le fonti (puliti) | 52 |
| Delta aggregato | +17,138 kWh (+8,20%) |
| Media delta giornaliero | +8,33% |
| Deviazione standard | 1,37 punti percentuali |

**Decisione ancora in sospeso:** promuovere l'inverter diretto a fonte primaria per tutte le
riconciliazioni future, o mantenere entrambe le fonti sempre affiancate. Non decisa.

### Picco di potenza istantanea (15/06/2026)

| | Pannello 30578336 | Pannello 30578337 | Somma (array) |
|---|---:|---:|---:|
| Potenza massima | 375,94 W | 392,70 W | **768,64 W** |
| % del nominale (800 Wp) | 94,0% | 98,2% | **96,1%** |

Nessun clipping quel giorno specifico — coerente col pattern stagionale (§10).

### Identificazione pannelli (v7)

| Parametro | Valore |
|---|---:|
| Produttore | Shandong Dahai Group |
| Modello | Dahai Solar DHM54T35-400/MR |
| Pmax | 400 W (+5W) |
| Efficienza | 20,48% |
| Coeff. termico Pmax | -0,350%/°C *(riportato dall'utente, non verificato direttamente da Claude — datasheet non testuale)* |
| Garanzia | 15 anni prodotto (20 opzionali), 25 anni lineare potenza |

---

## 7. Contratto Sorgenia — dettagli emersi da CGC e Modulo di Adesione (nuovo, v9)

Documenti esaminati: Condizioni Generali/Economiche "Next Energy Sunlight/Luce" e Modulo di
Adesione (contratto n. 1221092/WB, sottoscritto 25/03/2026). Nessun impatto sul DB — sono
documenti contrattuali di riferimento, non dati di consumo.

### Formula tariffaria esatta (primo anno di fornitura)
```
Prezzo Indicizzato Luce = PUN Index GME × (1 + perdite di rete) + Fee
```
Dopo i primi 12 mesi si aggiunge anche l'Indice GO (valore di mercato delle Garanzie di
Origine), aggiornato mensilmente.

### Sconto Fedeltà Luce — progressivo nel tempo (informazione nuova, rilevante)

| Mese di fornitura | Sconto sul Servizio Commerciale |
|---|---:|
| 1–12 | 5% |
| 13–24 | 10% |
| 25–36 | 15% |
| 37+ | 20% |

**Implicazione metodologica:** il beneficio tariffario Sorgenia rispetto a Plenitude **non è
costante nel tempo** per due motivi indipendenti, entrambi da tenere presenti nei confronti
futuri:
1. Il prezzo energia è variabile e indicizzato al PUN (già noto da v8).
2. Lo sconto fedeltà **aumenta progressivamente** (dal 5% attuale fino al 20% al 37° mese).

Di conseguenza il confronto "beneficio cambio fornitore ≈ 7-8 €/mese" calcolato su maggio
2026 non va estrapolato ai mesi futuri: andrà ricalcolato ogni volta con il PUN e con lo
scaglione di sconto fedeltà applicabile a quel mese.

### Altri dettagli contrattuali
- Fatturazione mensile (non bimestrale, a differenza di Plenitude)
- Nessun onere di recesso anticipato
- Scadenza condizioni economiche: aprile 2027
- Codice offerta: 000390ESVFL01XXES013079010000000

---

## 8. Conferma bolletta Plenitude Mar-Apr 2026 (v8, invariata)

### Verifica da PDF originale (fattura 2619841892, emessa 09/05/2026)

| Campo | Valore |
|---|---:|
| Consumo totale | 813 kWh — confermato |
| Totale luce | 253,69 € — confermato |
| Canone TV | 9,00 € — confermato |
| Totale da pagare | 262,69 € — confermato |
| Prezzo medio quota consumi | 0,204600 €/kWh — confermato |
| F1 (bimestre aggregato) | 217 kWh — confermato |

### Limite di verificabilità dello split F2/F3

La bolletta Plenitude è **bioraria**: il PDF riporta solo F1 = 217 kWh e F23 = 596 kWh sul
bimestre aggregato. **Non esiste separazione F2/F3 nel documento fiscale.** Lo split
dettagliato (f2=257/f3=339 e il dettaglio mensile Mar F1=151/F2=162/F3=187, Apr F1=66/F2=95/
F3=152, presente anche nelle istruzioni progetto standard) **non è verificabile da questo
PDF** e resta una stima di derivazione esterna (probabile fonte: app Plenitude).

Lo storico consumi mensili totali a pag. 4 del PDF **conferma** i valori mensili aggregati:
Marzo = 500 kWh, Aprile = 313 kWh.

**Stato DB:** `confermato` per tutti i campi verificabili dal PDF; nota esplicita nel record
sullo split F2/F3 non confermabile.

---

## 9. Anomalia 09/05/2026 — risolta (v8, invariata)

| Data | Tapo (DB) | Inverter diretto (DB) | Delta |
|---|---:|---:|---:|
| 09/05/2026 | 4,021 kWh | 1,010 kWh | -3,011 kWh (-75%) |

**Causa identificata:** il valore "inverter=1,01 kWh" già in DB coincideva col campo
"Today's Energy" del Rapporto giornaliero EnverView per quel giorno, che risultava troncato
perché il log dettagliato copre solo dalle 14:12:47 alle 19:08:15 — coincidente con la
finestra di changeover inverter EVT560→EVT800. Somma reale registrata nella finestra
coperta: 1,55 + 1,82 = **3,37 kWh**, quasi il triplo del valore registrato come "inverter" e
più vicino (ma ancora sotto) al dato Tapo di 4,021 kWh.

**Conclusione:** non è un errore di misura di nessuna delle due fonti — è una differenza di
copertura temporale causata dal cambio hardware. Il giorno resta non comparabile tra le
fonti, ma con causa nota e documentata.

**Azione svolta:** 294 righe orarie importate in `letture_inverter_orarie` (14:12–19:08,
entrambi i pannelli); `produzione_giornaliera.copertura_ore` corretto da 24 a 5; note
aggiornate in `produzione_giornaliera` e `verifica_bias_tapo_inverter`.

---

## 10. Clipping dell'inverter (v6, invariato)

| Periodo | Giorni con clipping | % giorni | Durata media plateau |
|---|---:|---:|---:|
| Maggio (22 gg) | 7 | 31,8% | 55 min |
| Giugno (30 gg) | 5 | 16,7% | 9 min |
| Luglio (3 gg) | 0 | 0% | — |
| **Totale (55 gg)** | **12** | **21,8%** | **36 min** |

Pattern stagionale netto: frequente a metà maggio (temperature moderate), raro a giugno
(perdita termica tiene la produzione sotto 800W), assente nei 3 giorni di luglio disponibili.

---

## 11. Effetto piscina isolato per fascia oraria (v6, invariato)

Confronto immissione media oraria, pre-piscina (18-29 maggio) vs post-piscina (30 maggio-17
giugno):

| Fascia | Immissione media pre-piscina | Immissione media post-piscina | Delta |
|---|---:|---:|---:|
| Pompa attiva (9-13, 16-19) | 0,572 kWh | 0,109 kWh | **-80,9%** |
| Pompa in pausa (13-16) | 0,449 kWh | 0,218 kWh | **-51,4%** |

Il calo dell'immissione è quasi doppio nelle ore con pompa programmata attiva — riscontro
diretto e granulare del meccanismo pompa→autoconsumo.

---

## 12. Chiusura contabile giugno 2026 (piscina) — versione corrente

Corretta in v7 dopo la scoperta di un buco di 10h nei dati eWeLink di giugno (334,05→339,92
kWh prelievo; 6,59→6,74 kWh immissione).

| | Tapo (v7, corretto) | Inverter diretto (v7, corretto) |
|---|---:|---:|
| Prelievo rete | 339,92 kWh | 339,92 kWh |
| Produzione FV | 124,482 kWh | 134,81 kWh |
| Immissione | 6,74 kWh | 6,74 kWh |
| Autoconsumo FV | 117,74 kWh | 128,07 kWh |
| **Consumo reale casa** | **457,66 kWh** | **467,99 kWh** |
| Δ vs giugno 2025 (440 kWh) | +4,01% | **+6,36%** |
| Δ prelievo rete vs giugno 2025 | -22,75% | -22,75% |

```
Autoconsumo FV giugno (Tapo)     = 124,482 - 6,74 = 117,74 kWh
Consumo reale casa (Tapo)        = 339,92 + 117,74 = 457,66 kWh

Autoconsumo FV giugno (Inverter) = 134,81 - 6,74 = 128,07 kWh
Consumo reale casa (Inverter)    = 339,92 + 128,07 = 467,99 kWh
```

**Lettura:** il fotovoltaico continua ad assorbire la maggior parte del carico piscina; il
prelievo dalla rete resta in calo del 22,75% rispetto a giugno 2025 nonostante la piscina
attiva tutto il mese. **Nota (v7, conservata):** la tabella `riconciliazione_mensile`
mantiene lo storico di tutte le versioni di questo calcolo (v4→v7), mai sovrascritte.

---

## 13. Dati presenti nel DB — bollette

### Bollette (fasce, importi, prezzi)

| Periodo | Fornitore | F1 | F2 | F3 | Tot kWh | Luce € | €/kWh | Fissa | Pot. | Stato |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| Mar-Apr 2025 | Plenitude | 217 | 269 | 314 | 800 | 249,84 | — | — | — | confermato |
| Mag-Giu 2025 | Plenitude | 227 | 243 | 331 | 801 | 236,93 | 0,200037 | 14,01 | 9,48 | confermato |
| Lug-Ago 2025 | Plenitude | 280 | 253 | 361 | 894 | 277,93 | 0,207383 | 14,005 | 9,48 | confermato |
| Set-Ott 2025 | Plenitude | 244 | 269 | 303 | 816 | 255,34 | 0,204216 | 14,005 | 9,48 | confermato |
| Nov-Dic 2025 | Plenitude | 276 | 276 | 400 | 952 | 286,09 | 0,201155 | 14,005 | 9,48 | confermato |
| Gen-Feb 2026 (baseline) | Plenitude | 297 | 306 | 347 | 950 | 290,35 | 0,206895 | 14,025 | 8,89 | confermato |
| Mar-Apr 2026 (FV installato) | Plenitude | 217 | 257 | 339 | 813 | 253,69 | 0,204600 | 14,025 | 8,89 | confermato — split F2/F3 non verificabile dal PDF (bioraria), vedi §8 |
| Maggio 2026 | Sorgenia | 41,7 | 102,6 | 144,3 | 288,6 | 88,69 | 0,201421 | 8,19 | 8,89 | confermato — riverificato due volte, nessuna discrepanza |

Dettaglio mensile per confronti anno-su-anno:

| Mese | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Maggio 2025 | 95 | 125 | 141 | 361 |
| Giugno 2025 | 132 | 118 | 190 | 440 |
| Marzo 2026 | 151 | 162 | 187 | 500 |
| Aprile 2026 | 66 | 95 | 152 | 313 |
| Maggio 2026 | 41,7 | 102,6 | 144,3 | 288,6 |

**Nota:** il dettaglio F1/F2/F3 di marzo/aprile in questa tabella resta la stima esterna già
nota; solo i totali mensili (500/313) sono confermati dal PDF Plenitude.

### Letture Sonoff/eWeLink — aggregato mensile

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

## 14. Confronti principali consolidati

### Maggio 2025 vs maggio 2026

| | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Maggio 2025 | 95 | 125 | 141 | 361 |
| Maggio 2026 | 41,7 | 102,6 | 144,3 | 288,6 |
| Differenza | −53,3 | −22,4 | +3,3 | −72,4 |
| Variazione | **−56,1%** | −17,9% | +2,3% | **−20,1%** |

Lettura: F1 crolla del 56%, F3 stabile → coerente con effetto FV.

### Aprile 2026 vs maggio 2026

| | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Aprile 2026 | 66 | 95 | 152 | 313 |
| Maggio 2026 | 41,7 | 102,6 | 144,3 | 288,6 |
| Differenza | −24,3 | +7,6 | −7,7 | −24,4 |

Il calo aprile-maggio è quasi tutto in F1.

### Baseline Gen-Feb 2026 vs maggio 2026
```
Media mensile baseline = 475 kWh/mese
Maggio 2026 = 288,6 kWh
Riduzione = -186,4 kWh/mese = -39,2%
```

### Cambio fornitore — effetto stimato per maggio (da ricalcolare per mesi futuri, §7)

| Voce | Plenitude Mar-Apr 2026 | Sorgenia Mag 2026 |
|---|---:|---:|
| Prezzo medio quota consumi | 0,204600 €/kWh | 0,201421 €/kWh |
| Quota fissa | 14,025 €/mese | 8,19 €/mese |
| Quota potenza | 8,89 €/mese | 8,89 €/mese |

```
Beneficio cambio fornitore (maggio) ≈ 7-8 €/mese  (soprattutto quota fissa più bassa)
Beneficio FV/minor prelievo          ≈ 45-50 €/mese (causa PRINCIPALE)
```
**Attenzione (v9):** questa stima è specifica di maggio. Per giugno e mesi successivi va
ricalcolata considerando sia il PUN del mese sia lo scaglione di sconto fedeltà applicabile
(§7) — il beneficio tariffario Sorgenia tenderà a crescere nel tempo per via dello sconto
fedeltà progressivo, indipendentemente dalle variazioni del PUN.

Il prezzo Sorgenia non è molto più basso di quello Plenitude: il risparmio nasce
soprattutto dal minor prelievo, non dal prezzo dell'energia.

---

## 15. Formule di riconciliazione (usare sempre)

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

**Buona pratica:** verificare sempre completezza oraria (giorni×24) prima di considerare un
mese "chiuso" nei dati eWeLink/Sonoff. Quando si riceve un file "Rapporto giornaliero"
EnverView, controllare sempre l'intervallo di copertura oraria effettivo del log dettagliato,
non fidarsi del solo campo "Today's Energy" — può essere troncato in giorni con eventi
hardware (v. §9).

---

## 16. Pendenze aperte (dati da completare)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **giugno 2026** | Sorgenia (attesa metà/fine luglio) | **Alta** | Serve per fasce F1/F2/F3 e conto economico definitivo del mese piscina; ricalcolare con PUN di giugno E scaglione sconto fedeltà applicabile |
| Bolletta Sorgenia luglio 2026 | Sorgenia | Futura | — |
| Split F2/F3 confermato per Mar-Apr 2026 | App Plenitude (non nel PDF fiscale) | Bassa | Il PDF Plenitude bioraria non separa F2/F3; resta stima esterna se non reperita altrove |
| Produzione FV **luglio 2026** — completare | Tapo + inverter | Media | Solo 3-4/31 giorni disponibili |
| Dati orari inverter **marzo–08 maggio 2026** | Inverter (export giornaliero) | Bassa | La copertura parte ora dal 09/05 pomeriggio |
| **Decisione**: promuovere inverter a fonte primaria? | — | Media | In sospeso da v5 |
| Rigenerare token GitHub | — | Consigliata | Riutilizzato in più sessioni consecutive |

**Azione a più alto impatto per la prossima sessione:** ingest bolletta Sorgenia giugno
appena disponibile, applicando sia il PUN del mese sia lo scaglione di sconto fedeltà
corretto (mese 3 di fornitura → ancora 5%, dato che il contratto è datato 25/03/2026).

---

## 17. Metodo standard per ogni nuova bolletta/mese

Quando si analizza una nuova bolletta o un nuovo mese, produrre sempre:

1. Riepilogo dati principali
2. Consumi per fascia F1/F2/F3
3. Confronto con mese precedente
4. Confronto con stesso mese anno precedente, se disponibile
5. Confronto con baseline pre-FV (Gen-Feb 2026)
6. Costo medio energia e costo medio all-in luce
7. Separazione effetto kWh vs effetto tariffa (**ora includendo lo scaglione sconto
   fedeltà applicabile, v9**)
8. Esclusione del canone TV dal confronto energetico
9. Eventuale impatto piscina
10. Tabella finale con conclusione netta

Domanda di controllo su ogni risparmio osservato: *è risparmio da prezzo più basso o da
meno kWh prelevati?*

---

## 18. Regole operative del progetto

1. **Canone TV sempre escluso** dai confronti energetici/economici.
2. Periodi bimestrali sempre **normalizzati su base mensile**.
3. Distinguere sempre: dato **confermato** (PDF/export verificato) / **stima** / **da
   aggiornare**.
4. Non attribuire risparmio al fornitore se i prezzi medi sono simili: verificare sempre se
   deriva da meno kWh prelevati.
5. **Segnale FV affidabile** = F1 cala, F3 stabile o in lieve aumento.
6. **No PII nel repo/handoff** (POD, intestatario, indirizzo, CF, IBAN, email, telefono, IP
   restano fuori) — regola rispettata anche con documenti contenenti questi dati in chiaro.
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
14. Per bollette biorarie (es. Plenitude "Fixa Time"), lo split F2/F3 non è presente nel PDF
    fiscale: qualsiasi dettaglio F2/F3 più fine va sempre marcato come proveniente da fonte
    esterna.
15. Per bollette a prezzo variabile indicizzato (es. Sorgenia), il prezzo medio non è
    costante mese su mese: ogni confronto tariffario va ricalcolato sul PUN del mese
    specifico.
16. Quando si riceve un export "Rapporto giornaliero" EnverView, verificare l'intervallo di
    copertura oraria effettivo prima di fidarsi del campo riepilogativo "Today's Energy".
17. **(Nuova, v9)** Per il contratto Sorgenia, tenere conto anche dello **sconto fedeltà
    progressivo** (5% mesi 1-12, 10% mesi 13-24, 15% mesi 25-36, 20% dal mese 37) nel
    calcolo del beneficio tariffario mese su mese — non solo il PUN.
18. **(Nuova, v9)** Prima di elaborare un documento caricato, verificare se è un duplicato
    di un file già processato in sessione (stesso numero fattura/contratto) per evitare
    analisi ridondanti — verificato con successo in questa sessione su un doppio invio della
    bolletta Sorgenia maggio.

**Programma indicativo pompa piscina:**

| Fascia oraria | Stato pompa |
|---|---|
| 9/10 – 13 | Attiva |
| 13 – 16 | Pausa |
| 16 – 19 | Attiva |

---

## 19. Strumentazione

| Strumento | Dato misurato | Attendibilità |
|---|---|---|
| Sonoff POWCT / eWeLink | Prelievo orario dalla rete | Alta (validato su bollette); verificare completezza oraria |
| Sonoff POWCT / eWeLink | Immissione oraria in rete | Attendibile da ~18/05/2026 |
| Tapo (foglio Anno/Mese) | Produzione FV mensile/giornaliera | Media — bias di sottostima ~8,2-8,3% |
| Inverter diretto (export mensile/giornaliero) | Produzione FV mensile/giornaliera | Alta — misura di primo livello |
| Inverter diretto (letture orarie) | Potenza istantanea, temperatura, tensioni per pannello | Alta — verificato per integrazione (<1,1% scarto vs mensile); attenzione a copertura troncata in giorni con eventi hardware |
| Bolletta Areti/distributore | Prelievo ufficiale fatturato | Dato definitivo |
| Etichetta pannello (foto) | Dati elettrici/meccanici nominali | Alta — letta direttamente da Claude |
| Datasheet ufficiale Dahai (coefficienti termici) | Coefficienti di temperatura | Media — riportati dall'utente, non verificati direttamente da Claude |
| Bollette PDF originali (Plenitude/Sorgenia) | Totali, importi, prezzi ufficiali | Alta — fonte definitiva; attenzione ai limiti di disaggregazione fasce per bollette biorarie |
| Contratto Sorgenia (CGC + Modulo Adesione) | Formula tariffaria, sconto fedeltà, condizioni | Alta — letto direttamente da Claude |

---

## 20. Conclusione attuale del progetto

La riduzione osservata resta **coerente e verificata**, ora su una base dati ancora più
solida e con due pendenze storiche chiuse (conferma Mar-Apr 2026, anomalia 09/05).

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

Il calo è concentrato nella fascia diurna (F1), coerente con l'effetto fotovoltaico: nessun
vero abbattimento della fascia notturna.

Decomposizione del risparmio (vs baseline gen-feb 2026, riferita a maggio):
- Fotovoltaico/autoconsumo → ~45-50 €/mese (causa **principale**)
- Cambio fornitore → ~7-8 €/mese (causa **secondaria**, e destinata a **crescere nel tempo**
  per via dello sconto fedeltà progressivo Sorgenia — nuova nota v9, indipendente dalle
  oscillazioni del PUN)

**Chiusura giugno 2026 (piscina), dato corretto:** il consumo reale della casa cresce del
+4,01% (Tapo) o +6,36% (inverter) rispetto a giugno 2025, nonostante la piscina attiva tutto
il mese, mentre il prelievo dalla rete cala comunque del -22,75%. Isolato anche per fascia
oraria: l'immissione crolla dell'80,9% nelle ore con pompa attiva contro il 51,4% nelle ore
di pausa — la firma più diretta ottenuta finora del meccanismo pompa→autoconsumo.

**Frontiera aperta:** manca la bolletta Sorgenia di giugno per il conto economico definitivo
(€, fasce F1/F2/F3) del mese piscina. Quando arriverà, andrà applicato sia il PUN di giugno
sia lo scaglione di sconto fedeltà corretto (ancora 5%, essendo il terzo mese di fornitura).
Resta anche da decidere se promuovere l'inverter diretto a fonte primaria per tutte le
riconciliazioni future.

---
*Fine handoff v9. Prossima azione consigliata: (1) ingest bolletta Sorgenia giugno appena
emessa, applicando PUN + sconto fedeltà del mese; (2) valutare se estendere la copertura
oraria a marzo-08 maggio; (3) decidere se promuovere l'inverter a fonte primaria; (4)
rigenerare il token GitHub; (5) completare produzione FV luglio con più giorni.*
