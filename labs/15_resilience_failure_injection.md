# Lab 15 — Resilience: simulare un failure e osservare recovery

Riferimento lezione: `slides_deck/lecture_15_resilience_fault_tolerance_en.md`

## Obiettivo

- Simulare un piccolo errore (config/health) e osservare cosa succede.
- Esercitare il “debug flow”: events → stopped reason → logs → ALB target health.
- Ripristinare lo stato stabile.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Un ECS service dietro ALB **oppure** un service con health check.
- Accesso a CloudWatch Logs.

---

## Mini-project (ongoing)

Esercita una “operational drill” sul progetto “hello-api”.

Deliverable:

- rompi deliberatamente `/health` (o una config) e osserva target unhealthy / task restart
- ripristina e verifica ritorno a healthy + service stable
- descrivi il debug flow che hai seguito (events → stopped reason → logs → target health)

---

## Step (numerati)

1) **Apri target group health (stato iniziale)**
   - Target group → Targets
   - Output atteso: targets healthy.

2) **Rompi deliberatamente l’health check**
   - Target group → Health checks
   - Cambia path in uno inesistente (es. `/health-broken`)

3) **Osserva la transizione a unhealthy**
   - Output atteso: targets diventano unhealthy

4) **Osserva ECS service events e log**
   - Service → Events
   - CloudWatch Logs → log group

5) **Ripristina health check corretto**
   - Rimetti path originale (es. `/health`)

6) **Verifica ritorno a healthy**

---

## Output atteso

- Targets diventano unhealthy e poi tornano healthy.
- Flusso di troubleshooting esercitato.

## Checkpoint

- Sai elencare l’ordine corretto di debug.
- Sai dire perché health check è cruciale per deploy sicuri.

---

## Troubleshooting rapido

- **Non ho ALB**: alternativa: cambia command/env per far crashare il task e osserva STOPPED reason.
- **Targets non cambiano**: controlla intervalli/threshold del health check.

---

## Cleanup obbligatorio

- Riporta health check ai valori originali.
- Verifica che service sia “stable”.

---

## Parole chiave Google (screenshot/guide)

- ALB target group health check path screenshot
- target group unhealthy troubleshooting
- ECS service events unhealthy targets
- deployment circuit breaker rollback ECS
