# Lab 10 - IAM per ECS: execution role, task role e secrets

## Obiettivo

- Distinguere **execution role** vs **task role**.
- Dare all’app permessi minimi (least privilege).
- Passare un segreto al container tramite **SSM Parameter Store** (SecureString) o Secrets Manager.

## Durata (timebox)

30 minuti.

## Prerequisiti

- ECS/Fargate già visto (Lab 06).
- Permessi su IAM, ECS, SSM Parameter Store (o Secrets Manager).
- Immagine applicativa che legge una variabile d’ambiente (es. `APP_SECRET`).
  > **⚠️ AWS Academy Learner Lab**
  >
  > - **`iam:CreateRole`** (per nomi custom) è **bloccato**. Usa il ruolo pre-creato **`LabRole`** come execution role.
  > - `iam:CreateRole` è permesso solo per il pattern `service-role/*`.
  > - **`ssm:PutParameter`** e **`secretsmanager:Create*`** sono **disponibili** — puoi creare parametri e secret reali.

---

## Mini-project (ongoing)

Comprendi come funziona la secret injection in ECS.

Deliverable:

- crea un parametro SecureString in SSM Parameter Store
- esamina `LabRole` in IAM → Roles: identifica trust policy + permessi allegati
- spiega la differenza tra **execution role** (piattaforma) e **task role** (applicazione)
- nella task definition, configura la sezione **Secrets** (key + value ARN) e verifica l'injection

---

## Step (numerati)

1. **Crea un parametro SecureString (SSM Parameter Store)**
   - Systems Manager ──► Parameter Store ──► Create parameter
   - Name: `/containersaws/lab/app_secret`
   - Type: SecureString
   - Value: (test) `super-secret-value`
   - Clicca **Create parameter**

2. **Verifica execution role ECS**
   - **Nel lab AWS Academy**: usa **`LabRole`** (già pre-creato, include ECR read + CloudWatch Logs).
   - Non creare `ecsTaskExecutionRole` — la lab policy lo blocca.

3. **Usa (o osserva) la task role per l'app** 🎯 _Sfida_
   - ⚠️ **Lab AWS Academy**: `iam:CreateRole` per nomi custom è bloccato, ma è **permesso per `service-role/*`**. Puoi creare un role con quel pattern, oppure usa `LabRole` come esempio da esaminare.
   - Esamina la policy allegata: deve consentire `ssm:GetParameter(s)` solo sul parametro specifico.
   - _Sfida_: leggi la policy JSON e verifica che il Resource punti SOLO al tuo parametro (no wildcard).
   - _Concetto_: in un ambiente reale, saresti tu a creare questo role con `iam:CreateRole`.

4. **Aggiorna la task definition** 🎯 _Sfida_
   - Aggiungi il secret nel container definition:
     - Secrets: env var `APP_SECRET` ──► parametro SSM
   - Imposta:
     - Execution role: **`LabRole`**
     - Task role: ruolo pre-creato (es. `lab-ecs-task-role`)
   - _Sfida_: spiega perché usi `secrets` e non `environment` per valori sensibili.

5. **Redeploy del service**
   - ECS ──► Service ──► Update ──► force new deployment

6. **Verifica nei log**
   - L’app non deve stampare il secret in chiaro.
   - Verifica solo che il secret è “caricato”.

---

## Output atteso

- Parametro SecureString creato.
- Task role con permessi minimi.
- Task ECS che legge il secret tramite integrazione ECS ──► SSM.

## Checkpoint

- Sai spiegare perché l’app usa **task role** e non execution role.
- Sai indicare dove vedi un `AccessDenied`.

---

## Troubleshooting rapido

- **AccessDenied su SSM**: controlla policy della task role (ARN Resource corretto).
- **Secret non iniettato**: assicurati di usare `secrets` (non `environment`).
- **Task non parte**: controlla execution role (ECR/Logs).

---

## Cleanup obbligatorio

1. Elimina il parametro SSM: Systems Manager → Parameter Store → seleziona `/containersaws/lab/app_secret` → **Delete**.
   (Se hai creato un role `service-role/*`: puoi eliminarlo. Non eliminare `LabRole`.)
2. Non eliminare `LabRole` o la task role pre-creata — servono per i lab successivi.

---

## Parole chiave Google (screenshot/guide)

- ECS task execution role vs task role difference
- site:docs.aws.amazon.com ecs secrets parameter store securestring
- IAM policy allow ssm:GetParameters resource ARN example
- ECS AccessDenied ssm troubleshooting

---

## Tutorial consigliati

- [Amazon ECS Task IAM Role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)
- [Specifying Sensitive Data Using SSM](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data-parameters.html)
- [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

---

## Soluzioni

<details>
<summary>🎯 Sfida Step 3: policy JSON least-privilege</summary>

**Policy corretta** (sostituisci account-id e region):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ssm:GetParameter", "ssm:GetParameters"],
      "Resource": "arn:aws:ssm:us-east-1:123456789012:parameter/containersaws/lab/app_secret"
    }
  ]
}
```

**Perché no wildcard**:

- `*` permetterebbe di leggere TUTTI i parametri
- Rischio: altri segreti esposti se l'app viene compromessa
- Best practice: un parametro = una risorsa esplicita

**Tip**: per più parametri sotto un prefix, usa:

```json
"Resource": "arn:aws:ssm:us-east-1:123456789012:parameter/containersaws/lab/*"
```

</details>

<details>
<summary>🎯 Sfida Step 4: secrets vs environment</summary>

**Differenza critica**:

| Campo         | Dove viene salvato               | Visibile in Console | Sicurezza  |
| ------------- | -------------------------------- | ------------------- | ---------- |
| `environment` | Task definition JSON             | Sì (in chiaro)      | ❌ Esposto |
| `secrets`     | Riferimento a SSM/SecretsManager | Solo ARN            | ✅ Sicuro  |

**Cosa succede con `secrets`**:

1. ECS legge il valore da SSM al momento del deploy
2. Il valore viene iniettato nel container come env var
3. La task definition contiene solo l'ARN, non il valore

**Perché è importante**:

- La task definition può essere letta da chiunque abbia accesso ECS
- I log di CloudTrail mostrano la task definition
- Con `secrets`, il valore resta in SSM (criptato con KMS)

</details>
