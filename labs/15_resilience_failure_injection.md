# Lab 15 - Resilienza: rompi e ripristina

## Obiettivo

- Simulare un errore controllato.
- Osservare il flusso di debug tra ALB, ECS e CloudWatch.
- Ripristinare il service in stato stabile.

## Durata

25 minuti.

## Prerequisiti

- `hello-api-svc` dietro ALB.
- Target Group e log CloudWatch disponibili.
- Immagine `hello-api` aggiornata: i path sconosciuti devono rispondere con HTTP `404` (necessario per far fallire l'health check ALB).

## Specifiche del laboratorio

- In questo lab rompi volontariamente una parte controllata del sistema per vedere come reagisce.
- Devi seguire una catena precisa di osservazione: stato target, eventi ECS, log CloudWatch.
- Il lab e completato solo quando hai anche ripristinato lo stato `healthy` e `stable`.

## Guida del lab

0. **Prepara il target group (consigliato)**
   - EC2 -> `Target Groups` -> `demo-tg` -> tab `Attributes` -> `Edit`
   - **Deregistration delay**: da `300` a `30` secondi -> `Save`
   - Durante i deploy ECS i target vecchi spariscono piu in fretta. Utile in aula; in produzione spesso si preferisce un valore piu alto.

1. **Controlla lo stato iniziale**
   - EC2 -> `Target Groups` -> `demo-tg` -> `Targets`
   - Tutti i target devono essere `healthy`.
   - ECS -> `hello-api-svc` -> verifica stato `stable`.

2. **Rompi il health check**
   - EC2 -> `Target Groups` -> `demo-tg` -> `Health checks` -> `Edit`
   - Cambia path da `/health` a `/does-not-exist`
   - Salva le modifiche

3. **Osserva cosa succede**
   - Attendi che il target diventi `unhealthy` (circa `interval` × `unhealthy threshold`, es. 10s × 2 = ~20s).
   - Apri `Health status details` e verifica la reason: `Health checks failed with these codes: [404]`.
   - Controlla anche `Deployments and events` nel service.

4. **Leggi i log** 🎯 _Sfida_
   - CloudWatch -> log group di `hello-api`
   - Cerca richieste fallite o segnali utili al debug.
   - Scrivi l'ordine del tuo debug: ALB, ECS, CloudWatch.

5. **Ripristina il path corretto**
   - Riporta il health check a `/health`
   - Attendi il ritorno a `healthy`

## Output atteso

- Target passato a `unhealthy` e poi tornato `healthy`.
- Catena di debug esercitata.

## Checkpoint

- Sai dove guardare per primo se l'app non risponde dietro ALB.
- Sai spiegare perche un health check corretto protegge i deploy.

## Troubleshooting

- **Nessun cambiamento nello stato target**: ricontrolla `interval` e `threshold` del health check. Se il target resta `healthy`, verifica che ECS stia usando l'immagine aggiornata di `hello-api` (path sconosciuti devono restituire `404`, non `200`).
- **Target in `draining` per molti minuti**: non e lo stesso di `unhealthy`. Succede quando ECS sostituisce un task; con deregistration delay a `30` s compare per poco tempo, con il default `300` puo durare fino a 5 minuti.
- **Non hai un ALB**: fai la versione alternativa rompendo il comando del container e osserva `Stopped reason`.

## Cleanup

- Rimetti il health check al valore originale.
- Verifica che il service sia tornato `stable`.
