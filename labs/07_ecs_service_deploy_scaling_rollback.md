# Lab 07 — ECS Service: update, scaling e rollback (timebox)

Riferimento lezione: `slides_deck/lecture_07_ecs_service_scaling_en.md`

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

1) **Apri il service e osserva lo stato iniziale**
   - ECS → Cluster → Services → seleziona il service
   - Nota: `Desired count`, `Running count`, `Events`.

2) **Crea una nuova task definition revision**
   - Task definition → Create new revision
   - Cambia un parametro semplice (es. tag immagine o env var)

3) **Aggiorna il service**
   - Service → Update
   - Seleziona la nuova revision
   - Avvia deployment.

4) **Osserva gli Events**
   - Output atteso: eventi di draining/starting.

5) **Rollback**
   - Service → Update → seleziona la revision precedente
   - Output atteso: ritorno allo stato “stable”.

6) **(Opzionale) Autoscaling**
   - Service → Auto Scaling → policy semplice (CPU target)

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
