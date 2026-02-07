# Lab 10 — IAM per ECS: execution role, task role e secrets

Riferimento lezione: `slides_deck/lecture_10_iam_for_ecs_secrets_en.md`

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

---

## Mini-project (ongoing)

Aggiungi secret injection al progetto “hello-api” nel modo corretto.

Deliverable:

- crea un parametro SecureString (o un secret) di test
- crea una **task role** least-privilege che può leggere **solo** quel parametro/secret
- inietta il secret nel container via integrazione ECS (non hard-coded)
- verifica dai log che l’app non stampa il segreto in chiaro

---

## Step (numerati)

1) **Crea un parametro SecureString (SSM Parameter Store)**
   - Systems Manager → Parameter Store → Create parameter
   - Name: `/containersaws/lab/app_secret`
   - Type: SecureString
   - Value: (test) `super-secret-value`

2) **Verifica/crea execution role ECS**
   - Tipico: `ecsTaskExecutionRole`
   - Deve poter leggere da ECR e scrivere log CloudWatch.

3) **Crea una task role per l’app**
   - IAM → Roles → Create role → ECS Task
   - Policy minima: consenti `ssm:GetParameter(s)` solo sul parametro creato.

4) **Aggiorna la task definition**
   - Aggiungi il secret nel container definition:
     - Secrets: env var `APP_SECRET` → parametro SSM
   - Imposta:
     - Execution role: `ecsTaskExecutionRole`
     - Task role: ruolo creato

5) **Redeploy del service**
   - ECS → Service → Update → force new deployment

6) **Verifica nei log**
   - L’app non deve stampare il secret in chiaro.
   - Verifica solo che il secret è “caricato”.

---

## Output atteso

- Parametro SecureString creato.
- Task role con permessi minimi.
- Task ECS che legge il secret tramite integrazione ECS → SSM.

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

1) Elimina il parametro `/containersaws/lab/app_secret`.
2) Elimina la role del lab (se non riutilizzata).

---

## Parole chiave Google (screenshot/guide)

- ECS task execution role vs task role difference
- site:docs.aws.amazon.com ecs secrets parameter store securestring
- IAM policy allow ssm:GetParameters resource ARN example
- ECS AccessDenied ssm troubleshooting
