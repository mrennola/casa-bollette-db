# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 21 — aggiornata al 30/09/2026

**Data:** 2026-09-30
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v21:**
1. **Scoperta e quantificata l'immissione post-sostituzione pannelli**: dal 2,9% al 29,2% della produzione, causata da DUE fattori concomitanti — più produzione (Megasol) E piscina spenta (rimossa insieme ai pannelli durante i lavori del 21-26/09).
2. **Simulazione stagionale 2026 completa** (produzione mese per mese, modello calibrato sui dati reali, scarto <3%) e analisi delle finestre di rischio immissione.
3. **Caratterizzazione dei carichi stagionali della casa** (confermata dall'utente): giu-set = AC + pompa piscina; nov-feb = pompa di calore riscaldamento. Le vere finestre di rischio immissione sono le transizioni (maggio, ottobre), non i mesi di picco.
4. **Verifica tecnica margine EVT800 post-sostituzione**: l'inverter ora satura ESATTAMENTE a 800W (prima arrivava solo a ~786W) — nessuna violazione della soglia plug&play (il limite è sull'uscita AC dell'inverter, non sulla potenza nominale dei pannelli), ma zero margine residuo invece di ~14W.
5. **Ricerca di mercato su accumulo (batterie plug&play)**: prezzi reali trovati (Anker Solarbank E1600 839€, Solarbank 2 AC non disponibile, EcoFlow STREAM AC Pro 609€). Payback stimato 6-10+ anni dato il volume di immissione ancora piccolo — sconsigliato per ora.
6. **Discussione scenario di upgrade formale** (aggiungere 2 pannelli Megasol + nuovo inverter, ~1,76 kWp): supererebbe la soglia 800W plug&play, richiedendo iter ordinario completo con Areti (pratica di connessione, elettricista abilitato, contatore bidirezionale, registrazione GSE) — ma apre alla possibilità di essere PAGATI per l'immissione, cosa che oggi non avviene.

---

## 1. Obiettivo del progetto

Invariato: ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo.

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v21

| Tabella | Righe | Note v21 |
|---|---:|---|
| `cronologia_eventi` | **18** (era 17) | + riga "caratterizzazione carichi stagionali" (piscina/AC giu-set, pompa di calore nov-feb) |
| Le altre tabelle | — | invariate da v20 |

Nessun'altra modifica di schema o dati in questa sessione — il lavoro è stato soprattutto analitico/di simulazione (vedi sotto), non ha richiesto nuovi import.

---

## 4. NOVITÀ v21 — L'immissione post-sostituzione è dovuta a DUE cause, non solo ai pannelli

Nella sessione precedente (v20) avevo isolato l'effetto pannelli dall'effetto meteo. In questa sessione l'utente ha segnalato un secondo fattore confondente: **la piscina è stata scollegata insieme ai pannelli durante i lavori di sostituzione** (21-26/09), e non risulta ancora riattivata.

```
Immissione/produzione:
  Pre-sostituzione  (14-21/09, 8 gg) =  2,9%
  Post-sostituzione (26-29/09, 4 gg) = 29,2%
```

Questo salto non isola più solo l'effetto "pannelli più potenti" — è confuso con "niente più carico piscina che assorbe il surplus". Il dato di produzione totale (misurato all'origine, +54-59%) resta valido; è la lettura dell'immissione/autoconsumo che va contestualizzata con questo secondo fattore.

---

## 5. NOVITÀ v21 — Simulazione stagionale 2026 completa

Modello di produzione calibrato sui 5 mesi misurati con i pannelli Dahai (apr-ago 2026, scarto stima/reale sempre <3%), poi proiettato su tutti i 12 mesi e aggiornato con il boost Megasol (+54/+59%) da ottobre in poi.

| Mese | Produzione FV | Prelievo rete | Immissione | Stato |
|---|---:|---:|---:|---|
| Gennaio | — | 501,2 kWh | 0,00 kWh | confermato — impianto non installato |
| Febbraio | — | 447,5 kWh | 0,00 kWh | confermato — impianto non installato |
| Marzo | 23,8 kWh | 504,3 kWh | 0,00 kWh | confermato — FV dal 23/24, mese quasi tutto senza |
| Aprile | 101,5 kWh | 314,2 kWh | 0,00 kWh | confermato |
| Maggio | 115,2 kWh | 291,8 kWh | 13,46 kWh | confermato |
| Giugno | 124,5 kWh | 340,0 kWh | 6,74 kWh | confermato (piscina attiva) |
| Luglio | 127,8 kWh | 597,7 kWh | 3,31 kWh | confermato (piscina + clima) |
| Agosto | 115,8 kWh | 566,1 kWh | 7,39 kWh | confermato |
| Settembre | ~105 kWh (stima, mese misto) | 367,4 kWh | 8,96 kWh | in parte confermato, mese misto |
| Ottobre | **116-120 kWh** | da stimare | **stima: bassa, in calo** | stima |
| Novembre | **86-89 kWh** | da stimare | **stima: vicina a zero** | stima |
| Dicembre | **75-77 kWh** | da stimare | **stima: zero/quasi zero** | stima |

