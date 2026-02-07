# Lab 06 — ECS/Fargate: cluster + task definition + service (minimale)

Riferimento lezione: `slides_deck/lecture_06_ecs_task_definition_en.md`

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
   - ECS → Clusters → Create cluster

3) **Crea una task definition (Fargate)**
   - ECS → Task definitions → Create
   - Launch type: Fargate
   - CPU/Mem: valori minimi compatibili
   - Container:
      - Image: `<account>.dkr.ecr.<region>.amazonaws.com/hello-api:1.0`
      - Port mapping: `8080` (se la tua app usa 8080)
   - Logging:
      - Abilita CloudWatch Logs (awslogs)

4) **Esegui un task (più veloce) oppure crea un service minimale**
   - Opzione A (rapida): Run task → 1 task
   - Opzione B: Service → desired count 1

5) **Verifica RUNNING**
   - ECS → Service/Tasks

6) **Leggi eventi e log**
   - ECS → Service → Events
   - CloudWatch Logs → log group del task

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
