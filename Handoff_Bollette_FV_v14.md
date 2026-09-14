# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 14 — aggiornata al 14/09/2026

**Data:** 2026-09-14
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v14:**
1. **Agosto 2026 chiuso per prelievo/immissione** (era preliminare 01-26/08 in v13, ora completo 01-31/08 via nuovo export eWeLink storico 09/03→14/09).
2. **Settembre 2026 avviato**: prelievo/immissione 01-14/09 (parziale, fino alle 13:00), produzione FV Tapo 08-14/09 (parziale, giorno 14 fino alle 13:52).
3. **Scheda tecnica ufficiale Envertech EVT800** acquisita e verificata (PDF datasheet fotografato dall'utente) — conferma tutti i parametri già a verbale e ne aggiunge di nuovi (vedi §5).
4. **Buco residuo identificato**: produzione FV (Tapo) mancante per 28/08–07/09 (11 giorni) — prelievo/immissione per lo stesso periodo è invece completo.

---

## 1. Obiettivo del progetto

Ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo.

**Domanda guida:**
```
Quanto del risparmio deriva dal fotovoltaico/autoconsumo
e quanto dal cambio fornitore?
```

Distinzione metodologica sempre obbligatoria: effetto kWh, effetto tariffa, effetto piscina, effetto climatizzazione, canone TV sempre escluso. (invariato, vedi v13 §1)

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v14

| Tabella | Righe | Copertura |
|---|---:|---|
| `bollette` | 10 | invariato da v13 (Mar 2025 → Luglio 2026) |
| `produzione_fv` | 6 | invariato da v13 (Mar 2026 → Ago 2026 parziale) |
| `produzione_giornaliera` | **154** (era 147) | 23/03 → 27/08/2026 **+ 08/09 → 14/09/2026** (buco 28/08–07/09, vedi §6) |
| `letture_inverter_orarie` | 94.662 | invariato — nessun nuovo export inverter diretto in questa sessione |
| `letture_quadro` | **6.360** (era 5.930) | 23/12/2025 → **14/09/2026 13:00** |
| `verifica_bias_tapo_inverter` | 88 | invariato — serve nuovo export inverter diretto per estenderla |
| `riconciliazione_mensile` | 9 | invariato — agosto resta "preliminare" nella tabella (vedi §7) |
| `cronologia_eventi` | 16 | invariato |
| `meteo_giornaliero` / `meteo_orario` | 65 / 1.560 | invariato — fermi al 04/08 |
| `fornitori_storico` | 4 | invariato |
| `pannelli_fv_specifiche` | 2 | invariato |

---

## 4. NOVITÀ v14 — Agosto 2026 chiuso per prelievo/immissione

Il nuovo export eWeLink (storico continuo 09/03→14/09/2026, 4.536 righe) ha colmato il buco 27/08 15:00 → 31/08 rimasto in v13, e prosegue ininterrotto fino al 14/09.

| Mese | Prelievo | Immissione | Righe | Stato (v13 → v14) |
|---|---:|---:|---:|---|
| **Agosto 2026** | **566,10 kWh** | **7,39 kWh** | 744 (31 gg × 24h) | preliminare (01-26/08, ~450 kWh proiettato) → **completo** |
| **Settembre 2026** | 197,07 kWh | 2,66 kWh | 326 (14 gg, parziale fino alle 13:00) | nuovo |

```
Media prelievo/giorno: giugno 11,48 → luglio 20,04 → agosto 18,26 kWh/g
```

Il dato definitivo di agosto (18,26 kWh/g) conferma la lettura preliminare di v13 (17,32 kWh/g su 01-26/08): l'attenuazione rispetto al picco di luglio (20,04 kWh/g) è reale ma modesta, non un rientro alla normalità di giugno. Va ricordato che il dato include ancora il periodo vacanza 17-22/08 (-80,5% di prelievo), quindi il "vero" consumo della casa occupata in agosto è probabilmente più alto di 18,26 kWh/g medio.

**Nota metodologica:** il prelievo eWeLink resta sistematicamente qualche punto percentuale sotto la bolletta ufficiale (giugno: bolletta 344,5 vs Sonoff ~340; luglio: bolletta 621,1 vs Sonoff 597,45, +3,96%). Applicando lo stesso scarto ad agosto (566,10 kWh Sonoff), la bolletta definitiva sarà verosimilmente intorno a **585-590 kWh** — da verificare quando Sorgenia la emette (attesa metà settembre).

---

## 5. NOVITÀ v14 — Scheda tecnica ufficiale Envertech EVT800

Acquisito il datasheet ufficiale del microinverter (fotografato dall'utente, non solo l'etichetta come per i pannelli Dahai in v7). Conferma i parametri già a verbale e ne aggiunge di nuovi:

| Parametro | Valore |
|---|---:|
| Potenza di ingresso consigliata (STC) | (180W-550W+)×2 |
| Massimo ingresso CC | 60V |
| Corrente massima cortocircuito ingresso | 25A |
| Intervallo operativo | 16V-60V |
| Corrente ingresso massima continua | 14A×2 |
| **Intervallo di tensione MPPT** | **22V-50V** |
| Tensione uscita nominale | 220V-230V |
| Corrente uscita max continua | 3,63A |
| Potenza max continua uscita | 800W |
| Fattore di potenza nominale | ±0,90 |
| **THD** | **<3%** |
| Efficienza ponderata Euro | 96,8% |
| Efficienza MPPT | 99,9% |
| Consumo energetico notturno | <100mW |
| Comunicazione | PLCC / Wi-Fi |
| IP | 67, Classe I |
| Temperatura operativa | -40°C a +65°C |
| Categoria sovratensione | OVC III (CA), OVC II (PV) |
| **Dimensioni** | **264×194×35,5mm** |
| **Peso** | **3,7 kg** |
| Garanzia | 15 anni (20 opzionali) |

**Rilevanza per il progetto:** il limite di 14A continui per canale MPPT (menzionato in v13 §6 come vincolo per l'eventuale sostituzione con pannelli bifacciali Megasol, margine attuale 3,2%) è ora confermato dalla fonte ufficiale, non solo da nota interna. Nessun altro dato cambia le analisi già consolidate — è una conferma di fonte, non una revisione.

Non è stata creata una tabella dedicata nel DB (non esiste un analogo di `pannelli_fv_specifiche` per l'inverter); il dato resta documentato qui. Da valutare se serva in una sessione futura, specialmente in vista dell'eventuale sostituzione pannelli.

---

## 6. NOVITÀ v14 — Buco residuo: produzione FV 28/08–07/09

A differenza di prelievo/immissione (ora continui fino al 14/09), la produzione FV (fonte Tapo) ha un buco di **11 giorni** tra l'ultimo dato disponibile in v13 (27/08) e il primo di questa sessione (08/09). Nessun file caricato in questa sessione copre quel periodo.

**Impatto:** non è possibile calcolare l'autoconsumo/consumo reale per la seconda metà di agosto né per la prima settimana di settembre finché non arriva un nuovo export Tapo che copra quell'intervallo. Il prelievo dalla rete per lo stesso periodo è invece già disponibile e affidabile (§4).

**Dato settembre disponibile (solo produzione, parziale):**

| Data | kWh (Tapo) | Copertura | Note |
|---|---:|---:|---|
| 08/09 | 3,74 | 24h | — |
| 09/09 | 3,38 | 24h | — |
| **10/09** | **0,69** | 24h | **anomalo — crollo rispetto alla media (~3,7), da verificare (nuvolosità? buco dati Tapo?)** |
| 11/09 | 3,63 | 24h | — |
| 12/09 | 3,81 | 24h | — |
| 13/09 | 3,79 | 24h | — |
| 14/09 | 2,06 | 14h (parziale) | export troncato alle 13:52 |

Il giorno 10/09 rompe nettamente il pattern regolare degli altri giorni (tutti tra 3,4 e 3,8 kWh). Non è stato possibile verificarlo con la seconda fonte (inverter diretto), che non è stata caricata in questa sessione. Marcato per verifica futura, analogamente all'anomalia 09/05/2026 già nota (mai risolta, v5 §7).

---

## 7. Impatto sulla riconciliazione di agosto (invariato nel merito, aggiornabile in dettaglio)

La tabella `riconciliazione_mensile` non è stata toccata in questa sessione: la riga di agosto resta quella preliminare di v13 (01-26/08, dato Tapo 98,80 kWh produzione). Con il nuovo dato di prelievo/immissione completo (§4) ma senza produzione FV completa (§6), un ricalcolo integrale del mese non è ancora possibile in modo affidabile — verrà fatto quando si colmerà il buco Tapo 28/08-07/09 e si completerà la produzione di fine agosto.

Quello che si può già dire con il solo dato di prelievo (fonte definitiva, non stimata):
```
Prelievo agosto 2026 (completo) = 566,10 kWh
vs prelievo luglio 2026 (bolletta) = 621,10 kWh   → -8,9%
vs prelievo giugno 2026 (bolletta) = 344,50 kWh   → +64,3%
```
Agosto resta ben sopra giugno nonostante l'attenuazione rispetto al picco di luglio — coerente con la lettura già proposta in v13 (piscina + climatizzazione ancora attive per buona parte del mese, solo parzialmente compensate dai 6 giorni di vacanza).

---

## 8. Dati invariati da v13

Bollette, confronti principali (maggio 2025 vs 2026, cambio fornitore, chiusura definitiva di luglio), cronologia tecnica, stato pannelli bifacciali Megasol (non ancora installati) — tutto invariato. Vedi v13 per il dettaglio completo, non ripetuto qui.

---

## 9. Pendenze aperte (aggiornate v14)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Produzione FV (Tapo) **28/08–07/09** | Tapo/EnverView, export foglio Mese | **Alta** | Unico buco rimasto nella serie continua; blocca la chiusura di agosto |
| Produzione FV **inverter diretto**, da fine agosto in poi | Export EnverView giornaliero per pannello | Media | Ferma al 25/08 — impedisce di aggiornare il bias Tapo/inverter oltre quella data |
| Verifica anomalia **10/09/2026** (produzione Tapo -81% vs media) | Export inverter diretto o conferma meteo | Media | Analoga all'anomalia 09/05 mai risolta |
| Bolletta Sorgenia **agosto 2026** | Sorgenia (attesa metà settembre) | **Alta** | Chiusura definitiva di agosto, incluso conto economico del periodo vacanza |
| Dati meteo Settebagni oltre 04/08 | Open-Meteo (via browser utente) | Media | Utile per correlazione temperatura/prelievo su agosto-settembre |
| Analisi economica sostituzione pannelli/inverter | — | Bassa (lavoro non eseguito) | invariato da v13 |
| Rigenerare token GitHub | — | Consigliata | Riutilizzato in molte sessioni consecutive (invariato da v12/v13) |

**Azione a più alto impatto per la prossima sessione:** colmare il buco Tapo 28/08-07/09 → sblocca la chiusura energetica completa di agosto (prelievo già pronto, manca solo la produzione).

---

## 10. Regole operative del progetto (invariate)

Tutte le regole precedenti (1-21+, vedi v12/v13) restano valide e sono state applicate senza eccezioni in questa sessione.

---

## 11. Conclusione attuale del progetto

Agosto 2026 è ora **chiuso dal lato prelievo/immissione** (566,10 kWh / 7,39 kWh, mese intero) ma resta **aperto dal lato produzione FV** per un buco di 11 giorni (28/08-07/09) non coperto da alcun file disponibile in questa sessione. Il prelievo di agosto si attenua rispetto al picco di luglio (18,26 kWh/g medi contro 20,04) ma resta ben sopra giugno (+64,3%), coerente con piscina e climatizzazione ancora parzialmente attive, solo in parte compensate dai 6 giorni di vacanza già identificati in v13.

Settembre è avviato con i primi 14 giorni di prelievo/immissione e una settimana di produzione FV (08-14/09), con un giorno anomalo (10/09, produzione crollata dell'81% rispetto alla media) da verificare quando sarà disponibile il dato inverter diretto.

La scheda tecnica ufficiale dell'inverter EVT800 è ora acquisita e conferma tutti i parametri già noti, incluso il vincolo di corrente (14A continui per canale) già identificato come punto di attenzione per l'eventuale sostituzione pannelli.

---
*Fine handoff v14. Prossima azione consigliata: (1) colmare il buco produzione Tapo 28/08-07/09; (2) aggiornare l'export inverter diretto per estendere il bias Tapo/inverter oltre il 25/08; (3) ingest bolletta Sorgenia agosto appena emessa; (4) verificare l'anomalia del 10/09.*
