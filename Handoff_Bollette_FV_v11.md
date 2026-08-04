# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 11 — aggiornata al 04/08/2026 (con causa luglio chiarita)

**Data:** 2026-08-04
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v11 (continuazione della sessione v10, stesso giorno):**
1. **Gap eWeLink colmato**: nuovo export CSV eWeLink (27/01→04/08/2026) caricato in
   `letture_quadro`. Luglio 2026 ora **completo al 100%** (744/744 letture orarie).
2. **Dati Tapo di luglio e agosto arrivati** (mai disponibili prima, sempre "da esportare"):
   file `Consumo_di_Energia.xls` con fogli Anno/Mese/Giorno. Bias Tapo-vs-inverter
   riconfermato stabilissimo: **8,20% medio su 87 giorni** (era 8,20% su 52 — invariato).
3. **Prelievo di luglio molto più alto (597,45 kWh) — CAUSA CHIARITA**: l'utente ha
   confermato uso intensivo di due impianti di climatizzazione durante l'ondata di caldo di
   luglio (split 12000 BTU salone + multisplit 3 unità camere). Verifica di plausibilità
   effettuata (§7) — ordine di grandezza coerente. Non è un'anomalia tecnica di impianto o
   di misura, ma un fattore comportamentale/stagionale nuovo da tracciare nel progetto.
4. Chiusura **preliminare** di luglio in `riconciliazione_mensile` (in attesa della bolletta
   Sorgenia di luglio per la chiusura definitiva).

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

---

## 2. Nota privacy (regola di progetto, invariata e vincolante)

Dati identificativi (intestatario, indirizzo, POD, CF, IBAN, email, telefono, IP, codice
cliente) **mai memorizzati né inclusi** in repo/handoff/DB. Rispettata anche in questa
sessione (bolletta Sorgenia giugno con PII in chiaro; export eWeLink/Tapo senza PII).

Dati tecnici dell'utenza (riportabili):

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
- **File principale:** `reqa_bollette.db` (SQLite)
- **Ultimo commit pushato:** `d134e61` (handoff v10) — **questa sessione (v11) non ancora
  pushata**, in attesa di token

### Accesso futuro
```bash
git clone --depth 1 https://github.com/mrennola/casa-bollette-db.git
cd casa-bollette-db
```
Lettura libera. Scrittura: token `claude-casa-bollette` come file `token.txt` (mai in
chiaro). **Nota sicurezza (invariata da v6):** consigliata rigenerazione.

### Struttura del database (aggiornata v11)

| Tabella | Righe | Copertura |
|---|---:|---|
| `bollette` | 9 | Mar 2025 → **Giugno 2026** |
| `produzione_fv` | 6 | Mar 2026 → **Ago 2026** (luglio ora confermato entrambe le fonti) |
| `produzione_giornaliera` | 124 | 23/03 → 04/08/2026 (**Tapo e inverter ora completi in parallelo per lug-ago**) |
| `letture_inverter_orarie` | 76.928 | 09/05 (parziale) → 04/08/2026 11:08 |
| `letture_quadro` | **5.374** (era 4.629) | 23/12/2025 → **04/08/2026 11:00** |
| `verifica_bias_tapo_inverter` | **88** (era 53) | 09/05 → **04/08/2026** |
| `riconciliazione_mensile` | **8** (era 6) | Giugno (definitivo v10) + **Luglio (preliminare, nuovo)** |
| `cronologia_eventi` | 10 | invariata |
| `fornitori_storico` | 3 | invariata |
| `pannelli_fv_specifiche` | 1 | invariata |

**Nuova fonte dati (v11):** `Consumo_di_Energia.xls` — export Tapo/EnverView con fogli
Anno (mensile), Mese (giornaliero), Giorno (5 min, finestra breve). Stesso formato del
consueto export Tapo già usato nelle versioni precedenti.

**Fonte non integrata (v11):** `Potenza.xls` (Sonoff, potenza istantanea W, finestra breve
5min/1settimana) — ridondante con `letture_quadro` (già orario), non caricata nel DB.
Disponibile per analisi puntuali future se richiesto.

---

## 4. Cronologia tecnica rilevante (invariata da v10, salvo ultima riga)

