# Lab 04 — ECS: eseguire il primo task su Fargate (console)

Riferimento lezione: `slides_deck/lecture_04_ecs_fundamentals_en.md`

## Obiettivo

- Avviare un **task ECS su Fargate** partendo da una definizione semplice.
- Capire dove leggere e in che ordine: **events**, **stopped reason**, **log**.
- Fare cleanup completo.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Account AWS con permessi ECS e CloudWatch.
- VPC esistente con almeno 2 subnet (anche default va bene).
- (Consigliato) Un security group pronto o permessi per crearne uno.

## Scenario

Lanciamo un task “hello” (immagine pubblica) su Fargate per imparare il flusso base.

---

## Mini-project (ongoing)

Questo lab serve per imparare il workflow ECS “run + debug” che riuserai nel progetto.

Deliverable:

- sai mostrare il debug flow su ECS: events → stopped reason → logs
- sai dire cosa cambierà quando eseguirai la tua image “hello-api”

---

## Step (numerati)

1) **Crea (o usa) un cluster ECS**
   - ECS → Clusters → Create cluster
   - Nome: `containers-<gruppo>-cluster`

2) **Crea una task definition (Fargate)**
   - ECS → Task definitions → Create
   - Compatibilità: Fargate
   - Container:
     - Image: `public.ecr.aws/docker/library/nginx:alpine`
     - Port mapping: 80
   - Logging: abilita `awslogs` se disponibile nel wizard

3) **Run task** 🎯 *Sfida*
   - Cluster → Tasks → Run new task
   - Launch type: Fargate
   - Networking:
     - Subnet: scegli 2 subnet (se possibile)
     - Public IP: abilita (solo per test rapido)
     - SG: apri 80 solo dal tuo IP (se fattibile) oppure temporaneo
   - *Sfida*: prima di cliccare "Run", annota quante subnet hai scelto e perché.

4) **Verifica stato task**
   - Output atteso: task in `RUNNING`.

5) **Controlla events e stopped reason (se succede)** 🎯 *Sfida*
   - ECS → Task → "Stopped reason"
   - ECS → Cluster/Service → "Events" (se applicabile)
   - *Sfida*: se il task si ferma, trova il motivo esatto prima di chiedere aiuto.

6) **(Opzionale) Controlla log**
   - CloudWatch → Logs → Log groups
   - Cerca log group del task.

7) **(Opzionale) Test via browser**
   - Se hai public IP: `http://<public-ip>`.

---

## Output atteso

- Task ECS avviato e visibile in `RUNNING`.
- Sai trovare events e stopped reason.

## Checkpoint

- Sai dire dove cambiare CPU/memoria della task.
- Sai trovare log group/stream (se `awslogs` configurato).

---

## Troubleshooting rapido

- **Task STOPPED subito**: controlla “Stopped reason” e config porta/command.
- **No logs**: awslogs non configurato o permessi mancanti.
- **Non raggiungo l’IP**: SG/route/public IP errati.

---

## Cleanup obbligatorio

1) ECS → Cluster → Tasks → Stop task
2) ECS → Task definitions → deregister (opzionale) le revisioni create
3) Se hai creato SG: elimina SG (se non serve)
4) Verifica che non restino risorse “in running”

---

## Parole chiave Google (screenshot/guide)

- ECS run task Fargate console screenshot
- ECS task stopped reason screenshot
- ECS create task definition Fargate nginx alpine
- ECS awslogs logConfiguration screenshot
- public IP ECS Fargate reachability troubleshooting

---

## Tutorial consigliati

- [Getting Started with Amazon ECS using Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-fargate.html)
- [ECS Task Definition Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html)
- [ECS Troubleshooting](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/troubleshooting.html)

---

## Soluzioni

<details>
<summary>Sfida Step 3: perché più subnet?</summary>

**Risposta**: scegliere 2+ subnet in Availability Zone diverse garantisce **alta disponibilità**.

Se una AZ ha problemi (es. data center down), Fargate può avviare task nell'altra AZ.

Per un task singolo non è critico, ma per un **Service** con più task è best practice.

</details>

<details>
<summary>Sfida Step 5: interpretare stopped reason</summary>

Motivi comuni e soluzioni:

| Stopped Reason | Causa | Soluzione |
|----------------|-------|-----------|
| `CannotPullContainerError` | Image non trovata o permessi ECR | Verifica nome image, controlla execution role |
| `Essential container exited` | Container crashato | Controlla i log in CloudWatch |
| `ResourceInitializationError` | Problema rete/ENI | Verifica subnet ha IP disponibili, route table corretta |
| `OutOfMemoryError` | Container usa più RAM del limite | Aumenta `memory` nella task definition |

**Passo debug**:

1. Leggi "Stopped reason" nel task
2. Vai in CloudWatch Logs per dettagli
3. Se è errore di rete: controlla SG e subnet

</details>