**Totale produzione FV stimato anno solare 2026: ~880-900 kWh** su un impianto passato da 0,8 a 0,88 kWp a fine settembre.

**Perché l'immissione crolla in inverno:** nei mesi gen-apr 2026 (pre-fotovoltaico o con FV appena installato), il consumo di base della casa (448-504 kWh/mese, presumibilmente già con pompa di calore attiva) ha assorbito interamente anche produzioni >100 kWh/mese, con immissione a zero. Lo stesso pattern è atteso per nov-feb 2026/2027 con i nuovi pannelli: la produzione (anche +55%) resta piccola rispetto al carico della pompa di calore.

---

## 6. NOVITÀ v21 — Caratterizzazione dei carichi stagionali (confermata dall'utente)

L'utente ha confermato il pattern dei due grandi carichi elettrici di casa, mai contemporanei:

- **Giugno-settembre**: condizionatori + pompa piscina (carico diurno, coincide con la stagione di massima produzione FV)
- **Novembre-febbraio**: pompa di calore per il riscaldamento (presumibilmente carico anche diurno, coincide con la stagione di minima produzione FV)

**Implicazione chiave**: le vere finestre di rischio immissione/eccedenza non sono i mesi di picco caldo o freddo (dove i grandi carichi assorbono tutto), ma le **due finestre di transizione dove nessuno dei due è attivo**:
- **Maggio** — dopo il riscaldamento, prima di piscina/AC (confermato: mese con più immissione del "vecchio" impianto, 13,46 kWh)
- **Ottobre-inizio novembre** — dopo piscina/AC, prima della pompa di calore (confermato dal picco di immissione concentrato di fine settembre: 5,47 kWh in soli 4 giorni)

L'utente comunicherà la data di accensione della pompa di calore per affinare l'analisi mese per mese quando disponibile.

---

## 7. NOVITÀ v21 — Verifica margine EVT800: nessuna violazione, ma zero margine

Confrontando la potenza istantanea massima registrata dall'inverter:

| Periodo | Potenza istantanea massima |
|---|---:|
| Pre-sostituzione (Dahai, 800Wp) | 786 W |
| **Post-sostituzione (Megasol, 880Wp)** | **800,0 W esatti, sostenuti per diversi minuti a mezzogiorno** |

L'inverter EVT800 è fisicamente incapace di superare gli 800W in uscita (limite hardware) — la soglia normativa plug&play (800W) si applica alla potenza immessa in rete, non alla potenza nominale dei pannelli, quindi **non c'è alcuna violazione**. Prima l'impianto restava con ~14W di margine; ora satura esattamente al limite, con zero margine residuo. Non è un problema ma va monitorato se si aggiungono ulteriori pannelli in futuro.

---

## 8. NOVITÀ v21 — Ricerca accumulo (batterie plug&play): prezzi reali, conclusione negativa per ora

