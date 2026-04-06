# Lab 03 - AWS setup: Region, identity, console tour (per container services)

## Obiettivo

- Impostare correttamente **Region** e contesto di lavoro del corso.
- Capire dove trovare (e leggere) le pagine principali: **ECS, EKS, ECR, CloudWatch, IAM, VPC**.
- Preparare una base minima di “igiene cloud”: naming/tagging e controlli costi (per operare e scegliere i servizi corretti).

## Durata (timebox)

30 minuti.

## Prerequisiti

- Account AWS (Free Tier o didattico) con accesso in Console.
- Docker installato sulla propria macchina.
- AWS CLI v2 installata (vedi Step 0 sotto).

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

### 0. Installa AWS CLI v2 (una sola volta)

La CLI serve per interagire con AWS da terminale. Installala sulla tua macchina locale.

#### Ubuntu / Debian

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip
unzip -q /tmp/awscliv2.zip -d /tmp
sudo /tmp/aws/install
aws --version
```

#### macOS

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o /tmp/AWSCLIV2.pkg
sudo installer -pkg /tmp/AWSCLIV2.pkg -target /
aws --version
```

#### Windows

1. Scarica l'installer: <https://awscli.amazonaws.com/AWSCLIV2.msi>
2. Esegui il `.msi` e segui il wizard (Next → Next → Install)
3. Apri un nuovo terminale (cmd o PowerShell) e verifica:

```powershell
aws --version
```

> **Verifica**: il comando `aws --version` deve restituire `aws-cli/2.x.x ...`.
> Se il comando non viene trovato, chiudi e riapri il terminale.

---

### 0b. Configura le credenziali Learner Lab

Ogni volta che avvii (o riavvii) il Learner Lab, devi copiare le credenziali temporanee.

1. Apri il **Learner Lab** nel browser
2. Clicca **AWS Details** (in alto a destra nella pagina del lab)
3. Clicca **Show** accanto a **AWS CLI**
4. Vedrai tre righe:

```
[default]
aws_access_key_id = ASIA...
aws_secret_access_key = ...
aws_session_token = ...
```

5. **Copia** le tre righe (tutto il blocco da `[default]` fino alla fine del token).

6. **Incolla nel file credentials:**

#### Linux / macOS

```bash
mkdir -p ~/.aws
nano ~/.aws/credentials
```

Incolla le tre righe, poi **salva**: `Ctrl+O`, `Invio`, `Ctrl+X`.

#### Windows

Apri Notepad e incolla. Salva in: `C:\Users\<TuoNome>\.aws\credentials`

7. Imposta la Region di default:

```bash
aws configure set region us-east-1
```

8. Verifica che la connessione funzioni:

```bash
aws sts get-caller-identity
```

> **Output atteso**: Account ID, UserId, Arn del tuo ruolo Learner Lab.
>
> **⚠️ Le credenziali scadono ogni poche ore.** Se ricevi `ExpiredToken`, ripeti dal passo 2.

---

1. **Seleziona la Region del corso**
   - Console AWS ──► selettore Region (in alto a destra)
   - Nota: useremo una sola Region per tutto il corso.

2. **Verifica l’identità attiva**
   - In alto a destra ──► “Security credentials” / “Account”
   - Output atteso: sai dire _chi_ sei (utente/ruolo) e in che account sei.

3. **Definisci convenzioni minime**
   - Tag consigliati:
     - `Project=ContainersAWS`
     - `Owner=<nome_gruppo>`
   - Naming: prefisso `containers-<gruppo>-...`

4. **Console tour: ECS** 🎯 _Sfida_
   - Apri ECS ──► guarda "Clusters", "Task definitions", "Services".
   - Obiettivo: trovare Events e "stopped reason".
   - _Sfida_: trova dove puoi vedere il motivo per cui un task è stato fermato.

5. **Console tour: ECR**
   - Apri ECR ──► "Repositories".
   - Obiettivo: capire dove vedi tag/digest e scan.

6. **Console tour: CloudWatch**
   - Apri CloudWatch ──► "Logs", "Log groups", "Alarms".
   - Obiettivo: capire dove leggerai i log ECS.

7. **Console tour: IAM (solo lettura)** 🎯 _Sfida_
   - Apri IAM ──► "Roles".
   - Obiettivo: differenza concettuale tra execution role e task role.
   - _Sfida_: cerca un ruolo che contiene "ecsTaskExecution" nel nome e leggi le policy attached.

8. **(Opzionale, se consentito) Controlli costi**
   - Billing ──► Cost Explorer / Budgets.
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
<summary>🎯 Sfida Step 4: dove vedere "stopped reason"</summary>

1. Vai in **ECS ──► Clusters ──► [tuo cluster] ──► Tasks**
2. Clicca su un task con stato **STOPPED**
3. In alto vedrai **"Stopped reason"** con la spiegazione (es. "Essential container exited", "CannotPullContainerError", ecc.)
4. Nella tab **Events** del service (se presente) trovi la cronologia degli eventi

</details>

<details>
<summary>🎯 Sfida Step 7: trovare LabRole (execution role)</summary>

1. Vai in **IAM ──► Roles**
2. Cerca `LabRole` nella barra di ricerca
3. Clicca su **LabRole**
4. Vedrai le policy attached (es. `AmazonEC2ContainerRegistryReadOnly` + managed policies del lab)
5. Differenza chiave:
   - **Execution role**: permessi per ECS agent (pull image, push logs)
   - **Task role**: permessi per il tuo codice applicativo (es. accesso S3, DynamoDB)

</details>
