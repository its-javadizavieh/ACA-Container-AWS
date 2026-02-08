# Lab 03 — AWS setup: Region, identity, console tour (per container services)

Riferimento lezione: `slides_deck/lecture_03_aws_containers_overview_en.md`

## Obiettivo

- Impostare correttamente **Region** e contesto di lavoro del corso.
- Capire dove trovare (e leggere) le pagine principali: **ECS, EKS, ECR, CloudWatch, IAM, VPC**.
- Preparare una base minima di “igiene cloud”: naming/tagging e controlli costi (per operare e scegliere i servizi corretti).

## Durata (timebox)

30 minuti.

## Prerequisiti

- Account AWS (Free Tier o didattico) con accesso in Console.
- (Opzionale) AWS CLI già configurata.

## Scenario

Prima di creare risorse container, uniformiamo setup e convenzioni e sappiamo “dove guardare” quando qualcosa va storto.

---

## Mini-project (ongoing)

Imposta il contesto operativo del progetto su AWS.

Deliverable:

- scegli e annota la singola AWS Region del progetto
- definisci naming/tagging per tutte le risorse del progetto
- sai dove trovare: ECS Events, task stopped reason, CloudWatch Logs, ECR Images

---

## Step (numerati)

1) **Seleziona la Region del corso**
   - Console AWS → selettore Region (in alto a destra)
   - Nota: useremo una sola Region per tutto il corso.

2) **Verifica l’identità attiva**
   - In alto a destra → “Security credentials” / “Account”
   - Output atteso: sai dire *chi* sei (utente/ruolo) e in che account sei.

3) **Definisci convenzioni minime**
   - Tag consigliati:
     - `Project=ContainersAWS`
     - `Owner=<nome_gruppo>`
   - Naming: prefisso `containers-<gruppo>-...`

4) **Console tour: ECS** 🎯 *Sfida*
   - Apri ECS → guarda "Clusters", "Task definitions", "Services".
   - Obiettivo: trovare Events e "stopped reason".
   - *Sfida*: trova dove puoi vedere il motivo per cui un task è stato fermato.

5) **Console tour: ECR**
   - Apri ECR → "Repositories".
   - Obiettivo: capire dove vedi tag/digest e scan.

6) **Console tour: CloudWatch**
   - Apri CloudWatch → "Logs", "Log groups", "Alarms".
   - Obiettivo: capire dove leggerai i log ECS.

7) **Console tour: IAM (solo lettura)** 🎯 *Sfida*
   - Apri IAM → "Roles".
   - Obiettivo: differenza concettuale tra execution role e task role.
   - *Sfida*: cerca un ruolo che contiene "ecsTaskExecution" nel nome e leggi le policy attached.

8) **(Opzionale, se consentito) Controlli costi**
   - Billing → Cost Explorer / Budgets.
   - Se non hai permessi: annota la limitazione.

---

## Output atteso

- Region selezionata e annotata.
- Convenzioni di tag/naming definite.
- Familiarità con pagine principali per troubleshooting.

## Checkpoint

- Sai indicare in Console dove guardare:
  - eventi del service ECS
  - log in CloudWatch
  - immagini in ECR

---

## Troubleshooting rapido

- **Non vedo Billing**: permessi limitati (normale in account didattici).
- **Servizi non visibili**: verifica Region corretta.

---

## Cleanup obbligatorio

- Nessuna risorsa creata (lab principalmente di consultazione).

---

## Parole chiave Google (screenshot/guide)

- AWS console region selector screenshot
- Amazon ECS clusters task definitions services screenshot
- Amazon ECR repositories images tags digest screenshot
- CloudWatch log groups ECS awslogs screenshot
- IAM role vs user console screenshot

---

## Tutorial consigliati

- [AWS Console Getting Started](https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/learn-whats-new.html)
- [Amazon ECS Concepts](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [CloudWatch Logs Concepts](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)

---

## Soluzioni

<details>
<summary>Sfida Step 4: dove vedere "stopped reason"</summary>

1. Vai in **ECS → Clusters → [tuo cluster] → Tasks**
2. Clicca su un task con stato **STOPPED**
3. In alto vedrai **"Stopped reason"** con la spiegazione (es. "Essential container exited", "CannotPullContainerError", ecc.)
4. Nella tab **Events** del service (se presente) trovi la cronologia degli eventi

</details>

<details>
<summary>Sfida Step 7: trovare ecsTaskExecutionRole</summary>

1. Vai in **IAM → Roles**
2. Cerca `ecsTaskExecution` nella barra di ricerca
3. Clicca su **AmazonECSTaskExecutionRolePolicy**
4. Vedrai le policy attached:
   - `AmazonECSTaskExecutionRolePolicy` — permette pull da ECR e scrittura log CloudWatch
5. Differenza chiave:
   - **Execution role**: permessi per ECS agent (pull image, push logs)
   - **Task role**: permessi per il tuo codice applicativo (es. accesso S3, DynamoDB)

</details>
