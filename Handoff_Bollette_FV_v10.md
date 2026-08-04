# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 10 — aggiornata al 04/08/2026

**Data:** 2026-08-04
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v10:**
1. **Copertura oraria inverter estesa**: 34 nuovi export EnverView "Rapporto giornaliero" (02/07 → 04/08/2026, con l'ultimo giorno parziale/in corso) caricati e deduplicati su `(sn, ts)`. `letture_inverter_orarie` passa da 49.378 a **76.928 righe**.
2. **Luglio 2026 chiuso** (fonte inverter diretto, Tapo mai disponibile per questo mese): **137,92 kWh**, 31/31 giorni.
3. **Agosto 2026 avviato** (parziale, 4 giorni, 14,16 kWh, ultimo giorno in corso).
4. **Bolletta Sorgenia giugno 2026 ricevuta e ingerita** — la pendenza a priorità più alta del progetto è chiusa. Fasce F1/F2/F3 reali, prezzo PUN-indicizzato di giugno, sconto fedeltà 5% confermato.
5. **Chiusura contabile definitiva del mese piscina (giugno 2026)**, ora basata sul prelievo da bolletta ufficiale (344,5 kWh) anziché sulla stima Sonoff — nuova versione `v10 (definitivo, bolletta ufficiale)` in `riconciliazione_mensile`.
6. **Scoperta rilevante**: a giugno il beneficio del cambio fornitore si è quasi azzerato (PUN alto quel mese) — il risparmio è quasi interamente dovuto al fotovoltaico. Vedi §8.

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
indirizzo email, telefono, indirizzo IP, codice cliente — **non vengono mai memorizzati né
inclusi in repo, handoff, DB o output di alcun tipo**, anche quando forniti esplicitamente
dall'utente nei documenti caricati (bollette, contratti). Questa regola è stata rispettata
anche nella sessione corrente, in cui la bolletta Sorgenia giugno 2026 (PDF completo,
7 pagine) conteneva nome, indirizzo, CF, POD, codice cliente e coordinate SDD in chiaro.

Numero di fattura: coerentemente con le versioni precedenti (già presente per Plenitude e
Sorgenia maggio), il numero di fattura elettronica è considerato dato tecnico/identificativo
del documento (non della persona) e viene mantenuto nelle note per tracciabilità.

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
- **Ultimo commit prima di questa sessione:** `1efc1ce` (handoff v9)

### Come accedere in sessione futura
```bash
git clone --depth 1 https://github.com/mrennola/casa-bollette-db.git
cd casa-bollette-db
# poi python3 + sqlite3 per le query (row_factory = sqlite3.Row)
```
Nessun token necessario per la **lettura**. Per **scrivere** aggiornamenti serve il token
`claude-casa-bollette` (fine-grained, Contents Read/Write, scadenza 28/09/2026), fornito in
chat come file `token.txt` (mai incollato in chiaro). **Nota di sicurezza (invariata da
v6-v9):** il token è stato riutilizzato più volte in sessioni successive — consigliata
rigenerazione su GitHub appena possibile.

### Struttura del database (aggiornata v10)

| Tabella | Righe | Contenuto | Copertura |
|---|---:|---|---|
| `bollette` | **9** (era 8) | Plenitude/Sorgenia, F1/F2/F3, importi | Mar 2025 → **Giugno 2026** |
| `produzione_fv` | **6** (era 5) | Produzione FV mensile (Tapo + inverter) | Mar 2026 → **Ago 2026 (parziale)** |
| `produzione_giornaliera` | **124** (era 92) | Serie giornaliera FV (Tapo + inverter) | 23/03 → **04/08/2026** |
| `letture_inverter_orarie` | **76.928** (era 49.378) | Letture orarie per singolo microinverter | 09/05 (parziale) → **04/08/2026 11:08** |
| `letture_quadro` | 4.629 (invariato) | Sonoff/eWeLink: prelievo + immissione | 23/12/2025 → 04/07/2026 10:00 |
| `cronologia_eventi` | 10 | Date eventi tecnici | 05/2025 → 06/2026 |
| `fornitori_storico` | 3 | Prezzi medi comparati | Mag25 / Mar26 / Mag26 |
| `verifica_bias_tapo_inverter` | 53 (invariato) | Confronto giornaliero Tapo vs inverter | 09/05 → 30/06/2026 |
| `riconciliazione_mensile` | **6** (era 4) | Storico versioni calcolo chiusura piscina giugno | v4-v6 (superato), v7 (superato), **v10 definitivo** |
| `pannelli_fv_specifiche` | 1 | Scheda tecnica pannello Dahai | — |

**Nota importante (v10):** `letture_quadro` (Sonoff/eWeLink) **non è stata aggiornata** in
questa sessione — resta ferma al 04/07/2026 h10:00. È una fonte diversa (contatore rete, non
FV) rispetto ai 34 file EnverView caricati. Gap aperto: 04/07 h10 → oggi.

Schema `letture_inverter_orarie`: `sn`, `ts`, `vdc`, `vac`, `potenza_w`, `freq_hz`,
`temperatura_c`, `energia_tot_kwh` (contatore cumulato lifetime), `fonte_file`.
Deduplicazione su `(sn, ts)` — confermata funzionante: 1.792 righe (02-03/07, già presenti)
correttamente scartate come duplicati durante l'import v10.

Aggregazione mensile: `substr(ts,1,7)`; giornaliera: `substr(ts,1,10)`.

---

## 4. Cronologia tecnica rilevante

| Data | Evento | Impatto atteso |
|---|---|---|
| Mag-Giu 2025 | Periodo pre-FV anno precedente | Benchmark storico |
| Gen-Feb 2026 | Periodo benchmark pre-FV | Riferimento pulito senza FV |
| 23/24 mar 2026 | Installazione iniziale FV plug & play | Marzo mese misto |
| 04 apr 2026 | Riposizionamento/ottimizzazione pannelli | Da qui il calo F1 è più leggibile |
| 25 mar 2026 | Sottoscrizione contratto Sorgenia (data adesione) | Contratto n. 1221092/WB |
| 30 apr 2026 | Fine contratto/fattura Plenitude | Chiusura vecchio fornitore |
| 01 mag 2026 | Decorrenza Sorgenia | Inizio nuovo fornitore |
| 9 mag 2026 (14:12 circa) | Inizio changeover inverter EVT560→EVT800 | Causa nota dell'anomalia 09/05 |
| 9/17 mag 2026 | Completamento cambio inverter → Envertech EVT800 | Maggiore potenza massima FV |
| 18 mag 2026 | Sonoff legge bene supply/immissione | Da qui immissione attendibile |
| 30 mag 2026 | Installazione piscina | Nuovo carico elettrico diurno |
| Da giu 2026 | Pompa piscina attiva tutto il mese | Aumenta consumo reale e autoconsumo FV |
| 16/07/2026 | Emissione bolletta Sorgenia giugno 2026 | Ricevuta e ingerita in questa sessione |
| **04 ago 2026** | **Sessione v10: import inverter esteso, chiusura definitiva giugno** | — |

---

## 5. Fotovoltaico — configurazione

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

### Produzione mensile — Tapo vs Inverter diretto (aggiornata v10)

| Mese | Tapo (foglio Anno) | Inverter diretto | Delta | Stato |
|---|---:|---:|---:|---|
| Marzo 2026 | 23,836 kWh | — | — | confermato (solo Tapo) |
| Aprile 2026 | 101,507 kWh | — | — | confermato (solo Tapo) |
| Maggio 2026 | 115,205 kWh | 92,45 kWh (09-31/05) | +8,0% (pulito) | confermato |
| Giugno 2026 | 124,482 kWh | 134,81 kWh (completo) | +8,3% | confermato |
| **Luglio 2026** | — (mai disponibile) | **137,92 kWh (completo, 31/31 gg)** | — | **confermato (v10, solo inverter)** |
| **Agosto 2026** | — | **14,16 kWh (01-04/08, parziale)** | — | **da-aggiornare** |

**Verifica di qualità (v10):** per ogni giorno del nuovo import, la produzione è stata
ricalcolata per integrazione del contatore lifetime (max−min di `energia_tot_kwh` per
pannello, sommato sui 2 pannelli) e confrontata con il campo "Today's Energy" degli header
giornalieri EnverView. **Match esatto su tutti i 33 giorni conclusi** (04/07→03/08); il
04/08 mostra una piccola discrepanza attesa (1,53 vs 1,44 kWh) perché il giorno era ancora
in corso al momento dell'estrazione.

---

## 6. NOVITÀ v10 — Bolletta Sorgenia giugno 2026 (pendenza chiusa)

### Dati principali (NO PII)

| Voce | Valore |
|---|---:|
| Periodo | Giugno 2026 |
| Fornitore/offerta | Sorgenia — Next Energy Sunlight |
| Consumo totale fatturato | 344,5 kWh |
| F1 | 68,0 kWh |
| F2 | 102,9 kWh |
| F3 | 173,6 kWh |
| Totale bolletta (escl. canone TV) | 108,97 € |
| Canone TV | 9,00 € |
| Totale da pagare | 117,97 € |
| Prezzo medio quota consumi | 0,218578 €/kWh |
| Quota fissa | 8,19 €/mese |
| Quota potenza | 8,89 €/mese |
| Bonus Codice Amico | −1,25 € (una tantum) |

### Meccanismo tariffario (nuova nota metodologica)

Offerta a **prezzo variabile indicizzato PUN**, formula: `(PUN Index GME) * (1 + Perdite di
Rete) + Dispacciamento + Fee`. Valori di giugno 2026: PUN F1=0,1258 / F2=0,1517 / F3=0,1272
€/kWh; Perdite di Rete=10%; Dispacciamento=0,0199 €/kWh; Fee=0,0060 €/kWh.

**Sconto fedeltà confermato:** −0,325 €/mese sulla quota fissa, pari al **5%** — coerente
con lo scaglione mesi 1-12 del contratto (terzo mese di fornitura, sottoscritto 25/03/2026),
come previsto dalla nota metodologica introdotta in v9.

### Verifica incrociata Sonoff/eWeLink vs bolletta ufficiale

```
Sonoff (v7, corretto): 339,92 kWh
Bolletta ufficiale:    344,5 kWh
Delta: -4,58 kWh (-1,33%)
```

Buon accordo, in linea con gli altri scarti tipici del progetto (0,1%-1,3%). Il Sonoff
sottostima leggermente il prelievo ufficiale.

---

## 7. NOVITÀ v10 — Chiusura contabile DEFINITIVA di giugno 2026 (piscina)

Con la bolletta ufficiale disponibile, il prelievo dalla rete usato nella riconciliazione
passa dalla stima Sonoff (339,92 kWh) al **dato fatturato definitivo (344,5 kWh)** — la
fonte più autorevole secondo la gerarchia di strumentazione del progetto.

```
Autoconsumo FV giugno (Tapo)     = 124,482 - 6,74 = 117,74 kWh
Autoconsumo FV giugno (Inverter) = 134,81 - 6,74  = 128,07 kWh

Consumo reale casa (Tapo)     = 344,5 + 124,482 - 6,74 = 462,24 kWh
Consumo reale casa (Inverter) = 344,5 + 134,81 - 6,74  = 472,57 kWh
```

### Confronto storico delle versioni di calcolo (tabella `riconciliazione_mensile`)

| Versione | Fonte prelievo | Prelievo kWh | Fonte FV | Consumo reale kWh | Δ vs giu 2025 |
|---|---|---:|---|---:|---:|
| v4-v6 (superato) | Sonoff incompleto | 334,05 | Tapo | 451,94 | +2,71% |
| v5-v6 (superato) | Sonoff incompleto | 334,05 | Inverter | 462,27 | +5,06% |
| v7 (superato) | Sonoff corretto | 339,92 | Tapo | 457,66 | +4,01% |
| v7 (superato) | Sonoff corretto | 339,92 | Inverter | 467,99 | +6,36% |
| **v10 (definitivo)** | **Bolletta ufficiale** | **344,5** | **Tapo** | **462,24** | **+5,06%** |
| **v10 (definitivo)** | **Bolletta ufficiale** | **344,5** | **Inverter** | **472,57** | **+7,40%** |

```
Δ prelievo rete vs giugno 2025 (440 kWh) = -21,70%
```

**Lettura definitiva:** il consumo reale della casa a giugno cresce del **+5,1%** (fonte
Tapo) o **+7,4%** (fonte inverter diretto) rispetto a giugno 2025, nonostante la piscina
attiva tutto il mese — leggermente più della stima v7 (basata su Sonoff), perché la bolletta
ufficiale riporta un prelievo leggermente superiore al Sonoff. Il prelievo dalla rete resta
comunque **-21,7%** rispetto a giugno 2025. Il fotovoltaico continua ad assorbire la maggior
parte del nuovo carico della pompa piscina — conclusione qualitativa stabile in tutte le
versioni di calcolo, dalla prima stima v4 a questa chiusura definitiva.

**Nota metodologica:** le versioni superate (v4-v7) restano in DB con versioning esplicito,
non sovrascritte, per tracciabilità storica di come le stime sono migliorate mano a mano che
arrivavano dati più solidi (correzione buco Sonoff in v7, bolletta ufficiale in v10).

---

## 8. NOVITÀ v10 — Separazione effetto tariffa vs effetto kWh su giugno (risultato sorprendente)

Applicando il metodo standard del progetto (confronto a parità di kWh e a parità di
tariffa), con i **PUN specifici di giugno** (regola operativa #15):

### Effetto tariffa (a parità di kWh, 344,5)

| Componente | Plenitude baseline (gen-feb) | Sorgenia giugno | Delta |
|---|---:|---:|---:|
| Quota consumi (344,5 kWh) | 71,28 € (@0,206895) | 75,30 € (@0,218578) | **+4,02 €** |
| Quota fissa | 14,025 €/mese | 8,19 €/mese | **-5,84 €** |
| Quota potenza | 8,89 €/mese | 8,89 €/mese | 0,00 € |
| **Netto effetto tariffa** | | | **-1,81 €/mese** |

### Effetto kWh (a parità di tariffa Plenitude baseline)

```
Scenario baseline (475 kWh/mese @ tariffa Plenitude): 121,19 €
Giugno reale (344,5 kWh @ tariffa Plenitude baseline): 94,19 €
Effetto kWh (minor consumo): -27,00 €/mese
```

### Lettura — risultato sorprendente rispetto ai mesi precedenti

```
A maggio: beneficio cambio fornitore stimato ≈ 7-8 €/mese
A giugno: beneficio cambio fornitore ≈ -1,81 €/mese (quasi azzerato)
```

Il motivo: il **PUN di giugno era relativamente alto** (0,218578 €/kWh quota consumi contro
0,206895 € della tariffa fissa Plenitude), tanto da annullare quasi completamente il
vantaggio strutturale della quota fissa più bassa di Sorgenia. Questo **conferma
empiricamente** la regola operativa #15 del progetto: con un'offerta a prezzo indicizzato,
il beneficio tariffario **non è costante** e va sempre ricalcolato mese per mese sul PUN
specifico — non si può più assumere un beneficio fisso di "7-8 €/mese".

A giugno, quindi, il risparmio osservato rispetto alla baseline gen-feb è **quasi
interamente attribuibile al fotovoltaico/minor prelievo** (-27,00 €/mese), non al cambio
fornitore. Lo sconto fedeltà progressivo (destinato a salire dal 5% al 10% dal mese 13, cioè
da marzo 2027) potrebbe in futuro riportare il beneficio tariffario in territorio
consistentemente positivo, ma a giugno 2026 non è così.

**Nota di cautela:** il calcolo sopra è una stima semplificata (non modella la variazione
proporzionale di oneri generali/imposte, che scalano comunque in modo simile su entrambi gli
scenari); l'ordine di grandezza e soprattutto la direzione del risultato (beneficio
tariffario quasi nullo a giugno) sono comunque robusti.

---

## 9. Analisi standard giugno 2026 — riepilogo completo

### 1-2. Riepilogo e fasce
Vedi §6.

### 3. Confronto mese precedente (maggio 2026)

| | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Maggio 2026 | 41,7 | 102,6 | 144,3 | 288,6 |
| Giugno 2026 | 68,0 | 102,9 | 173,6 | 344,5 |
| Differenza | +26,3 | +0,3 | +29,3 | +55,9 |
| Variazione | **+63,1%** | +0,3% | +20,3% | **+19,4%** |

Lettura: F1 sale molto (+63%) — atteso, perché giugno ha più ore di luce/temperature più
alte che aumentano il prelievo diurno residuo nonostante il FV assorba una parte crescente.
F2 sostanzialmente stabile. F3 sale (+20%) — probabile maggior uso serale (ventilatori,
condizionamento leggero, piscina in orari F2/F3 residuali).

### 4. Confronto anno precedente (giugno 2025, pre-FV/pre-piscina)

| | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Giugno 2025 | 132 | 118 | 190 | 440 |
| Giugno 2026 | 68,0 | 102,9 | 173,6 | 344,5 |
| Differenza | -64,0 | -15,1 | -16,4 | -95,5 |
| Variazione | **-48,5%** | -12,8% | -8,6% | **-21,7%** |

Lettura: F1 crolla quasi della metà — il segnale FV resta forte anche col nuovo carico
piscina. F3 cala anch'essa (-8,6%), diversamente dal pattern di maggio (F3 stabile), un
segnale che richiederebbe più mesi per essere confermato come pattern stabile.

### 5. Confronto baseline gen-feb 2026

```
Baseline media: 475 kWh/mese
Giugno 2026: 344,5 kWh
Riduzione: -130,5 kWh/mese (-27,5%)
```

### 6-7. Costo medio e separazione tariffa/kWh
Vedi §8.

### 8. Canone TV escluso
Costo energetico giugno: **108,97 €** (esclusi i 9,00 € di canone TV).

### 9. Impatto piscina
Vedi §7 — chiusura definitiva: consumo reale casa +5,1%/+7,4% (Tapo/Inverter) vs giugno
2025, prelievo rete -21,7%.

### 10. Tabella finale e conclusione

| Indicatore | Valore |
|---|---:|
| Prelievo rete | 344,5 kWh (-21,7% vs giu 2025) |
| Consumo reale casa | 462,24-472,57 kWh (+5,1%/+7,4% vs giu 2025) |
| Costo energia (escl. TV) | 108,97 € |
| Effetto tariffa | ≈ -1,81 €/mese (quasi nullo, PUN alto a giugno) |
| Effetto kWh | ≈ -27,00 €/mese (causa principale del risparmio) |

**Conclusione netta:** a giugno 2026, nonostante la piscina attiva tutto il mese, il
prelievo dalla rete resta ben al di sotto (-21,7%) di giugno 2025. Il risparmio economico è
quasi interamente merito del fotovoltaico/autoconsumo; il cambio fornitore, che a maggio
contribuiva positivamente, a giugno ha un effetto pressoché nullo a causa del PUN elevato di
quel mese specifico.

---

## 10. Bollette (tabella completa aggiornata v10)

| Periodo | Fornitore | F1 | F2 | F3 | Tot kWh | Luce € | €/kWh | Fissa | Pot. | Stato |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| Mar-Apr 2025 | Plenitude | 217 | 269 | 314 | 800 | 249,84 | — | — | — | confermato |
| Mag-Giu 2025 | Plenitude | 227 | 243 | 331 | 801 | 236,93 | 0,200037 | 14,01 | 9,48 | confermato |
| Lug-Ago 2025 | Plenitude | 280 | 253 | 361 | 894 | 277,93 | 0,207383 | 14,005 | 9,48 | confermato |
| Set-Ott 2025 | Plenitude | 244 | 269 | 303 | 816 | 255,34 | 0,204216 | 14,005 | 9,48 | confermato |
| Nov-Dic 2025 | Plenitude | 276 | 276 | 400 | 952 | 286,09 | 0,201155 | 14,005 | 9,48 | confermato |
| Gen-Feb 2026 (baseline) | Plenitude | 297 | 306 | 347 | 950 | 290,35 | 0,206895 | 14,025 | 8,89 | confermato |
| Mar-Apr 2026 (FV installato) | Plenitude | 217 | 257 | 339 | 813 | 253,69 | 0,204600 | 14,025 | 8,89 | confermato |
| Maggio 2026 | Sorgenia | 41,7 | 102,6 | 144,3 | 288,6 | 88,69 | 0,201421 | 8,19 | 8,89 | confermato |
| **Giugno 2026** | **Sorgenia** | **68,0** | **102,9** | **173,6** | **344,5** | **108,97** | **0,218578** | **8,19** | **8,89** | **confermato (v10)** |

Dettaglio mensile per confronti anno-su-anno:

| Mese | F1 | F2 | F3 | Totale |
|---|---:|---:|---:|---:|
| Maggio 2025 | 95 | 125 | 141 | 361 |
| Giugno 2025 | 132 | 118 | 190 | 440 |
| Marzo 2026 | 151 | 162 | 187 | 500 |
| Aprile 2026 | 66 | 95 | 152 | 313 |
| Maggio 2026 | 41,7 | 102,6 | 144,3 | 288,6 |
| **Giugno 2026** | **68,0** | **102,9** | **173,6** | **344,5** |

### Letture Sonoff/eWeLink — aggregato mensile (invariato, gap aperto da 04/07)

| Mese | Prelievo kWh | Immissione kWh | Note |
|---|---:|---:|---|
| Dic 2025 | 154,95 | 0,00 | parziale (dal 23/12) |
| Gen 2026 | 501,20 | 0,00 | = bolletta 502 kWh ✓ |
| Feb 2026 | 447,50 | 0,00 | = bolletta 448 kWh ✓ |
| Mar 2026 | 504,34 | 0,00 | FV a fine mese |
| Apr 2026 | 314,17 | 0,00 | pannelli riposizionati |
| Mag 2026 | 291,82 | 13,46 | immissione leggibile dal 18/05 |
| Giu 2026 | 339,92 | 6,74 | corretto v7; ora anche verificato vs bolletta (-1,33%) |
| Lug 2026 (parziale) | 48,49 | 0,78 | 01-04/07, 83 letture — **non aggiornato oltre il 04/07 h10** |

---

## 11. Formule di riconciliazione (usare sempre)

```
Autoconsumo FV     = Produzione FV - Immissione
Consumo reale casa = Prelievo rete + Produzione FV - Immissione
```

Specificare sempre la fonte di Produzione FV usata (Tapo vs inverter diretto), dato lo
scarto sistematico dell'8,2-8,3%. **Per il prelievo rete, preferire sempre la bolletta
ufficiale quando disponibile** (gerarchia di attendibilità: bolletta > Sonoff/eWeLink,
v. §19 strumentazione) — nuova prassi consolidata in v10.

Verifica incrociata Sonoff vs bolletta:
- Gen 2026: Sonoff 501,2 / Bolletta 502 → −0,8 kWh (~0,2%) ✓
- Feb 2026: Sonoff 447,5 / Bolletta 448 → −0,5 kWh (~0,1%) ✓
- **Giugno 2026: Sonoff 339,92 / Bolletta 344,5 → −4,58 kWh (~1,33%) ✓ (nuova, v10)**

Verifica incrociata produzione FV: foglio "Anno" vs somma foglio "Mese" → match esatto.
Verifica incrociata energia integrata da potenza oraria vs contatore mensile/header
giornaliero → scarto <1,1% (v6); **confermata su tutti i 33 nuovi giorni v10 (match esatto)**.

---

## 12. Pendenze aperte (aggiornate v10)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| **Letture eWeLink/Sonoff aggiornate** | eWeLink export | **Alta** | Fermo al 04/07 h10 — gap di un mese sul lato prelievo/immissione rete (non FV) |
| Bolletta Sorgenia **luglio 2026** | Sorgenia | Media | Prossima in ordine cronologico |
| Produzione FV **agosto 2026** — completare | Inverter (+ Tapo se disponibile) | Media | Solo 4/31 giorni (parziale) |
| Export Tapo luglio/agosto | Tapo (mai arrivato per lug/ago) | Bassa | Non blocca alcuna analisi, solo utile per il confronto bias |
| Analisi economica sostituzione pannelli/inverter | — | Media | In sospeso da v7, non affrontata in questa sessione |
| **Decisione**: promuovere inverter a fonte primaria? | — | Media | In sospeso da v5, non decisa in v10 |
| Rigenerare token GitHub | — | Consigliata | Riutilizzato in molte sessioni consecutive |

**Azione a più alto impatto per la prossima sessione:** aggiornare `letture_quadro` con un
nuovo export eWeLink (gap di un mese) per chiudere anche il lato prelievo/immissione rete al
pari della produzione FV, ormai aggiornata a oggi.

---

## 13. Regole operative del progetto (invariate + 1 nuova)

1. **Canone TV sempre escluso** dai confronti energetici/economici.
2. Periodi bimestrali sempre **normalizzati su base mensile**.
3. Distinguere sempre: dato **confermato** (PDF/export verificato) / **stima** / **da aggiornare**.
4. Non attribuire risparmio al fornitore se i prezzi medi sono simili: verificare sempre se
   deriva da meno kWh prelevati.
5. **Segnale FV affidabile** = F1 cala, F3 stabile o in lieve aumento.
6. **No PII nel repo/handoff** (POD, intestatario, indirizzo, CF, IBAN, email, telefono, IP,
   codice cliente restano fuori).
7. Effetto piscina quantificato energeticamente, **chiuso definitivamente in v10** con
   prelievo da bolletta ufficiale: +5,1%/+7,4% consumo reale a seconda della fonte FV.
8. Quando si citano dati di produzione FV, specificare sempre la fonte (Tapo vs inverter
   diretto) dato lo scarto sistematico dell'8,2-8,3%.
9. Token GitHub sempre fornito come file `token.txt`, mai in chiaro; eliminarlo dal disco
   subito dopo il push quando possibile.
10. I dati orari per pannello (`letture_inverter_orarie`) usano deduplica su `(sn, ts)` —
    verificato robusto anche su un batch di 34 file con 2 giorni di overlap (v10).
11. Prima di considerare "chiuso" un mese in `letture_quadro`, verificare completezza oraria
    (giorni×24).
12. Quando un dato viene ricalcolato a seguito di una correzione a monte, conservare sia il
    valore precedente sia quello corretto con versioning esplicito.
13. Coefficienti tecnici riportati dall'utente da fonti non verificabili direttamente da
    Claude vanno salvati con nota di provenienza esplicita.
14. Per bollette biorarie (es. Plenitude), lo split F2/F3 non è presente nel PDF fiscale.
15. Per bollette a prezzo variabile indicizzato (es. Sorgenia), ricalcolare ogni confronto
    tariffario sul PUN del mese specifico — **confermato empiricamente cruciale in v10**: a
    maggio il beneficio tariffario era ~7-8€/mese, a giugno è sceso a ~-1,8€/mese per un PUN
    più alto. Non assumere mai un beneficio tariffario costante.
16. Quando si riceve un export "Rapporto giornaliero" EnverView, verificare l'intervallo di
    copertura oraria effettivo prima di fidarsi del campo riepilogativo "Today's Energy".
17. Per il contratto Sorgenia, tenere conto dello sconto fedeltà progressivo (5% mesi 1-12,
    10% mesi 13-24, 15% mesi 25-36, 20% dal mese 37).
18. Prima di elaborare un documento caricato, verificare se è un duplicato di uno già
    processato in sessione.
19. **(Nuova, v10)** Quando è disponibile sia il dato Sonoff/eWeLink sia la bolletta
    ufficiale per lo stesso mese, usare sempre la bolletta come prelievo di riferimento
    nelle riconciliazioni definitive (Sonoff resta utile per il dettaglio orario/timing, ma
    non come fonte primaria del totale mensile).

**Programma indicativo pompa piscina:**

| Fascia oraria | Stato pompa |
|---|---|
| 9/10 – 13 | Attiva |
| 13 – 16 | Pausa |
| 16 – 19 | Attiva |

---

## 14. Strumentazione (aggiornata v10)

| Strumento | Dato misurato | Attendibilità |
|---|---|---|
| **Bolletta Areti/distributore (tramite fornitore)** | **Prelievo ufficiale fatturato** | **Dato definitivo — fonte primaria per il prelievo mensile quando disponibile (v10)** |
| Sonoff POWCT / eWeLink | Prelievo orario dalla rete | Alta ma secondaria al dato bolletta; utile per il dettaglio orario/timing. Verificato vs bolletta giugno: -1,33% |
| Sonoff POWCT / eWeLink | Immissione oraria in rete | Attendibile da ~18/05/2026 |
| Tapo (foglio Anno/Mese) | Produzione FV mensile/giornaliera | Media — bias di sottostima ~8,2-8,3% confermato su 52 giorni |
| Inverter diretto (export mensile/giornaliero) | Produzione FV mensile/giornaliera | Alta — misura di primo livello |
| Inverter diretto (letture orarie) | Potenza istantanea, temperatura, tensioni per pannello | Alta — verificato per integrazione su 88 giorni totali (<1,1% scarto, match esatto sui 33 nuovi giorni v10) |
| Etichetta pannello (foto) | Dati elettrici/meccanici nominali | Alta — letta direttamente da Claude |
| Datasheet ufficiale Dahai (coefficienti termici) | Coefficienti di temperatura | Media — riportati dall'utente, non verificati direttamente |
| Bollette PDF originali (Plenitude/Sorgenia) | Totali, importi, prezzi ufficiali | Alta — fonte definitiva |
| Contratto Sorgenia (CGC + Modulo Adesione) | Formula tariffaria, sconto fedeltà | Alta — letto direttamente da Claude |

---

## 15. Conclusione attuale del progetto

La riduzione osservata resta **coerente e verificata**, ora con il mese piscina chiuso
definitivamente sul lato sia energetico sia economico.

```
Maggio 2025 → 361 kWh
Maggio 2026 → 288,6 kWh
Riduzione   → -72,4 kWh (-20,1%)
```

```
Giugno 2025 → 440 kWh
Giugno 2026 → 344,5 kWh
Riduzione   → -95,5 kWh (-21,7%)
```

Dato decisivo (fascia diurna, maggio):
```
F1 maggio 2025 = 95 kWh
F1 maggio 2026 = 41,7 kWh   → -56,1%
F3 sostanzialmente invariata (+2,3%)
```

Anche a giugno il segnale F1 resta forte (-48,5% vs giugno 2025), nonostante il nuovo carico
piscina.

**Chiusura piscina (giugno 2026), dato DEFINITIVO (v10):** con la bolletta ufficiale, il
consumo reale della casa cresce del **+5,1%** (fonte Tapo) o **+7,4%** (fonte inverter
diretto) rispetto a giugno 2025, mentre il prelievo dalla rete cala comunque del **-21,7%**.
Il fotovoltaico assorbe la maggior parte del nuovo carico della pompa piscina.

**Scoperta v10 — il beneficio tariffario non è stabile:** a differenza di maggio (~7-8
€/mese di risparmio dal cambio fornitore), a giugno l'effetto tariffa è quasi nullo (≈-1,8
€/mese) a causa di un PUN elevato quel mese. Il risparmio di giugno è quasi interamente
merito del fotovoltaico (~-27 €/mese di effetto kWh). Questo conferma che, con un'offerta a
prezzo indicizzato, il confronto va sempre rifatto mese per mese.

**Frontiera aperta:** il gap sul lato Sonoff/eWeLink (fermo al 04/07) va colmato per
allinearlo alla copertura ormai aggiornata della produzione FV. Resta anche da decidere se
promuovere l'inverter diretto a fonte primaria, e da completare l'analisi economica di
un'eventuale sostituzione pannelli/inverter.

---
*Fine handoff v10. Prossima azione consigliata: (1) aggiornare `letture_quadro` con un nuovo
export eWeLink per colmare il gap 04/07→oggi; (2) ingest bolletta Sorgenia luglio quando
disponibile; (3) completare produzione FV agosto; (4) rigenerare il token GitHub; (5)
opzionale — analisi economica payback sostituzione pannelli/inverter.*
