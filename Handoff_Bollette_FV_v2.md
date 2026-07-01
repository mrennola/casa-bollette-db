# Handoff — Analisi consumi elettrici, fotovoltaico e bollette
## Versione 2 — aggiornata al 30/06/2026

**Data:** 2026-06-30  
**Tipo:** handoff/continuità  
**Progetto:** Analisi tecnico-contabile consumi elettrici domestici

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
- **Canone TV** = sempre escluso dai confronti energetici

---

## 2. Database e infrastruttura dati

### Repository GitHub
- **URL pubblico:** `https://github.com/mrennola/casa-bollette-db`
- **Visibilità:** pubblico (no dati identificativi nel DB)
- **File principale:** `reqa_bollette.db` (SQLite)

### Come accedere in sessione futura
```bash
git clone https://github.com/mrennola/casa-bollette-db.git
cd casa-bollette-db
# poi usare python3 + sqlite3 per le query
```
Nessun token necessario per la lettura. Per scrivere aggiornamenti
serve il token `claude-casa-bollette` (fine-grained, Contents Read/write,
scadenza 28/09/2026) — fornire come file `token.txt`.

**Nota operativa token:** il token va configurato correttamente su GitHub:
Settings → Developer settings → Fine-grained tokens → `claude-casa-bollette`
→ Edit (quello accanto a "Access on mrennola", NON quello del nome)
→ Only select repositories → `casa-bollette-db`
→ Permissions → Repositories → Contents: Read and write → Update.

### Struttura del database

| Tabella | Righe | Contenuto |
|---|---|---|
| `bollette` | 8 | Bollette Plenitude/Sorgenia con F1/F2/F3 e importi |
| `produzione_fv` | 4 | Produzione FV mensile (mar/apr confermati, mag/giu da aggiornare) |
| `produzione_giornaliera` | 29 | Dettaglio giornaliero produzione (mar-mag 2026) |
| `letture_quadro` | 4536 | Letture orarie Sonoff POWCT (prelievo+immissione, dic2025-giu2026) |
| `cronologia_eventi` | 10 | Date eventi tecnici chiave |
| `fornitori_storico` | 3 | Prezzi medi comparati Plenitude/Sorgenia |

### Dati presenti nel database (riepilogo completo)

**Bollette (da bollette ufficiali verificate su PDF):**

| Periodo | F1 | F2 | F3 | Totale kWh | Luce € | Stato |
|---|---|---|---|---|---|---|
| Mar-Apr 2025 | 217 | 269 | 314 | 800 | 249,84 | confermato |
| Mag-Giu 2025 | 227 | 243 | 331 | 801 | 236,93 | confermato |
| Lug-Ago 2025 | 280 | 253 | 361 | 894 | 277,93 | confermato |
| Set-Ott 2025 | 244 | 269 | 303 | 816 | 255,34 | confermato |
| Nov-Dic 2025 | 276 | 276 | 400 | 952 | 286,09 | confermato |
| Gen-Feb 2026 (baseline pre-FV) | 297 | 306 | 347 | 950 | 290,35 | confermato |
| Mar-Apr 2026 (FV installato) | 217 | 257 | 339 | 813 | 253,69 | **stima** |
| Maggio 2026 (Sorgenia) | 41,7 | 102,6 | 144,3 | 288,6 | 88,69 | confermato |

**Produzione FV mensile:**

| Mese | kWh | Stato |
|---|---|---|
| Marzo 2026 | 23,84 | confermato (foglio Anno Tapo) |
| Aprile 2026 | 101,51 | confermato (foglio Anno Tapo) |
| Maggio 2026 | — | da aggiornare |
| Giugno 2026 | — | da aggiornare |

**Letture orarie Sonoff (prelievo / immissione mensile):**

| Mese | Prelievo Sonoff | Immissione Sonoff | Note |
|---|---|---|---|
| Dic 2025 | 154,95 | 0,00 | parziale (dal 23/12) |
| Gen 2026 | 501,20 | 0,00 | corrisponde a bolletta 502 kWh ✓ |
| Feb 2026 | 447,50 | 0,00 | corrisponde a bolletta 448 kWh ✓ |
| Mar 2026 | 504,34 | 0,00 | FV inizia a fine mese |
| Apr 2026 | 314,17 | 0,00 | FV con pannelli riposizionati |
| Mag 2026 | 291,82 | 13,46 | Sonoff legge immissione da metà mese |
| Giu 2026 | 334,05 | 6,59 | parziale (fino al 30/06) |

---

## 3. Dato tecnico confermato: effetto riposizionamento pannelli

Dalla serie di 34 giorni estratta da Tapo (foglio Mese/Anno):

| Periodo | n giorni | Media produzione |
|---|---|---|
| Pre-riposizionamento (23 mar - 6 apr) | 10 | 2,27 kWh/giorno |
| Post-riposizionamento (7 apr - 1 mag) | 17 | 3,71 kWh/giorno |
| **Incremento** | | **+63,9%** |

