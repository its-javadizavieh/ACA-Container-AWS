# Lab 15 - Resilienza: rompi e ripristina

## Obiettivo

- Simulare un errore controllato.
- Osservare il flusso di debug tra ALB, ECS e CloudWatch.
- Ripristinare il service in stato stabile.

## Durata

25 minuti.

## Prerequisiti

- `hello-api-service` dietro ALB.
- Target Group e log CloudWatch disponibili.

## Guida del lab

1. **Controlla lo stato iniziale**
   - EC2 -> `Target Groups` -> `demo-tg` -> `Targets`
   - Tutti i target devono essere `healthy`.
   - ECS -> `hello-api-service` -> verifica stato `stable`.

2. **Rompi il health check**
   - EC2 -> `Target Groups` -> `demo-tg` -> `Health checks` -> `Edit`
   - Cambia path da `/health` a `/health-broken`
   - Salva le modifiche

3. **Osserva cosa succede**
   - Attendi che il target diventi `unhealthy`.
   - Apri `Health status details`.
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

- **Nessun cambiamento nello stato target**: ricontrolla `interval` e `threshold` del health check.
- **Non hai un ALB**: fai la versione alternativa rompendo il comando del container e osserva `Stopped reason`.

## Cleanup

- Rimetti il health check al valore originale.
- Verifica che il service sia tornato `stable`.
