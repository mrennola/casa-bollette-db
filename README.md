# casa-bollette-db

Database SQLite per il progetto di analisi consumi elettrici, fotovoltaico e bollette
(Massimo Rennola). Nessun dato identificativo (no nomi, no POD, no indirizzi).

## Tabelle

- `bollette` — bollette Plenitude/Sorgenia per periodo, con F1/F2/F3 in kWh
- `produzione_fv` — produzione fotovoltaica mensile (kWh)
- `produzione_giornaliera` — dettaglio giornaliero produzione (periodo mar-mag 2026)
- `letture_quadro` — letture orarie Sonoff POWCT (prelievo + immissione), dic2025-giu2026

## Aggiornato da Claude (sessione assistita)
