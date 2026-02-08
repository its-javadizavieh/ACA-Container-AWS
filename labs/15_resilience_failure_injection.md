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

2) **Rompi deliberatamente l'health check** 🎯 *Sfida*
   - Target group → Health checks
   - Cambia path in uno inesistente (es. `/health-broken`)
   - *Sfida*: prevedi cosa succederà prima di applicare. Quanto tempo prima che i target diventino unhealthy?

3) **Osserva la transizione a unhealthy**
   - Output atteso: targets diventano unhealthy

4) **Osserva ECS service events e log** 🎯 *Sfida*
   - Service → Events
   - CloudWatch Logs → log group
   - *Sfida*: descrivi la catena di eventi: cosa succede al service quando i target sono unhealthy?

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

---

## Tutorial consigliati

- [ECS Service Health Checks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/healthcheck.html)
- [ALB Target Group Health Checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)
- [ECS Deployment Circuit Breaker](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-circuit-breaker.html)

---

## Soluzioni

<details>
<summary>Sfida Step 2: prevedere tempo unhealthy</summary>

**Calcolo tempo transizione a unhealthy**:

| Parametro | Valore default | Significato |
|-----------|----------------|-------------|
| Interval | 30s | Controllo ogni 30 secondi |
| Unhealthy threshold | 2 | 2 fallimenti consecutivi |
| Timeout | 5s | Aspetta max 5s per risposta |

**Tempo minimo**: `Interval × Unhealthy threshold` = 30s × 2 = **~60 secondi**

**Tempo massimo**: dipende da quando inizia il prossimo check

**Con config Lab 09** (interval 10s, threshold 2):

- Tempo: 10s × 2 = **~20 secondi** (più veloce)

**Tip**: in produzione, threshold più alto (3-5) evita falsi positivi.

</details>

<details>
<summary>Sfida Step 4: catena di eventi dopo unhealthy</summary>

**Sequenza eventi (con deployment circuit breaker abilitato)**:

1. **ALB health check fallisce** (path non esiste → 404)
2. **Target diventa Unhealthy** dopo N check falliti
3. **ALB smette di inviare traffico** a quel target
4. Se tutti i target sono unhealthy:
   - Utenti vedono **502 Bad Gateway** o timeout
5. **Se era un deployment in corso**:
   - Circuit breaker rileva failure
   - Rollback automatico alla revision precedente
6. **Se non era un deployment**:
   - I task restano RUNNING (non vengono uccisi)
   - Il service non fa nulla automaticamente
   - Devi intervenire tu!

**Evento ECS tipico**:

```
service hello-api has 1 unhealthy targets in target-group ...
```

**Importante**: ECS **non** riavvia i task solo perché sono unhealthy per ALB!
Lo fa solo se:

- Il container crasha (exit code != 0)
- Il container health check fallisce (HEALTHCHECK nel Dockerfile)

</details>
