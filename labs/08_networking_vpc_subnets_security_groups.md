# Lab 08 - Networking design: VPC, subnet, Security Groups (ECS/Fargate)

## Obiettivo

- Progettare una rete minima per ECS Fargate.
- Definire regole SG corrette: **ALB ──► Task**.
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
- definisci SG-ALB e SG-TASK con regole minime (ALB ──► task sulla porta app)
- spiega come i task in subnet private raggiungono ECR/CloudWatch (NAT oppure VPC endpoints)

## Scenario

Esercizio guidato: scegliamo una VPC esistente e progettiamo come metterci ALB + tasks.

---

## Step (numerati)

1. **Identifica la VPC**
   - VPC ──► Your VPCs
   - Nota: CIDR, numero AZ.

2. **Identifica subnet pubbliche e private**
   - VPC ──► Subnets
   - Per ogni subnet annota: AZ, route table, presenza route verso IGW.

3. **Disegna lo schema (anche su foglio)**
   - ALB in subnet pubbliche (2 AZ)
   - Tasks ECS in subnet private (2 AZ)

4. **Security Group: crea SG-ALB** 🎯 _Sfida_
   - Console → **EC2** → menu a sinistra **Security Groups** → **Create security group**

   **Basic details:**
   - Security group name: `SG-ALB`
   - Description: `Allow HTTP from internet`
   - VPC: seleziona la default VPC

   **Inbound rules** → **Add rule**:

   | Type | Protocol | Port range | Source | Description |
   |------|----------|-----------|--------|-------------|
   | HTTP | TCP | 80 | Custom → `0.0.0.0/0` | HTTP from internet |

   **Outbound rules**: lascia il default (All traffic → 0.0.0.0/0).

   → **Create security group**. Annota l'ID `sg-xxx`.

5. **Security Group: crea SG-TASK** 🎯 _Sfida_
   - Ancora in **Security Groups** → **Create security group**

   **Basic details:**
   - Security group name: `SG-TASK`
   - Description: `Allow SG Task`
   - VPC: stessa default VPC

   **Inbound rules** → **Add rule**:

   | Type | Protocol | Port range | Source | Description |
   |------|----------|-----------|--------|-------------|
   | Custom TCP | TCP | 9090 | Custom → scrivi `SG-ALB` e selezionalo dal dropdown | Only ALB can reach task |

   **Outbound rules**: lascia il default (All traffic → 0.0.0.0/0).

   → **Create security group**.

   - _Sfida_: perché la Source è un security group e non un CIDR come `0.0.0.0/0`? Scrivi la risposta.

6. **Discussione rapida: accesso a ECR/CloudWatch da subnet private** 🎯 _Sfida_
   - Opzione A: NAT Gateway
   - Opzione B: VPC endpoints (ECR/Logs)
   - Nota: pro/contro costi.
   - _Sfida_: calcola il costo mensile approssimativo di un NAT Gateway (hint: cerca il prezzo per ora + GB).

---

## Output atteso

- Schema di rete minimo definito.
- `SG-ALB` creato con inbound HTTP 80 da 0.0.0.0/0.
- `SG-TASK` creato con inbound Custom TCP 9090 da **SG-ALB**.

## Checkpoint

- Sai dire quando una subnet è pubblica (route `0.0.0.0/0` ──► IGW).
- Sai spiegare perché la Source di SG-TASK è un security group e non un CIDR.
- Sai l'ordine di cancellazione: prima SG-TASK (che referenzia SG-ALB), poi SG-ALB.

---

## Troubleshooting rapido

- **Non capisco se subnet è pubblica**: controlla route table ──► route `0.0.0.0/0` verso IGW.
- **Traffico ALB──►task non passa**: spesso è SG-TASK inbound o target group port.

---

## Cleanup obbligatorio

- EC2 → **Security Groups** → seleziona `SG-TASK` → **Actions** → **Delete security groups** (prima SG-TASK, poi SG-ALB).

---

## Parole chiave Google (screenshot/guide)

- AWS VPC public vs private subnet route table IGW
- security group source another security group example
- ECS Fargate awsvpc ENI security group
- VPC endpoints ECR CloudWatch Logs private subnet
- NAT gateway cost per hour warning

---

## Tutorial consigliati

- [AWS VPC User Guide - How It Works](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html)
- [Security Groups for Your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [VPC Endpoints for AWS Services](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)


