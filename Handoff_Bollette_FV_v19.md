# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 19 — aggiornata al 29/09/2026

**Data:** 2026-09-29
**Tipo:** handoff / continuità
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici
**Novità v19:**
1. **Dati orari inverter diretto (EVT800/EnverView) estesi dal 13/09 al 29/09/2026** (+8.160 letture, entrambi i pannelli), colmando esattamente la finestra pre/post sostituzione pannelli che mancava in v18.
2. **Segnale +56,4% (Tapo) CONFERMATO anche sulla fonte hardware primaria (inverter diretto): +54,1%** — doppia conferma indipendente dell'effetto della sostituzione pannelli Dahai→Megasol.
3. **22-25/09 confermati dall'utente**: impianto fisicamente scollegato per la sostituzione, nessun dato disponibile in quella finestra (coerente con lo 0,00 kWh/gg già registrato in v18).
4. **Nessun segnale di limitazione di corrente EVT800** nei dati orari post-sostituzione osservati finora (26-29/09): potenza massima per pannello resta clippata a 400W come prima, stima di corrente nei picchi (~11A) sotto il limite noto di 14A — ma campione ancora piccolo (4 giorni).
5. `verifica_bias_tapo_inverter` esteso di 12 righe (14-21/09, 26-29/09): bias sistematico confermato, +5-7%, leggermente più basso nei giorni Megasol (~5%) che nei giorni Dahai (~7%) — da tenere d'occhio ma non ancora significativo.

---

## 1. Obiettivo del progetto

Invariato: ricostruire in modo contabile e verificabile l'andamento dei consumi elettrici, confrontando bollette, produzione fotovoltaica, dati contatore, cambiamenti tecnici e dati meteo.

---

## 2. Nota privacy (invariata)

Dati identificativi mai memorizzati nel repo/handoff/DB. Dati tecnici utenza: Areti (Lazio), domestico residente, 4,5/5,0 kW, 220V, misuratore 2G.

---

## 3. Database — struttura aggiornata v19

