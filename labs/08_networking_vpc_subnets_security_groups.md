# Lab 08 — Networking design: VPC, subnet, Security Groups (ECS/Fargate)

Riferimento lezione: `slides_deck/lecture_08_ecs_networking_basics_en.md`

## Obiettivo

- Progettare una rete minima per ECS Fargate.
- Definire regole SG corrette: **ALB → Task**.
- Capire quando servono **NAT** o **VPC endpoints**.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Accesso alla Console AWS.
- Conoscenze base: VPC, subnet, route table.

---

## Mini-project (ongoing)

Progetta il networking del progetto “hello-api” (ALB + ECS tasks) in modo sicuro e operabile.

Deliverable:

- scegli (e motiva) subnet pubbliche per ALB e subnet private per i task
- definisci SG-ALB e SG-TASK con regole minime (ALB → task sulla porta app)
- spiega come i task in subnet private raggiungono ECR/CloudWatch (NAT oppure VPC endpoints)

## Scenario

Esercizio guidato: scegliamo una VPC esistente e progettiamo come metterci ALB + tasks.

---

## Step (numerati)

1) **Identifica la VPC**
   - VPC → Your VPCs
   - Nota: CIDR, numero AZ.

2) **Identifica subnet pubbliche e private**
   - VPC → Subnets
   - Per ogni subnet annota: AZ, route table, presenza route verso IGW.

3) **Disegna lo schema (anche su foglio)**
   - ALB in subnet pubbliche (2 AZ)
   - Tasks ECS in subnet private (2 AZ)

4) **Security Groups: regole minime**
   - SG-ALB inbound: 80/443 da Internet (o dal tuo IP)
   - SG-TASK inbound: porta app **solo da SG-ALB**

5) **Discussione rapida: accesso a ECR/CloudWatch da subnet private**
   - Opzione A: NAT Gateway
   - Opzione B: VPC endpoints (ECR/Logs)
   - Nota: pro/contro costi.

---

## Output atteso

- Schema di rete minimo definito.
- Regole SG scritte correttamente.

## Checkpoint

- Sai dire quando una subnet è pubblica (route `0.0.0.0/0` → IGW).
- Sai spiegare perché non aprire la porta del task a `0.0.0.0/0`.

---

## Troubleshooting rapido

- **Non capisco se subnet è pubblica**: controlla route table → route `0.0.0.0/0` verso IGW.
- **Traffico ALB→task non passa**: spesso è SG-TASK inbound o target group port.

---

## Cleanup obbligatorio

- Nessuna risorsa creata (se hai creato SG per prova, eliminalo).

---

## Parole chiave Google (screenshot/guide)

- AWS VPC public vs private subnet route table IGW
- security group source another security group example
- ECS Fargate awsvpc ENI security group
- VPC endpoints ECR CloudWatch Logs private subnet
- NAT gateway cost per hour warning
