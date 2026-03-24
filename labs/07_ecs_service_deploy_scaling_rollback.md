# Lab 07 — ECS Service: update, scaling e rollback (timebox)

## Obiettivo

- Aggiornare un **ECS Service** (nuova task definition revision).
- Vedere eventi di deployment e fare un rollback.
- (Opzionale) impostare una regola di autoscaling semplice.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Un cluster ECS già presente.
- Un service ECS già creato (anche dal Lab 06) con 1 task.
- Logging su CloudWatch (consigliato).

---

## Mini-project (ongoing)

Porta avanti il deployment del tuo servizio “hello-api”.

Deliverable:

- esegui un update del service usando una nuova task definition revision (cambia tag immagine o una env var)
- fai scaling manuale (es. desired count 2) e verifica che il service torni “stable”
- se qualcosa va male, esegui rollback alla revision precedente e identifica l’errore dagli Events/log

---

## Step (numerati)

1. **Apri il service e osserva lo stato iniziale**
   - ECS ──► Cluster ──► Services ──► seleziona il service
   - Nota: `Desired count`, `Running count`, `Events`.

2. **Crea una nuova task definition revision**
   - Task definition ──► Create new revision
   - Cambia un parametro semplice (es. tag immagine o env var)

3. **Aggiorna il service** 🎯 _Sfida_
   - Service ──► Update
   - Seleziona la nuova revision
   - Avvia deployment.
   - _Sfida_: prima di confermare, annota quale deployment strategy è configurata (rolling update %).

4. **Osserva gli Events**
   - Output atteso: eventi di draining/starting.

5. **Rollback** 🎯 _Sfida_
   - Service ──► Update ──► seleziona la revision precedente
   - Output atteso: ritorno allo stato "stable".
   - _Sfida_: durante il rollback, osserva quanti task "old" e "new" coesistono.

6. **(Solo walkthrough concettuale) Autoscaling**
   - Service ──► Auto Scaling ──► policy semplice (CPU target)
   - ⚠️ **Nel lab AWS Academy** la policy non include `application-autoscaling:RegisterScalableTarget` né `PutScalingPolicy`. Questo step è trattato come **walkthrough concettuale** (slides + screenshots). Gli studenti annotano i parametri.

---

## Output atteso

- Service aggiornato con nuova revision.
- Rollback eseguito correttamente.

## Checkpoint

- Sai dove si vedono errori di rollout (Events).
- Sai distinguere “task definition revision” vs “service”.

---

## Troubleshooting rapido

- **Deployment non stabilizza**: controlla events + stopped reason + log.
- **CannotPullContainerError**: ECR permissions nell’execution role.

---

## Cleanup obbligatorio

- Riporta il service alla revisione stabile.
- Se hai creato policy di autoscaling non necessarie: rimuovile.

---

## Parole chiave Google (screenshot/guide)

- ECS service update task definition revision screenshot
- ECS deployment circuit breaker rollback
- ECS service autoscaling CPU target tracking screenshot

---

## Tutorial consigliati

- [Amazon ECS — Updating a Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/update-service.html)
- [ECS Rolling Update](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-ecs.html)
- [ECS Service Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)

---

## Soluzioni

<details>
<summary>🎯 Sfida Step 3: deployment strategy (rolling update)</summary>

**Dove trovarla**: Service ──► Configuration ──► Deployment configuration

**Parametri tipici**:

- `minimumHealthyPercent`: 100 (non scendere mai sotto il desired count)
- `maximumPercent`: 200 (puoi avere il doppio temporaneamente)

**Significato pratico** (desired = 2 task):

- Con min 100%, max 200%: prima avvia 2 nuovi, poi draina 2 vecchi ──► **zero downtime**
- Con min 50%, max 100%: ferma 1 vecchio, avvia 1 nuovo ──► **risparmio risorse ma rischio**

**Best practice**: mantieni min 100% per servizi di produzione.

</details>

<details>
<summary>🎯 Sfida Step 5: osservare coesistenza task old/new</summary>

**Cosa vedere**:

1. Vai in **ECS ──► Cluster ──► Service ──► Tasks**
2. Durante il deployment vedrai:
   - Task con "Task definition revision: X" (vecchia)
   - Task con "Task definition revision: X+1" (nuova)

**Flusso tipico** (con min 100%, max 200%):

1. ECS avvia nuovi task (revision X+1)
2. Nuovi task passano health check
3. ECS mette in DRAINING i vecchi task
4. Vecchi task terminano dopo drain connections
5. Solo nuovi task rimangono

**Tip**: apri Events e osserva messaggi come:

- "has started 1 tasks"
- "has begun draining 1 tasks"
- "has stopped 1 running tasks"

</details>
