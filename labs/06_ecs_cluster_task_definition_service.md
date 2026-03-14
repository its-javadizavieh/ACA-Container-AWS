# Lab 06 — ECS/Fargate: cluster + task definition + service (minimale)

## Obiettivo

- Creare (o riusare) un **cluster ECS**.
- Definire una **task definition** (Fargate) corretta: CPU/mem, porte, `awslogs`, execution role vs task role.
- Avviare un **task** (o un **service** minimale) e verificare che rimanga in esecuzione.
- Leggere eventi ECS e log CloudWatch (debug flow).

## Durata (timebox)

30 minuti.

## Prerequisiti

- Account AWS con permessi su ECS, IAM, CloudWatch Logs.
- Un’immagine in ECR (consigliato: Lab 05).
- (Consigliato) Default VPC disponibile.

---

## Mini-project (ongoing)

Esegui la tua image di progetto su ECS.

Deliverable:

- la task definition punta alla tua image “hello-api” in ECR
- `awslogs` è abilitato e sai leggere i log in CloudWatch
- esegui 1 task (o un service con 1 task) e lo mantieni stabile in RUNNING

---

## Step (numerati)

1) **Scegli la Region del corso**

2) **Crea (o usa) un cluster ECS**
   - ECS ──► Clusters ──► Create cluster

3) **Crea una task definition (Fargate)** 🎯 *Sfida*
   - ECS ──► Task definitions ──► Create
   - Launch type: Fargate
   - CPU/Mem: valori minimi compatibili
   - Container:
      - Image: `<account>.dkr.ecr.<region>.amazonaws.com/hello-api:1.0`
      - Port mapping: `8080` (se la tua app usa 8080)
   - Logging:
      - Abilita CloudWatch Logs (awslogs)
   - *Sfida*: prima di salvare, annota quale combinazione CPU/Mem hai scelto e perché.

4) **Esegui un task (più veloce) oppure crea un service minimale**
   - Opzione A (rapida): Run task ──► 1 task
   - Opzione B: Service ──► desired count 1

5) **Verifica RUNNING**
   - ECS ──► Service/Tasks

6) **Leggi eventi e log** 🎯 *Sfida*
   - ECS ──► Service ──► Events
   - CloudWatch Logs ──► log group del task
   - *Sfida*: trova nel log una riga che conferma che la tua app è in ascolto (es. "listening on port 8080").

---

## Output atteso

- Cluster e task definition creati.
- 1 task in stato **RUNNING**.
- Log disponibili in CloudWatch.

## Checkpoint

- Sai spiegare differenza tra **task definition**, **task**, **service**.
- Sai trovare “Stopped reason” se un task fallisce.

---

## Troubleshooting rapido

- **Task STOPPED subito**: controlla immagine, port mapping, CPU/mem.
- **CannotPullContainerError**: controlla execution role e permessi ECR.
- **Nessun log**: verifica awslogs e log group.

---

## Cleanup obbligatorio

1) Stop task / Delete service creato per il lab.
2) CloudWatch Logs: elimina log group del lab (solo se creato apposta).

---

## Parole chiave Google (screenshot/guide)

- Amazon ECS create cluster console screenshot
- Amazon ECS task definition Fargate console screenshot
- ECS CannotPullContainerError troubleshooting
- CloudWatch Logs awslogs driver ECS Fargate

---

## Tutorial consigliati

- [Amazon ECS Developer Guide — Task Definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)
- [ECS Task Definition Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html)
- [Using awslogs Log Driver](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html)

---

## Soluzioni

<details>
<summary>🎯 Sfida Step 3: combinazioni CPU/Mem Fargate</summary>

Fargate ha combinazioni **fisse** di CPU e memoria:

| CPU (vCPU) | Memoria ammessa |
|------------|------------------|
| 0.25 | 0.5, 1, 2 GB |
| 0.5 | 1, 2, 3, 4 GB |
| 1 | 2, 3, 4, 5, 6, 7, 8 GB |
| 2 | 4–16 GB (incrementi di 1 GB) |
| 4 | 8–30 GB |

**Scelta consigliata per "hello-api"**: `0.25 vCPU + 0.5 GB` (minimo, basta per test).

**Perché**: è la combinazione più economica e sufficiente per un'app Node/Python semplice.

</details>

<details>
<summary>🎯 Sfida Step 6: trovare "listening on port" nei log</summary>

**Passo per passo**:

1. Vai in **CloudWatch ──► Logs ──► Log groups**
2. Cerca il log group (es. `/ecs/hello-api` o simile)
3. Clicca sul **log stream** più recente (nome = task ID)
4. Cerca la riga con "listening", "started", "ready" o simili

**Esempio output atteso**:

```
Server listening on port 8080
```

**Se non trovi nulla**: l'app potrebbe non stampare messaggi all'avvio — aggiungi un `console.log("Started")` nel codice.

</details>