| Data | Evento |
|---|---|
| Mag-Giu 2025 | Periodo pre-FV anno precedente |
| Gen-Feb 2026 | Periodo benchmark pre-FV |
| 23/24 mar 2026 | Installazione iniziale FV plug & play |
| 04 apr 2026 | Riposizionamento pannelli |
| 25 mar 2026 | Sottoscrizione contratto Sorgenia |
| 30 apr 2026 | Fine contratto Plenitude |
| 01 mag 2026 | Decorrenza Sorgenia |
| 9/17 mag 2026 | Cambio inverter → EVT800 |
| 18 mag 2026 | Sonoff legge bene immissione |
| 30 mag 2026 | Installazione piscina |
| Da giu 2026 | Pompa piscina attiva tutto il mese |
| 16/07/2026 | Emissione bolletta Sorgenia giugno |
| **04 ago 2026** | **Sessione v10+v11: import inverter, bolletta giugno, gap eWeLink colmato, dati Tapo lug/ago** |

---

## 5. Fotovoltaico — produzione mensile (tabella definitiva v11)

| Mese | Tapo (foglio Anno) | Inverter diretto | Delta | Stato |
|---|---:|---:|---:|---|
| Marzo 2026 | 23,836 kWh | — | — | confermato (solo Tapo) |
| Aprile 2026 | 101,507 kWh | — | — | confermato (solo Tapo) |
| Maggio 2026 | 115,205 kWh | 92,45 kWh (09-31/05) | +8,0% | confermato |
| Giugno 2026 | 124,482 kWh | 134,81 kWh | +8,3% | confermato |
| **Luglio 2026** | **127,777 kWh** | **137,92 kWh** | **+7,9%** | **confermato entrambe le fonti (v11)** |
| **Agosto 2026** (parziale) | **13,158 kWh** | **14,16 kWh** | +7,6% | da-aggiornare (4/31 gg) |

### Bias Tapo vs Inverter — riconfermato su 87 giorni

| | Valore |
|---|---:|
| Giorni confrontati (esclusa anomalia 09/05) | 87 |
| Media delta giornaliero | **8,196%** |
| Range | 5,95% – 14,54% |

Il bias resta stabilissimo anche estendendo il confronto a luglio — rafforza ulteriormente
l'ipotesi di un fattore moltiplicativo sistematico (non stagionale, non casuale).

---

## 6. NOVITÀ v11 — Gap eWeLink colmato

Import di `History_2026_01_27-2026_08_04_UTC.csv` (formato standard eWeLink: date, time
range, consumption/KWh, supply/KWh). 745 righe nuove inserite (04/07 11:00 → 04/08 11:00),
3.791 duplicate scartate correttamente (dedup su PK `ts`).

**Luglio 2026 ora completo:** 744/744 letture orarie (31 giorni × 24h esatto), nessun buco.

```
Prelievo rete luglio 2026 = 597,45 kWh
Immissione luglio 2026    = 3,31 kWh
```

---

## 7. Prelievo luglio molto alto — CAUSA CHIARITA DALL'UTENTE

Il dato iniziale era sorprendente:

| Mese | Prelievo rete |
|---|---:|
| Maggio 2026 | 291,82 kWh |
| Giugno 2026 | 344,5 kWh (bolletta) |
| **Luglio 2026** | **597,45 kWh** |
| *Luglio 2025 (pre-FV, pre-piscina)* | *468 kWh* |

Il prelievo di luglio 2026 non solo raddoppia rispetto a giugno, ma **supera anche il dato
dell'anno precedente pre-fotovoltaico** (+27,7%) — un risultato che rovescia il pattern
osservato in tutti i mesi precedenti (dove il FV riduceva sempre il prelievo vs anno
precedente).

**Verifica tecnica effettuata:** non è un artefatto di un singolo giorno anomalo — l'aumento
è distribuito su tutto il mese (giorni di picco: 16/07 = 30,94 kWh, 31/07 = 29,82 kWh, contro
un picco di giugno di 25,36 kWh il 27/06). Nessuna singola ora anomala rilevata (max singola
lettura oraria 3,7 kWh, plausibile). **Esclusa quindi una causa tecnica/di misura.**

### Causa confermata dall'utente (04/08/2026)

Uso intensivo di due impianti di climatizzazione durante l'ondata di caldo di luglio:
- **Split monozona 12.000 BTU** per il salone — il più usato
- **Multisplit** (un motore esterno, 3 unità interne) per le 3 camere da letto

### Verifica di plausibilità (stima, non misura)

