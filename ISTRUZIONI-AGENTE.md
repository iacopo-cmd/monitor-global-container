# Monitor Global — installazione del client per container (istruzioni per un agente)

Queste istruzioni sono per un agente (per esempio OpenClaw) che installa il client di Monitor Global **dentro un
container o come utente senza root**. Il client misura la macchina (CPU, sottrazione di CPU da parte dell'hypervisor,
rete) e manda i risultati al server di Monitor Global. Legge soltanto: non cambia la configurazione, non apre porte,
non installa pacchetti. **Non serve nessuna chiave**: la macchina parte anonima.

## Come comunicare con noi

- PRIMA di agire, di' cosa stai per fare (quali comandi), poi fallo.
- Lavora a BLOCCHI: al massimo 3 o 4 comandi per messaggio. Dopo ogni blocco manda un messaggio con cosa hai fatto,
  l'esito e cosa farai nel blocco dopo. Non restare in silenzio fino alla fine.
- Se ti fermi (errore, dubbio), dillo subito.

## Regole

- Non modificare il codice né i percorsi. Non usare sudo. Non installare pacchetti se non te lo diciamo noi.
- I file in `stato/` (`proprietario.txt`, `sale`, `id_locale`) sono SEGRETI della macchina: non mostrarli, non copiarli
  altrove, non cancellarli. `proprietario.txt` è la prova che la macchina è nostra.
- Se un comando fallisce, fermati e riporta l'errore esatto. Non improvvisare correzioni.
- Lancia il client una volta sola: non far girare due demoni insieme.

## Blocco 1 — cartella e file

```bash
export MINIBENCH_DIR=~/.openclaw/monitor-global
mkdir -p "$MINIBENCH_DIR" && cd "$MINIBENCH_DIR"
curl -fsSO https://raw.githubusercontent.com/iacopo-cmd/monitor-global-container/main/monitor-global.py
curl -fsSO https://raw.githubusercontent.com/iacopo-cmd/minibench.sh/main/minibench.sh
curl -fsS https://raw.githubusercontent.com/iacopo-cmd/monitor-global-container/main/config.env.esempio -o config.env
```

Riporta: `ls -la "$MINIBENCH_DIR"` (nomi e dimensioni, non il contenuto).

## Blocco 2 — verifica (non manda misure)

```bash
cd "$MINIBENCH_DIR" && python3 -I monitor-global.py --verifica
```

Riporta tutte le righe. Se l'ultima dice **FERMO**, fermati qui. Se dice **PRONTO**, vai avanti.

## Blocco 3 — un test e il demone

```bash
cd "$MINIBENCH_DIR" && python3 -I monitor-global.py --solo test
nohup env MINIBENCH_DIR="$MINIBENCH_DIR" python3 -I -u monitor-global.py --demone > demone.log 2>&1 &
sleep 240; tail -20 demone.log
```

Il test della CPU dura alcuni minuti e carica la macchina: è normale. Nel registro devono comparire righe con
`test: 201`, `check: 201`, `target: 200`, `ping: 201`.

## Blocco 4 — cosa riportarci

- sistema (`. /etc/os-release; echo $PRETTY_NAME`) e quante CPU vedi (`nproc`);
- le ultime 20 righe di `demone.log`;
- eventuali errori, con il testo esatto.

Non riportare il contenuto dei file in `stato/`.

## Se il container si riavvia

Il demone non riparte da solo. Per riaccenderlo:

```bash
export MINIBENCH_DIR=~/.openclaw/monitor-global && cd "$MINIBENCH_DIR"
pgrep -f "monitor-global.py --demone" || nohup env MINIBENCH_DIR="$MINIBENCH_DIR" python3 -I -u monitor-global.py --demone >> demone.log 2>&1 &
```

## Iscrizione (solo quando te lo diciamo noi)

```bash
cd "$MINIBENCH_DIR" && python3 -I monitor-global.py --rivendica
```

Salva da sola la chiave in `$MINIBENCH_DIR/chiave` (600). Non mostrarla.

## Se va fermato

```bash
pkill -f "monitor-global.py --demone"
```

I file restano dove sono: non cancellarli se non te lo chiediamo.
