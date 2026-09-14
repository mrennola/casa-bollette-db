# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 15 — aggiornata al 14/09/2026

**Data:** 2026-09-14
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v15:**
1. **17 export EnverView "Daily Report"** (un file per giorno, entrambi i pannelli, letture ogni ~2 minuti) caricati e ingeriti, copertura **28/08 → 13/09/2026**.
2. **Buco produzione FV 28/08-07/09 colmato lato inverter diretto** (rimane un buco molto più piccolo: solo 26-27/08 per l'inverter, solo 28-31/08 per Tapo).
3. **Anomalia 10/09/2026 confermata da due fonti indipendenti**: sia Tapo (0,69 kWh) sia inverter diretto (0,77 kWh) mostrano lo stesso crollo — non è un artefatto di misura Tapo, il calo di produzione è reale.
4. `letture_inverter_orarie` estesa di **12.924 righe** (era 94.662 → **107.586**), copertura ora continua 09/05 → 13/09/2026.
5. `verifica_bias_tapo_inverter` estesa a **94 righe** (+6, per i giorni 08-13/09 con doppia fonte).

---

## 1. Obiettivo del progetto

Ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo. (invariato, vedi v13-v14 §1)

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v15

| Tabella | Righe | Copertura |
|---|---:|---|
| `bollette` | 10 | invariato |
| `produzione_fv` | 6 | invariato in numero righe; **riga agosto aggiornata** (vedi §5) |
| `produzione_giornaliera` | **165** (era 154) | 23/03 → 27/08 **+ 28/08 → 14/09/2026, continua** (nessun buco residuo su nessuna delle due fonti oltre a quelli indicati in §5) |
| `letture_inverter_orarie` | **107.586** (era 94.662) | 09/05 → **13/09/2026**, continua |
| `letture_quadro` | 6.360 | invariato da v14 (23/12/2025 → 14/09/2026 13:00) |
| `verifica_bias_tapo_inverter` | **94** (era 88) | 09/05 → **13/09/2026** |
| `riconciliazione_mensile` | 9 | invariato — agosto resta senza nuova versione (vedi §6) |
| `cronologia_eventi` | 16 | invariato |
| `meteo_giornaliero` / `meteo_orario` | 65 / 1.560 | invariato — fermi al 04/08 |
| `fornitori_storico` | 4 | invariato |
| `pannelli_fv_specifiche` | 2 | invariato |

---

## 4. NOVITÀ v15 — Produzione FV: buco quasi chiuso

### Dati inseriti (inverter diretto, "Today's Energy" per giorno = somma dei due pannelli, verificato per delta cumulativo)

| Data | kWh (inverter) | Copertura |
|---|---:|---|
| 28/08 | 3,31 | nuovo |
| 29/08 | 3,19 | nuovo |
| 30/08 | 4,04 | nuovo |
| 31/08 | 4,21 | nuovo |
| 01/09 | 4,08 | nuovo |
| 02/09 | 4,13 | nuovo |
| 03/09 | 4,10 | nuovo |
| 04/09 | 4,05 | nuovo |
| 05/09 | 4,15 | nuovo |
| 06/09 | 4,04 | nuovo |
| 07/09 | 3,90 | nuovo |
| 08/09 | 4,98→**3,98** | integra riga Tapo già presente |
| 09/09 | 3,62 | integra riga Tapo già presente |
| **10/09** | **0,77** | integra — vedi anomalia sotto |
| 11/09 | 3,91 | integra riga Tapo già presente |
| 12/09 | 4,07 | integra riga Tapo già presente |
| 13/09 | 4,08 | integra riga Tapo già presente |

**Buchi rimasti (invariati nella sostanza, solo ristretti):**
- Tapo: manca ancora **28/08-31/08** (4 giorni) — nessun file Tapo caricato in questa sessione copre quel periodo.
- Inverter diretto: manca ancora **26/08-27/08** (2 giorni) — i 17 file partono dal 28/08.

Quindi oggi la serie **28/08 → 13/09** ha sempre almeno una fonte disponibile (inverter, e dal 08/09 anche Tapo); il vero buco residuo per un confronto a doppia fonte resta circoscritto a 26-27/08 (solo Tapo disponibile) e 28-31/08 (solo inverter disponibile).

### Verifica bias Tapo/inverter sui 6 giorni con doppia fonte (08-13/09)

| Data | Tapo | Inverter | Delta % |
|---|---:|---:|---:|
| 08/09 | 3,742 | 3,98 | +6,37% |
| 09/09 | 3,383 | 3,62 | +7,00% |
| 10/09 | 0,686 | 0,77 | +12,16% (denominatore piccolo, delta assoluto minimo) |
| 11/09 | 3,627 | 3,91 | +7,81% |
| 12/09 | 3,809 | 4,07 | +6,85% |
| 13/09 | 3,793 | 4,08 | +7,56% |

Bias medio (esclus 10/09 per denominatore anomalo): **~7,1%**, coerente col bias sistematico +7,8/+8,3% già documentato da maggio ad agosto. Nessuna sorpresa: il pattern regge anche a settembre.

### Anomalia 10/09/2026 — ora confermata, non più un sospetto isolato

```
Tapo:     0,69 kWh
Inverter: 0,77 kWh
Media altri giorni: ~3,7-4,1 kWh
```

A differenza dell'anomalia mai risolta del 09/05/2026 (dove le due fonti erano in disaccordo, -75%, segno opposto al pattern), qui **le due fonti indipendenti concordano**: il calo di produzione del 10/09 è reale, non un artefatto dell'app Tapo. Causa non nota — non ci sono dati meteo disponibili per quella data (la tabella `meteo_orario` è ferma al 04/08) per confermare nuvolosità come spiegazione più probabile. Resta `da verificare` ma con una diagnosi diversa e più solida rispetto al caso di maggio.

---

## 5. NOVITÀ v15 — Riga mensile agosto in `produzione_fv`

Aggiornata (non sostituita) la riga di agosto:

| | Tapo | Inverter diretto |
|---|---:|---:|
| Copertura | 27/31 giorni (01-27/08, il 27 parziale) | **29/31 giorni** (01-25/08 + 28-31/08) — era 25/31 in v13 |
| Totale | 101,25 kWh | **117,36 kWh** (era 102,61 kWh in v13) |
| Mancano | 28-31/08 (4 gg) | **solo 26-27/08 (2 gg)** |

Stato: resta `da-aggiornare` (nessuna delle due fonti copre il mese intero), ma il divario si è ristretto in modo significativo lato inverter.

---

## 6. Impatto sulla riconciliazione di agosto — perché non ho aggiornato `riconciliazione_mensile`

Il prelievo/immissione di agosto è completo dal mese scorso (566,10 kWh / 7,39 kWh, 31/31 giorni, v14). La produzione FV via inverter ora copre 29/31 giorni. Ho deciso di **non inserire una nuova riga di riconciliazione** questa sessione, perché mescolare un prelievo a copertura piena (31 gg) con una produzione a copertura parziale (29 gg) produrrebbe un consumo reale calcolato **sottostimato** di circa 2 giorni di autoconsumo — un errore silenzioso peggiore di lasciare il dato aperto.

La chiusura pulita di agosto resta quindi condizionata a uno di questi due eventi:
1. arrivo dei 2 giorni mancanti (26-27/08, export EnverView) per chiudere l'inverter a 31/31 — a quel punto la riconciliazione userebbe la fonte inverter come primaria (coerente con la pratica già seguita per giugno-luglio);
2. oppure, in alternativa più rapida, arrivo dei 4 giorni Tapo mancanti (28-31/08).

---

## 7. Dati invariati da v14

Bollette, confronti principali, cronologia tecnica, stato pannelli bifacciali Megasol, scheda tecnica ufficiale EVT800 — tutto invariato. Vedi v14 per il dettaglio completo.

---

## 8. Pendenze aperte (aggiornate v15)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Produzione FV **inverter diretto 26-27/08** | Export EnverView Daily Report | **Alta** | Solo 2 giorni: chiuderebbe agosto lato inverter (31/31) e sbloccherebbe la riconciliazione definitiva |
| Produzione FV **Tapo 28-31/08** | Tapo, foglio Mese | Media | Alternativa/complemento al punto sopra |
| Bolletta Sorgenia **agosto 2026** | Sorgenia (attesa metà settembre) | **Alta** | invariato da v14 |
| Dati meteo oltre il 04/08 | Open-Meteo (via browser utente) | Media | Servirebbe anche per spiegare l'anomalia del 10/09 (nuvolosità?) |
| Verifica causa anomalia **10/09/2026** | Meteo o log installatore | Bassa-Media | Ora confermata su due fonti, ma causa ancora ignota |
| Analisi economica sostituzione pannelli/inverter | — | Bassa | invariato |
| Rigenerare token GitHub | — | Consigliata | invariato, riutilizzato in molte sessioni |

**Azione a più alto impatto per la prossima sessione:** i soli 2 giorni mancanti (26-27/08, inverter) chiudono il mese di agosto quasi per intero — è il tassello più piccolo rimasto per il quadro più grande.

---

## 9. Regole operative del progetto (invariate)

Tutte le regole precedenti restano valide e sono state applicate senza eccezioni.

---

## 10. Conclusione attuale del progetto

Con i 17 export EnverView di questa sessione, il buco di produzione FV che bloccava la chiusura di agosto (28/08-07/09, 11 giorni completamente scoperti) è **praticamente richiuso**: oggi manca solo l'inverter di 2 giorni (26-27/08) o, in alternativa, il Tapo di 4 giorni (28-31/08). Non è stata forzata una riconciliazione con dati di copertura disomogenea, per non introdurre una sottostima silenziosa del consumo reale.

L'anomalia del 10/09/2026 (produzione quasi azzerata) è stata **confermata come reale** da due misure indipendenti che concordano tra loro — a differenza del caso isolato del 09/05, qui non c'è disaccordo tra le fonti, solo l'assenza di un dato meteo che ne spieghi la causa.

---
*Fine handoff v15. Prossima azione consigliata: (1) recuperare gli export EnverView del 26-27/08 per chiudere agosto lato inverter; (2) ingest bolletta Sorgenia agosto appena emessa; (3) se possibile, recuperare dati meteo di settembre per verificare la causa dell'anomalia del 10/09.*