```
Extra prelievo luglio vs giugno = 597,45 - 339,92 = 257,53 kWh su 31 giorni
                                 = 8,31 kWh/giorno

Potenza elettrica stimata (salone 12000 BTU inverter ~1,15 kW + multisplit ~1,8 kW) ≈ 3,0 kW
Ore/giorno equivalenti a pieno carico combinato per spiegare il delta ≈ 2,8 h/giorno
```

**Lettura:** ~2,8 ore/giorno a pieno carico combinato (o più ore a carico parziale, tipico
per unità inverter che modulano) è pienamente plausibile per un luglio caldo a Roma con uso
pomeridiano/serale del condizionamento. **Ordine di grandezza coerente**, causa accettata
come spiegazione primaria.

**Nota metodologica:** questa non è una misura diretta (non c'è uno smart plug dedicato sui
condizionatori), ma una verifica di plausibilità aritmetica. La causa resta "riportata
dall'utente, verificata per ordine di grandezza" — non "misurata direttamente" — coerente
con lo standard di rigore del progetto sulla provenienza dei dati.

**Nuovo evento in `cronologia_eventi`:** aggiunta voce categoria `climatizzazione` per
tracciare questo fattore stagionale, utile per confronti futuri (es. agosto, o luglio 2027).

---

## 8. NOVITÀ v11 — Chiusura PRELIMINARE di luglio 2026 (in attesa di bolletta)

Non essendo ancora disponibile la bolletta Sorgenia di luglio, il prelievo usato è quello
Sonoff/eWeLink (mese ora completo, alta affidabilità per il totale anche se non "definitivo"
come una bolletta).

```
Autoconsumo FV luglio (Tapo)     = 127,777 - 3,31 = 124,47 kWh
Autoconsumo FV luglio (Inverter) = 137,92 - 3,31  = 134,61 kWh

Consumo reale casa (Tapo)     = 597,45 + 127,777 - 3,31 = 721,92 kWh
Consumo reale casa (Inverter) = 597,45 + 137,92 - 3,31  = 732,06 kWh
```

### Confronto con luglio 2025 (pre-FV, pre-piscina)

| Indicatore | Luglio 2025 | Luglio 2026 | Δ |
|---|---:|---:|---:|
| Prelievo rete | 468 kWh | 597,45 kWh | **+27,7%** |
| Consumo reale casa | 468 kWh | 721,92-732,06 kWh | **+54,3%/+56,4%** |

**Lettura preliminare (causa ora chiarita, §7):** a differenza di maggio e giugno, a luglio
il fotovoltaico **non basta a compensare** l'aumento del carico — sia il prelievo sia il
consumo reale crescono sensibilmente rispetto all'anno precedente. Questo è un'inversione di
tendenza rispetto al pattern consolidato nel progetto, ma **non indica un problema
nell'impianto FV o nelle misure**: è spiegata dall'uso intensivo di climatizzazione durante
l'ondata di caldo (verificato per plausibilità in §7). Il fotovoltaico continua a produrre
regolarmente (127,78-137,92 kWh, in linea con giugno), semplicemente il carico aggiuntivo
(condizionatori) è troppo grande per essere assorbito interamente dall'autoconsumo diurno
disponibile (impianto piccolo, 800 Wp, senza accumulo).

**Nota sul dato di confronto luglio 2025:** i 468 kWh derivano da uno split mensile
riportato nelle note della bolletta Plenitude Lug-Ago 2025 (fonte esterna, non dal PDF
fiscale bioraria che riporta solo il bimestre aggregato) — stesso livello di affidabilità
già usato per gli altri confronti anno-su-anno del progetto.

Salvato in `riconciliazione_mensile` con `versione_calcolo = 'preliminare (Sonoff, bolletta
non disponibile)'` — da sostituire con chiusura definitiva appena arriva la bolletta
Sorgenia di luglio.

---

## 9. Bollette (invariato da v10)

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

**Ancora mancante:** bolletta Sorgenia luglio 2026 — priorità **alta**, anche più che nei
mesi precedenti data l'anomalia di prelievo da chiarire (§7).

---

## 10. Formule di riconciliazione (usare sempre)

```
Autoconsumo FV     = Produzione FV - Immissione
Consumo reale casa = Prelievo rete + Produzione FV - Immissione
```

Gerarchia fonti per il prelievo: **bolletta ufficiale > Sonoff/eWeLink** (usare Sonoff solo
per chiusure preliminari, sostituire con bolletta appena disponibile — v10/v11).