Confermato anche dal foglio Anno Tapo: aprile 101,51 kWh (media 3,38 kWh/giorno
sull'intero mese, coerente con la media pesata pre/post riposizionamento).

---

## 4. Confronti principali già consolidati

### Maggio 2025 vs maggio 2026 (confronto annuale più pulito)

| | F1 | F2 | F3 | Totale |
|---|---|---|---|---|
| Maggio 2025 | 95 kWh | 125 kWh | 141 kWh | 361 kWh |
| Maggio 2026 | 41,7 kWh | 102,6 kWh | 144,3 kWh | 288,6 kWh |
| Differenza | -53,3 kWh | -22,4 kWh | +3,3 kWh | -72,4 kWh |
| Variazione | **-56,1%** | -17,9% | +2,3% | **-20,1%** |

**Lettura:** F1 cala del 56%, F3 stabile → coerente con effetto FV.

### Baseline Gen-Feb 2026 vs maggio 2026

```
Media mensile baseline = 475 kWh/mese
Maggio 2026 = 288,6 kWh
Riduzione = -186,4 kWh/mese = -39,2%
```

### Cambio fornitore: effetto stimato

| Voce | Plenitude Mar-Apr 2026 | Sorgenia Mag 2026 |
|---|---|---|
| Prezzo medio quota consumi | 0,204600 €/kWh | 0,201421 €/kWh |
| Quota fissa | 14,025 €/mese | 8,190 €/mese |
| Quota potenza | 8,89 €/mese | 8,89 €/mese |

Beneficio cambio fornitore stimato: **~7-8 €/mese**
(principalmente dalla quota fissa più bassa, non dal prezzo energia)

Beneficio FV/minor prelievo vs baseline Gen-Feb 2026: **~45-50 €/mese**

---

## 5. Dati da completare (pendenze aperte)

| Dato mancante | Fonte | Urgenza |
|---|---|---|
| Bolletta Plenitude Mar-Apr 2026 (PDF originale) | Plenitude | Alta — attualmente in DB come **stima** |
| Produzione FV maggio 2026 completa | Tapo, foglio Anno — nuovo export | Alta |
| Produzione FV giugno 2026 completa | Tapo, foglio Anno — nuovo export | Alta |
| Bolletta Sorgenia giugno 2026 | Sorgenia (emissione attesa luglio 2026) | Media |
| Bolletta Sorgenia luglio 2026 | Sorgenia | Futura |

**Come aggiornare la produzione FV:** aprire app Tapo → grafico produzione →
selezionare vista "Anno" → esportare → il file contiene il foglio "Anno"
con totali mensili aggiornati. Caricare il file in chat e aggiornare DB.

---

## 6. Analisi da fare nella prossima sessione

### Giugno 2026 (quando disponibile bolletta Sorgenia)

Confronto con giugno 2025 (440 kWh, pre-FV) — sarà il più interessante
perché giugno 2026 include piscina attiva per tutto il mese.

Schema da produrre:

| Mese | Prelievo rete | Produzione FV | Immissione | Autoconsumo FV | Consumo reale |
|---|---|---|---|---|---|
| Giugno 2025 | 440 kWh | 0 | 0 | 0 | 440 kWh |
| Giugno 2026 | da bolletta | da Tapo | da Sonoff | calcolo | calcolo |

**Domanda chiave:**
```
La piscina aumenta il consumo reale,
ma il fotovoltaico riesce ad assorbirne una parte
riducendo il prelievo dalla rete?
```

Segnale da cercare: immissione in calo rispetto a maggio
(la pompa assorbe energia che altrimenti andrebbe in rete).

---

## 7. Formule di riconciliazione da usare sempre

```
Autoconsumo FV = Produzione FV - Immissione
Consumo reale casa = Prelievo rete + Produzione FV - Immissione
```

Verifica incrociata Sonoff vs bolletta ufficiale:
- Gen 2026: Sonoff 501,2 / Bolletta 502 → delta -0,8 kWh (OK, ~0,2%)
- Feb 2026: Sonoff 447,5 / Bolletta 448 → delta -0,5 kWh (OK, ~0,1%)

Il Sonoff è affidabile per il prelievo. Per l'immissione è attendibile
solo da metà maggio 2026 in avanti.

---

## 8. Regole operative del progetto

1. Il **canone TV va sempre escluso** dai confronti energetici/economici
2. I periodi bimestrali vanno sempre normalizzati su base mensile
3. Distinguere sempre tra dato **confermato** (PDF bolletta verificato),
   **stima** (non ancora verificato su PDF) e **da aggiornare**
4. Non attribuire risparmio al fornitore se i prezzi medi sono simili:
   verificare sempre se il risparmio deriva da meno kWh prelevati
5. Segnale FV affidabile = F1 cala, F3 stabile o in lieve aumento

---

## 9. Strumentazione

| Strumento | Dato misurato | Attendibilità |
|---|---|---|
| Sonoff POWCT / eWeLink | Prelievo orario dalla rete | Alta (validato su bollette) |
| Sonoff POWCT / eWeLink | Immissione oraria in rete | Attendibile da ~18/05/2026 |
| Tapo (foglio Anno) | Produzione FV mensile | Alta (dato aggregato ufficiale) |
| Tapo (foglio Giorno) | Produzione FV oraria | Solo ultima settimana disponibile |
| Bolletta Areti/distributore | Prelievo ufficiale fatturato | Dato definitivo |

---

## 10. Conclusione attuale del progetto

La riduzione osservata è **coerente e verificata**.

```
Maggio 2025 → 361 kWh
Maggio 2026 → 288,6 kWh
Riduzione → -72,4 kWh (-20,1%)
```

Dato decisivo:
```
F1 maggio 2025 = 95 kWh
F1 maggio 2026 = 41,7 kWh
Riduzione F1 = -56,1%
F3 sostanzialmente invariata (+2,3%)
```

Decomposizione del risparmio (vs baseline gen-feb 2026):
- Fotovoltaico/autoconsumo → ~45-50 €/mese (causa principale)
- Cambio fornitore → ~7-8 €/mese (causa secondaria)

Da giugno 2026 il progetto deve integrare l'effetto piscina/pompa,
verificando se l'aumento del consumo reale viene assorbito in parte
dall'autoconsumo FV invece che dal prelievo dalla rete.
