# Lab 09 - ECS + ALB: Target Group, health checks e Security Groups

## Obiettivo

- Pubblicare un service ECS dietro un **Application Load Balancer (ALB)**.
- Configurare **Target Group (type IP)** per Fargate.
- Impostare health checks e **Security Groups** corretti.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Service ECS esistente (da Lab 06) oppure task definition + immagine ECR disponibili.
- ALB + Target Group: gli studenti possono crearli direttamente (`elasticloadbalancing:Create*` è permesso dalla lab policy).
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

1. **Crea/usa Security Groups**
   - SG-ALB: inbound `80` da `0.0.0.0/0` (solo HTTP per lab)
   - SG-TASK: inbound porta app (es. `8080`) **solo da SG-ALB**

2. **Crea/usa ALB + Target Group** 🎯 _Sfida_
   - Target type: **IP** (per Fargate)
   - Health check path: `/health` (per "hello-api", oppure il path della tua app)
   - _Sfida_: configura health check con timeout 5s, interval 10s, unhealthy threshold 2.

3. **Aggiorna o crea ECS Service con Load Balancer**
   - ECS ──► Cluster ──► Service ──► Create/Update
   - Load balancing:
     - ALB selezionato
     - Target group selezionato
     - Container/Port: porta corretta (es. `8080`)
   - Networking:
     - Security group task: SG-TASK

4. **Verifica Target health** 🎯 _Sfida_
   - EC2 ──► Target Groups ──► Targets ──► devono diventare "healthy"
   - _Sfida_: se un target è "unhealthy", trova il motivo esatto (health check response code).

5. **Test**
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

1. ECS: elimina il service creato per il lab (o rimuovi integrazione ALB se richiesto).
2. Se ALB/TG sono condivisi di classe: **non eliminare**.
3. Se hai creato ALB/TG/SG dedicati: eliminali.

---

## Parole chiave Google (screenshot/guide)

- AWS ALB create application load balancer console screenshot
- ECS service attach load balancer target group screenshot
- Fargate target group type IP health check
- Security group allow traffic from another security group
- ALB targets unhealthy troubleshooting

---

## Tutorial consigliati

- [Creating an Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-application-load-balancer.html)
- [Register Targets with Your Target Group](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-register-targets.html)
- [Health Checks for Your Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)

---

## Soluzioni

<details>
<summary>🎯 Sfida Step 2: health check configuration</summary>

**Impostazioni consigliate**:

| Parametro           | Valore     | Perché                                              |
| ------------------- | ---------- | --------------------------------------------------- |
| Path                | `/health`  | Endpoint dedicato (non `/` che potrebbe fare altro) |
| Protocol            | HTTP       | Sufficiente per health check interni                |
| Timeout             | 5 secondi  | Tempo massimo per risposta                          |
| Interval            | 10 secondi | Frequenza di controllo                              |
| Healthy threshold   | 2          | 2 check OK ──► target healthy                       |
| Unhealthy threshold | 2          | 2 check FAIL ──► target unhealthy                   |

**Dove configurare**: EC2 ──► Target Groups ──► [tuo TG] ──► Health checks ──► Edit

**Best practice**: l'endpoint `/health` dovrebbe essere leggero (no DB query pesanti).

</details>

<details>
<summary>🎯 Sfida Step 4: diagnosticare target unhealthy</summary>

**Dove trovare il motivo**:

1. EC2 ──► Target Groups ──► [tuo TG] ──► Targets
2. Clicca sul target unhealthy
3. Leggi **"Health status details"**

**Codici comuni e cause**:

| Status                             | Causa                                | Soluzione                            |
| ---------------------------------- | ------------------------------------ | ------------------------------------ |
| `Unhealthy: Request timed out`     | App non risponde in tempo            | Aumenta timeout o ottimizza endpoint |
| `Unhealthy: Target is in DRAINING` | Task in chiusura                     | Aspetta o verifica deployment        |
| `Unhealthy: 404`                   | Path health check errato             | Correggi path nel TG                 |
| `Unhealthy: 503`                   | App non pronta                       | Controlla log container              |
| `Connection refused`               | Porta sbagliata o app non in ascolto | Verifica port mapping                |

**Tip**: controlla anche SG-TASK - deve permettere traffico dalla porta health check.

</details>
