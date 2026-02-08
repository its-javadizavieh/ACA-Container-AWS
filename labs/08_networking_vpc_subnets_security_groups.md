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

4) **Security Groups: regole minime** 🎯 *Sfida*
   - SG-ALB inbound: 80/443 da Internet (o dal tuo IP)
   - SG-TASK inbound: porta app **solo da SG-ALB**
   - *Sfida*: scrivi le regole esatte per SG-TASK (source = sg-xxx, non un CIDR).

5) **Discussione rapida: accesso a ECR/CloudWatch da subnet private** 🎯 *Sfida*
   - Opzione A: NAT Gateway
   - Opzione B: VPC endpoints (ECR/Logs)
   - Nota: pro/contro costi.
   - *Sfida*: calcola il costo mensile approssimativo di un NAT Gateway (hint: cerca il prezzo per ora + GB).

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

---

## Tutorial consigliati

- [AWS VPC User Guide — How It Works](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html)
- [Security Groups for Your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [VPC Endpoints for AWS Services](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)

---

## Soluzioni

<details>
<summary>Sfida Step 4: regole SG-TASK esatte</summary>

**SG-ALB** (per Application Load Balancer):

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| HTTP | TCP | 80 | 0.0.0.0/0 (o tuo IP) |
| HTTPS | TCP | 443 | 0.0.0.0/0 (o tuo IP) |

**SG-TASK** (per container ECS):

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| Custom TCP | TCP | 8080 | **sg-alb** |

**Perché usare SG come source**:

- Più sicuro: solo traffico che passa dall'ALB può raggiungere i task
- Più dinamico: se aggiungi/rimuovi ALB instances, le regole restano valide
- Best practice AWS per architetture multi-tier

</details>

<details>
<summary>Sfida Step 5: costo NAT Gateway</summary>

**Pricing NAT Gateway** (eu-west-1, Marzo 2025 circa):

- **Per ora**: ~$0.045/h → ~$32/mese (24/7)
- **Per GB processato**: ~$0.045/GB

**Esempio calcolo** (ambiente dev con 50 GB/mese):

- Costo orario: $0.045 × 730h = **$32.85**
- Costo dati: $0.045 × 50GB = **$2.25**
- **Totale: ~$35/mese** per un solo NAT Gateway

**Alternativa VPC Endpoints**:

- $0.01/h per endpoint → ~$7/mese
- Nessun costo per GB (per traffico AWS)
- Servono 3-4 endpoints (ECR.api, ECR.dkr, Logs, S3)
- **Totale: ~$28/mese** ma più complesso da configurare

**Consiglio**: per dev/test, usa NAT. Per produzione con molto traffico, valuta endpoints.

</details>
