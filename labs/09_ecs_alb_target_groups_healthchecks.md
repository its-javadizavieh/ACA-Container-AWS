# Lab 09 — ECS + ALB: Target Group, health checks e Security Groups

Riferimento lezione: `slides_deck/lecture_09_alb_healthchecks_en.md`

## Obiettivo

- Pubblicare un service ECS dietro un **Application Load Balancer (ALB)**.
- Configurare **Target Group (type IP)** per Fargate.
- Impostare health checks e **Security Groups** corretti.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Service ECS esistente (da Lab 06) oppure task definition + immagine ECR disponibili.
- (Consigliato) ALB + Target Group già predisposti dal docente per rispettare il timebox.
- Permessi su: EC2 (ALB/SG), ECS, IAM.

---

## Mini-project (ongoing)

Metti online “hello-api” dietro ALB con health check corretto.

Deliverable:

- ECS Service dietro un ALB con target group **type IP**
- health check su `/health` (endpoint del progetto)
- endpoint raggiungibili via ALB DNS (es. `/` e `/version`)

---

## Step (numerati)

1) **Crea/usa Security Groups**
   - SG-ALB: inbound `80` da `0.0.0.0/0` (solo HTTP per lab)
   - SG-TASK: inbound porta app (es. `8080`) **solo da SG-ALB**

2) **Crea/usa ALB + Target Group**
   - Target type: **IP** (per Fargate)
   - Health check path: `/health` (per “hello-api”, oppure il path della tua app)

3) **Aggiorna o crea ECS Service con Load Balancer**
   - ECS → Cluster → Service → Create/Update
   - Load balancing:
     - ALB selezionato
     - Target group selezionato
     - Container/Port: porta corretta (es. `8080`)
   - Networking:
     - Security group task: SG-TASK

4) **Verifica Target health**
   - EC2 → Target Groups → Targets → devono diventare “healthy”

5) **Test**
   - Apri DNS name dell’ALB

---

## Output atteso

- ALB con listener `80`.
- Target group con target “healthy”.
- App raggiungibile tramite DNS ALB.

## Checkpoint

- Sai spiegare differenza tra SG ALB e SG task.
- Sai spiegare perché target type deve essere IP per Fargate.

---

## Troubleshooting rapido

- **Targets unhealthy**: controlla health check path/port, SG-TASK inbound, container port.
- **Timeout dal browser**: verifica subnet pubbliche e route a IGW, listener e target group.

---

## Cleanup obbligatorio

1) ECS: elimina il service creato per il lab (o rimuovi integrazione ALB se richiesto).
2) Se ALB/TG sono condivisi di classe: **non eliminare**.
3) Se hai creato ALB/TG/SG dedicati: eliminali.

---

## Parole chiave Google (screenshot/guide)

- AWS ALB create application load balancer console screenshot
- ECS service attach load balancer target group screenshot
- Fargate target group type IP health check
- Security group allow traffic from another security group
- ALB targets unhealthy troubleshooting