Verifica incrociata Sonoff vs bolletta:
- Gen 2026: −0,8 kWh (~0,2%) ✓
- Feb 2026: −0,5 kWh (~0,1%) ✓
- Giugno 2026: −4,58 kWh (~1,33%) ✓

Verifica incrociata produzione FV: foglio Anno = somma foglio Mese, confermato anche per
luglio (127,776975 kWh esatto).

---

## 11. Pendenze aperte (aggiornate v11)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| ~~Chiarire il prelievo anomalo di luglio~~ | Utente | ~~Alta~~ | **RISOLTO (v11)** — causa: climatizzazione intensiva, vedi §7 |
| Bolletta Sorgenia **luglio 2026** | Sorgenia | **Alta** | Necessaria per chiusura definitiva (attualmente solo preliminare) |
| Produzione FV **agosto 2026** — completare | Tapo + inverter | Media | Solo 4/31 giorni |
| Push di questa sessione (v10+v11) su GitHub | — | **Alta** | **In attesa del token** — commit locale non ancora pushato |
| Analisi economica sostituzione pannelli/inverter | — | Media | In sospeso da v7 |
| Decisione: promuovere inverter a fonte primaria? | — | Media | In sospeso da v5 |
| Rigenerare token GitHub | — | Consigliata | Riutilizzato in molte sessioni |

**Azione a più alto impatto per la prossima sessione:** ingest della bolletta Sorgenia di
luglio appena disponibile, per trasformare la chiusura preliminare in definitiva (come già
fatto per giugno in v10).

---

## 12. Regole operative del progetto (invariate da v10 + 1 nuova)

1-19. Vedi v10 (invariate).

20. **(Nuova, v11)** Quando un mese mostra un pattern che rompe una tendenza consolidata nel
    progetto (es. prelievo che aumenta invece di calare), segnalarlo con enfasi e **non**
    proporre spiegazioni non verificate come fossero conclusioni — elencarle come ipotesi
    aperte da confermare con l'utente.

---

## 13. Strumentazione (invariata da v10)

Vedi v10 §14. Nuova nota: l'export Tapo `Consumo_di_Energia.xls` (fogli Anno/Mese/Giorno) è
lo stesso formato canonico già usato nelle sessioni precedenti — nessuna differenza di
schema o affidabilità.

---

## 14. Conclusione attuale del progetto

Fino a giugno 2026, il progetto ha un quadro solido e ben verificato: il fotovoltaico riduce
sistematicamente il prelievo rispetto all'anno precedente (maggio -20,1%, giugno -21,7%),
anche con la piscina attiva.

```
Giugno 2025 → 440 kWh
Giugno 2026 → 344,5 kWh (bolletta ufficiale)
Riduzione   → -95,5 kWh (-21,7%)
```

**Luglio 2026 rompe il pattern positivo, ma con causa nota:** il prelievo (597,45 kWh, dato
Sonoff completo) è superiore sia a giugno sia a luglio 2025 pre-fotovoltaico. Il consumo
reale della casa (fonte inverter) sale del +56% rispetto a luglio 2025. Questo è il primo
mese del progetto in cui il FV non basta a mantenere il prelievo sotto il livello dell'anno
precedente — **causa chiarita**: uso intensivo di climatizzazione (split salone 12000 BTU +
multisplit 3 camere) durante l'ondata di caldo, verificato per plausibilità (§7). Non
riflette un problema di impianto o di misura, ma un nuovo pattern stagionale/comportamentale
da tenere presente nei confronti futuri (es. agosto, o luglio dell'anno prossimo).

La chiusura di giugno (piscina) resta **definitiva e solida**: +5,1%/+7,4% consumo reale,
-21,7% prelievo rete, versione v10 basata su bolletta ufficiale.

**Frontiera aperta più urgente:** capire la causa dell'aumento di prelievo a luglio, e
ottenere la bolletta Sorgenia di luglio per la chiusura contabile definitiva.

---
*Fine handoff v11. Prossima azione consigliata: (1) chiedere all'utente possibili cause
dell'aumento di consumo a luglio (piscina, caldo, ospiti, nuovi carichi); (2) ingest bolletta
Sorgenia luglio appena disponibile; (3) push del commit locale su GitHub (serve token); (4)
completare produzione FV agosto; (5) rigenerare token GitHub.*
