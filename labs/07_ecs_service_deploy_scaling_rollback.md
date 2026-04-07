# Lab 07 - ECS Service: update, scaling e rollback (timebox)

## Obiettivo

- Creare un **ECS Service** dalla Console (campo per campo).
- Aggiornare il service con una nuova task definition revision.
- Vedere eventi di deployment e fare un rollback.
- (Opzionale) impostare una regola di autoscaling semplice.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Cluster `demo-cluster` già presente.
- Task definition `hello-api` già registrata (con image da ECR, logging CloudWatch, port mapping 9090).
- Security Group `demo-task-sg` già creato (inbound Custom TCP 9090 da 0.0.0.0/0).

---

## Mini-project (ongoing)

Porta avanti il deployment del tuo servizio "hello-api".

Deliverable:

- crea un service dal cluster
- esegui un update del service usando una nuova task definition revision (aggiungi env var `APP_VERSION=2.0`)
- fai scaling manuale (es. desired count 2) e verifica che il service torni "stable"
- se qualcosa va male, esegui rollback alla revision precedente e identifica l'errore dagli Events/log

---

## Step (numerati)

1. **Crea il Service**
   - ECS ──► Clusters ──► `demo-cluster` ──► tab **Services** ──► **Create**
   - La pagina **Create** è divisa in sezioni. Compilale:

   **Deployment configuration**
   - Application type: **Service**
   - Task definition → Family: `hello-api`
   - Revision: **LATEST**
   - Service name: `hello-api-service`
   - Service type: **Replica**
   - Desired tasks: **1**

   Espandi **Deployment failure detection**:
   - ☑ Use the Amazon ECS deployment circuit breaker
   - ☑ Rollback on failures

   **Networking**
   - VPC: default
   - Subnets: lascia quelle preselezionate
   - Security group → **Use an existing security group** → seleziona `demo-task-sg`
     - (rimuovi il SG `default` se presente)
   - Public IP: **Turned on**

   Lascia le altre sezioni ai valori di default ──► **Create**.

   - Output atteso: la colonna **Status** passa a _Active_, il running count sale a 1.

2. **Osserva lo stato iniziale del service**
   - Clicca il nome del service → tab **Deployments and events**
   - Nota: `Desired count`, `Running count`, eventi.
   - Tab **Tasks**: verifica che 1 task sia in stato `RUNNING`.
   - Copia il **Public IP** del task e prova in un browser: `http://<Public-IP>:9090/`

3. **Crea una nuova task definition revision**  🎯 _Sfida_
   - ECS ──► Task definitions ──► `hello-api` ──► **Create new revision** ──► **Create new revision**
   - Nella sezione **Container – 1** clicca il container name per espandere.
   - Scorri a **Environment variables – optional** ──► **Add environment variable**:
     - Key: `APP_VERSION`   Value: `2.0`
   - ──► **Create**
   - _Sfida_: quale numero di revision è stato creato?

4. **Aggiorna il service (Update)** 🎯 _Sfida_
   - ECS ──► Clusters ──► `demo-cluster` ──► Services ──► `hello-api-service` ──► **Update**
   - Revision: seleziona la nuova revision (es. `hello-api:2`)
   - ☑ Force new deployment
   - ──► **Update**
   - _Sfida_: prima di confermare, annota la configurazione del circuit breaker (è già abilitato dal passo 1).

5. **Osserva gli Events**
   - Tab **Deployments and events** → segui gli eventi in tempo reale.
   - Output atteso: eventi di draining/starting tipo:
     - "has started 1 tasks"
     - "has begun draining connections on 1 tasks"
     - "has stopped 1 running tasks"
     - Deployment status: **Completed**

6. **Scaling manuale (desired count)**
   - Service ──► **Update**
   - Cambia **Desired tasks** da 1 a **2** ──► **Update**
   - Attendi che `Running count` = 2.
   - Output atteso: 2 task in stato `RUNNING`.

7. **Rollback alla revision precedente**  🎯 _Sfida_
   - Service ──► **Update**
   - Revision: seleziona la revision precedente (es. `hello-api:1`)
   - ☑ Force new deployment
   - ──► **Update**
   - Output atteso: ritorno allo stato "stable" con la revision originale.
   - _Sfida_: durante il rollback, osserva quanti task "old" e "new" coesistono nella tab **Tasks**.

8. **(Solo walkthrough concettuale) Autoscaling**
   - Service ──► **Update** ──► espandi **Service auto scaling – optional**
   - Seleziona **Use service auto scaling**
   - Minimum number of tasks: 1
   - Maximum number of tasks: 4
   - Scaling policy type: **Target tracking** → ECSServiceAverageCPUUtilization → Target value: 70
   - ⚠️ **Nel lab AWS Academy** la policy potrebbe non avere i permessi `application-autoscaling:RegisterScalableTarget` né `PutScalingPolicy`. Questo step è trattato come **walkthrough concettuale** (slides + screenshots). Gli studenti annotano i parametri.

---

## Output atteso

- Service creato e raggiungibile su porta 9090.
- Service aggiornato con nuova revision (APP_VERSION=2.0).
- Scaling manuale a 2 task funzionante.
- Rollback eseguito correttamente.

## Checkpoint

- Sai dove si vedono errori di rollout (Deployments and events).
- Sai distinguere "task definition revision" vs "service".
- Sai attivare il circuit breaker con rollback on failures.

---

## Troubleshooting rapido

- **Deployment non stabilizza**: controlla events + stopped reason + log CloudWatch.
- **CannotPullContainerError**: ECR permissions nell'execution role.
- **Task si ferma subito**: immagine non trovata in ECR o porta sbagliata.
- **Connessione rifiutata**: controlla che il SG `demo-task-sg` abbia la regola TCP 9090 da 0.0.0.0/0, e che Public IP sia ON.

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

- [Amazon ECS - Updating a Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/update-service.html)
- [ECS Rolling Update](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-ecs.html)
- [ECS Service Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)