| Tabella | Righe | Note v19 |
|---|---:|---|
| `letture_inverter_orarie` | **117.342** (era 109.182) | + 8.160 letture, copertura ora 09/05 → **29/09/2026 17:55** (senza buchi tra 14/09 e 29/09, tranne 22-25/09 dove l'impianto era fisicamente scollegato) |
| `produzione_giornaliera` | 165 | `kwh_inverter` compilato per 14-21/09 e 26-29/09 (era vuoto, `None`) |
| `verifica_bias_tapo_inverter` | **112** (era 100) | + 12 righe (14-21/09, 26-29/09) |
| `cronologia_eventi` | 17 | riga sostituzione pannelli aggiornata con la conferma cross-source e il check margine di corrente |
| Le altre tabelle | — | invariate da v18 |

---

## 4. NOVITÀ v19 — Il segnale +56% è confermato anche sull'inverter diretto

Fonte dati: 12 export EnverView "Daily Report" (uno per giorno, entrambi i pannelli S/N 30578336 e 30578337 per file), caricati dall'utente il 29/09 subito dopo che gli avevo segnalato il buco dati post-13/09.

| Fonte | Pre-sostituzione | Post-sostituzione (27-29/09, 3 gg) | Incremento |
|---|---|---:|---:|
| Tapo (v18) | 15-21/09, 7 gg → 3,28 kWh/gg | 5,13 kWh/gg | **+56,4%** |
| **Inverter diretto (v19)** | 14-21/09, 8 gg → 3,50 kWh/gg | **5,39 kWh/gg** | **+54,1%** |

Le due stime indipendenti convergono entro 2,3 punti percentuali — una convergenza notevole che rafforza sensibilmente la fiducia nel segnale. Resta comunque un dato da trattare con cautela: il campione post-sostituzione è ancora di soli 3 giorni puliti (27-29/09), e non è possibile isolare del tutto l'effetto pannello da un'eventuale differenza di irradianza tra le due finestre di osservazione (mancano ancora dati meteo oltre il 04/08 — vedi pendenze).

Giorno di transizione: 26/09 (reinstallazione ore 13:00) → 2,58 kWh inverter / 2,45 kWh Tapo, coerente come giorno parziale (solo pomeriggio).

Bias sistematico Tapo/inverter nella finestra 14-29/09: +5,03% a +7,28%, media pesata sui 12 giorni ≈ +6,3% — in linea con il pattern storico (~7-8%). Leggero calo nei 4 giorni Megasol (media ~5,1%) vs Dahai (media ~6,7%): possibile effetto della maggiore potenza di picco che Tapo (soglia di campionamento diversa) potrebbe sottostimare leggermente di più nei picchi, ma il campione è troppo piccolo per trarre conclusioni.

---

## 5. NOVITÀ v19 — Verifica margine di corrente EVT800 (nessun segnale di criticità, per ora)

Preoccupazione documentata in `pannelli_fv_specifiche` fin da agosto: con i pannelli Megasol bifacciali, la corrente Impp stimata (anche con guadagno bifacciale minimo) potrebbe superare il limite di 14A di corrente continua massima per ingresso dell'EVT800.

Dati orari 26-29/09 (post-sostituzione) analizzati:

```
Potenza massima osservata per pannello: 400,0 W (identica al clipping già noto pre-sostituzione, nessun aumento)
Tensione DC ai picchi di potenza: 35,0-37,1 V
Corrente stimata (I = P/V) ai picchi: ~10,8-11,4 A → sotto il limite di 14A
```

**Nessun segnale di limitazione di corrente rilevato finora.** Va però tenuto presente che il campione è ancora piccolo (4 giorni, fine settembre, irradianza non massima) e che il guadagno bifacciale reale può variare molto con le condizioni di montaggio e riflettanza del suolo — da ricontrollare nei mesi con irradianza più alta (tarda primavera/estate 2027) o se si osservano giorni di cielo particolarmente luminoso/superficie riflettente.

---

## 6. NOVITÀ v19 — Conferma utente sul buco 22-25/09

L'utente ha confermato esplicitamente che dal 22 al 25/09 "i pannelli erano staccati" — nessun dato disponibile in quella finestra per nessuna fonte (Tapo, inverter, eWeLink lato produzione). Questo è pienamente coerente con quanto già registrato in `produzione_giornaliera` (0,00 kWh/gg quei 4 giorni) e non richiede alcuna correzione: il buco è un fermo impianto reale, non un buco di dati da colmare.

---

## 7. Dati invariati da v18

Bollette (incluso agosto, chiuso sia energeticamente che economicamente), scheda EVT800/pannelli, cronologia tecnica precedente, `letture_quadro` (fermo a 29/09 21:00), `riconciliazione_mensile` — tutto invariato. Vedi v18 per il dettaglio.

---

## 8. Pendenze aperte (aggiornate v19)

| Dato mancante | Fonte | Urgenza | Note |
|---|---|---|---|
| Bolletta Sorgenia **giugno 2026** — conto economico | Sorgenia | Alta | Invariato da v18 — unico mese pieno con piscina di cui manca ancora il lato € |
| Bolletta Plenitude **marzo-aprile 2026** (PDF originale) | Plenitude | Media | Ancora `stima` per lo split F2/F3 mensile |
| Più giorni di produzione post-sostituzione pannelli | Tapo/inverter | **Alta** | Solo 3-4 giorni puliti per il +54/+56% osservato (ora confermato su due fonti) — servono almeno 2-3 settimane per consolidare definitivamente |
| Dati meteo oltre il 04/08 | Open-Meteo | **Alta** | Necessario per isolare l'effetto irradianza dall'effetto pannello nel confronto pre/post sostituzione — ora la pendenza più critica per validare il +54/+56% |
| Monitoraggio margine di corrente EVT800 nei mesi ad alta irradianza | EnverView | Media | Nessun segnale di criticità nei 4 giorni post-sostituzione osservati (fine settembre); da riverificare in primavera/estate 2027 |
| Analisi economica sostituzione pannelli/inverter | — | Bassa | invariato |
| Bolletta Sorgenia **settembre 2026** | Sorgenia (attesa metà ottobre) | Bassa | Mese ancora in corso |

**Azione a più alto impatto per la prossima sessione:** raccogliere dati meteo (irradianza) per la finestra 14-29/09 per poter finalmente isolare l'effetto pannello dall'effetto irradianza nel confronto pre/post sostituzione — è l'unico tassello mancante per passare da "preliminare, forte" a "confermato" sul +54-56%. In alternativa/parallelo: ingest bolletta Sorgenia giugno per chiudere l'ultima pendenza economica aperta.

---

## 9. Regole operative del progetto (invariate)

Tutte le regole precedenti restano valide e sono state applicate senza eccezioni. Vedi anche `CLAUDE.md` nel repo per le note operative su push GitHub e convenzioni.

---

## 10. Conclusione attuale del progetto

La novità più rilevante di questa sessione è la **conferma cross-source** del segnale di incremento di produzione dopo la sostituzione dei pannelli Dahai→Megasol bifacciali: +56,4% su Tapo (v18) e **+54,1% sull'inverter diretto EVT800** (v19, fonte hardware di primo livello), due misure indipendenti che convergono entro 2,3 punti percentuali. Questo è un segnale molto più solido di quanto suggerisse la sola fonte Tapo in v18, anche se il campione post-sostituzione resta piccolo (3-4 giorni) e manca ancora un controllo sull'irradianza per escludere del tutto un effetto meteo confondente.

Sul fronte tecnico, il monitoraggio del margine di corrente dell'inverter EVT800 (preoccupazione sollevata fin da agosto per l'accoppiamento con pannelli bifacciali più performanti) non ha per ora rilevato segnali di criticità: la potenza per pannello resta clippata a 400W come prima, e le stime di corrente ai picchi restano sotto il limite di 14A. Il campione è però ancora limitato a giornate di fine settembre, non alla massima irradianza stagionale.

L'utente ha inoltre confermato esplicitamente il fermo impianto 22-25/09 (pannelli fisicamente scollegati), chiudendo ogni dubbio residuo su quel buco di dati.

---
*Fine handoff v19. Prossima azione consigliata: (1) recuperare dati meteo/irradianza per la finestra 14-29/09, per isolare l'effetto pannello dall'effetto irradianza sul +54-56%; (2) ingest bolletta Sorgenia giugno per l'ultima pendenza economica aperta; (3) continuare a raccogliere giorni di produzione post-sostituzione (Tapo e/o inverter) per consolidare la stima oltre il campione attuale di 3-4 giorni; (4) rimonitorare il margine di corrente EVT800 nei mesi a più alta irradianza (primavera/estate 2027).*
