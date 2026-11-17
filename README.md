# Monitor Global — client per container (senza root)

Client di **Monitor Global Server** («Valutazione indipendente di VPS e connettività») nella versione per
**container e utenti senza root**: niente systemd, niente `/etc` né `/var/lib`, tutto in una cartella sola.

- **Senza chiave**: la macchina parte *anonima*. Il client crea un segreto suo (`stato/proprietario.txt`, permessi 600,
  non lascia mai la macchina) e manda solo l'impegno `sha256(sha256(segreto))`. I dati restano in quarantena finché
  non ci si iscrive con `--rivendica`, che manda la prova `sha256(segreto)` e salva la chiave vera.
- **Legge soltanto**: non cambia la configurazione della macchina, non apre porte, non installa pacchetti.
- **Privacy**: per riconoscere la macchina solo impronte HMAC-SHA256 con un sale che resta qui; per la rete solo sì/no.
- **Da container** le misure sono segnate «misura da container»: le vCPU viste possono essere un limite del container.

Requisiti: python3 ≥ 3.9, curl, openssl. ping e ip NON servono (se mancano il client li fa in Python); traceroute o tracepath consigliati (senza, i percorsi si saltano). Il benchmark è
[minibench.sh](https://github.com/iacopo-cmd/minibench.sh).

## Installazione e avvio (a blocchi)

**1. Cartella**
```bash
export MINIBENCH_DIR=~/monitor-global          # o dove si vuole, purché scrivibile
mkdir -p "$MINIBENCH_DIR" && cd "$MINIBENCH_DIR"
curl -fsSO https://raw.githubusercontent.com/iacopo-cmd/monitor-global-container/main/monitor-global.py
curl -fsSO https://raw.githubusercontent.com/iacopo-cmd/minibench.sh/main/minibench.sh
curl -fsS https://raw.githubusercontent.com/iacopo-cmd/monitor-global-container/main/config.env.esempio -o config.env
```

**2. Verifica** (non manda misure; deve finire con `PRONTO`)
```bash
python3 -I monitor-global.py --verifica
```

**3. Un test, poi il demone in sottofondo**
```bash
python3 -I monitor-global.py --solo test
nohup python3 -I -u monitor-global.py --demone > demone.log 2>&1 &
tail -f demone.log
```
`MINIBENCH_DIR` va esportata anche per il demone (stessa shell, o nel comando che lo avvia).

**4. Iscrizione** (quando si vuole: i dati già mandati restano della macchina)
```bash
python3 -I monitor-global.py --rivendica
```

## Regole per chi lo installa (anche agenti)
- Prima di agire dire cosa si fa; lavorare a blocchi di 3-4 comandi e riportare l'esito di ogni blocco.
- Se la verifica dice `FERMO`, fermarsi e riportare le righe con `NO`. Non modificare il codice né i percorsi.
- Non mostrare né cancellare `stato/proprietario.txt`, `stato/sale`, `stato/id_locale`, né la chiave dopo l'iscrizione.

Istruzioni passo per passo (anche per agenti): [ISTRUZIONI-AGENTE.md](ISTRUZIONI-AGENTE.md).

Licenza: MIT.