Ricerca di mercato (via browser, il dominio api.open-meteo.com resta l'unico bloccato — Amazon.it, MediaWorld e i siti produttore sono raggiungibili normalmente):

| Prodotto | Capacità | Prezzo | Compatibilità |
|---|---:|---:|---|
| Anker SOLIX Solarbank E1600 (DC, tra pannelli e microinverter) | 1,6 kWh | 839 € (scontato da 1.199 €) | Compatibile 11-60V (Deye, Hoymiles, APsystems dichiarati) |
| Anker SOLIX Solarbank 2 E1600 AC (AC, dopo il microinverter) | 1,6 kWh, espandibile a 9,6 kWh | Non disponibile al momento della ricerca | 100% compatibile con qualsiasi microinverter esistente — la scelta tecnicamente più semplice per il tuo caso |
| EcoFlow STREAM AC Pro | 1,92 kWh | 609 € (sceso da 649 €) | Dichiarato 100% compatibile con tutti i microinverter |

**Non esistono taglie più piccole** di ~1,6 kWh per questa categoria di prodotto dedicato (sotto quella soglia non è economicamente sensato costruire l'elettronica MPPT+grid-tie dedicata); esistono solo power station generiche (es. EcoFlow River 2 Max, 512Wh, 299€) che però non catturano automaticamente il surplus senza uno smart plug/pinza amperometrica aggiuntiva.

**Conclusione economica**: con un'immissione stimata di 20-40 kWh/mese nei mesi migliori (e vicina a zero nov-feb), il risparmio annuo è nell'ordine di 50-100 €/anno → payback di 6-10+ anni sui prezzi trovati. **Non consigliato per ora**, a meno di un ampliamento dell'impianto (vedi punto successivo).

---

## 9. NOVITÀ v21 — Scenario discusso: upgrade formale a >800W

L'utente ha chiesto cosa comporterebbe aggiungere 2 pannelli Megasol + sostituire l'inverter (~1,76 kWp totali). Punti chiave (informazione generale, non verificata su fonti Areti/GSE aggiornate — da controllare prima di agire):

- Le fasce di potenza in Italia: ≤350W comunicazione minima; 351-800W "mini-fotovoltaico" (comunicazione unica + documentazione tecnica — **il suo impianto attuale è già in questa fascia**, non "senza carte" in senso stretto); >800W iter ordinario completo con progetto.
- Superando 800W: pratica di connessione formale ad Areti, elettricista abilitato obbligatorio (niente più fai-da-te), probabile sostituzione con contatore bidirezionale, possibilità di registrazione GSE (Scambio sul Posto o Ritiro Dedicato).
- **Vantaggio**: l'energia immessa comincerebbe finalmente a essere pagata, invece di essere regalata come ora.
- **Non verificati**: tempi, costi esatti (contributo di connessione, parcella elettricista), ed eventuali vincoli attuali (2026) sull'accesso allo Scambio sul Posto per nuovi impianti — da controllare direttamente sul portale Areti/GSE prima di decidere.

---

## 10. Dati invariati da v20

Dati inverter diretto (13-29/09), conferma cross-source Tapo/inverter (+54-59%), verifica meteo/irradianza, conferma utente sul fermo impianto 22-25/09 — tutto invariato. Vedi v19-v20 per il dettaglio.

---

## 11. Pendenze aperte (aggiornate v21)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **giugno 2026** — conto economico | Sorgenia | Alta | Invariato — unico mese pieno con piscina di cui manca ancora il lato € |
| Bolletta Plenitude **marzo-aprile 2026** (PDF originale) | Plenitude | Media | Ancora `stima` per lo split F2/F3 mensile |
| Data accensione pompa di calore (inverno 2026/2027) | Utente (comunicazione attesa) | Media | Serve per validare/correggere la proiezione "immissione vicina a zero nov-feb" |
| Prelievo rete proiettato ott-dic 2026 | — | Bassa | Non modellato in questa sessione (dipende da comportamento riscaldamento, non ancora osservato quest'inverno) |
| Verifica procedura/costi upgrade a >800W (Areti/GSE) | Areti/GSE (portale ufficiale) | Bassa | Solo se l'utente decide di valutare seriamente l'ampliamento impianto |
| Più settimane di produzione post-sostituzione pannelli | Tapo/inverter | Bassa | Il segnale è già confermato con ragionevole confidenza (v19-v20); utile solo per consolidare oltre il campione attuale |
| Analisi economica sostituzione pannelli/inverter | — | Bassa | invariato |
| Bolletta Sorgenia **settembre 2026** | Sorgenia (attesa metà ottobre) | Bassa | Mese ancora in corso |

**Azione a più alto impatto per la prossima sessione:** ingest bolletta Sorgenia giugno per chiudere l'ultima pendenza economica aperta. In parallelo, tenere traccia della data di accensione della pompa di calore quando l'utente la comunica.

---

## 12. Regole operative del progetto (invariate)

Tutte le regole precedenti restano valide. Vedi `CLAUDE.md` nel repo per le note operative su push GitHub (limite proxy di sessione, workaround via GitHub Desktop) e sul recupero dati Open-Meteo (via browser, non shell/WebFetch).

---

## 13. Conclusione attuale del progetto

Questa sessione è stata soprattutto di **analisi e simulazione** più che di nuovo ingest dati: a partire dal segnale di immissione osservato dopo la sostituzione pannelli, si è arrivati a (1) isolare correttamente i due fattori concomitanti (pannelli + piscina spenta), (2) costruire un modello di produzione stagionale ben calibrato sui dati storici (scarto <3%), (3) confermare con l'utente il pattern dei carichi stagionali della casa, che spiega perché le vere finestre di rischio immissione sono le transizioni (maggio, ottobre) e non i mesi di picco, (4) verificare che l'inverter EVT800 non sta violando alcuna soglia normativa nonostante ora saturi esattamente a 800W, e (5) valutare — con prezzi di mercato reali — che un accumulo non è economicamente sensato allo stato attuale, a meno di un ampliamento formale dell'impianto oltre gli 800W, discusso come scenario ma non ancora avviato.

Il progetto ha ora un quadro completo e coerente su produzione, autoconsumo e immissione per l'intero anno solare 2026, con solo il lato economico di giugno ancora da chiudere.

---
*Fine handoff v21. Prossima azione consigliata: (1) ingest bolletta Sorgenia giugno per l'ultima pendenza economica aperta; (2) quando l'utente comunica la data di accensione della pompa di calore, aggiornare la cronologia e validare la proiezione invernale; (3) se l'utente vuole approfondire l'upgrade a >800W, verificare procedura/costi reali su Areti/GSE prima di dare cifre; (4) continuare a monitorare il margine di corrente EVT800 (ora a saturazione esatta di 800W) se si aggiungono altri pannelli.*
