# Lab 13 — CI/CD: CodeBuild ──► ECR ──► deploy ECS

Riferimento lezione: `slides_deck/lecture_13_cicd_containers_en.md`

## Obiettivo

- Automatizzare build e push di una Docker image su ECR con **CodeBuild**.
- (Opzionale) creare una pipeline con **CodePipeline**.
- Aggiornare ECS service con una nuova versione.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Repository sorgente (GitHub o CodeCommit) con Dockerfile.
- ECR repository esistente (Lab 05).
- ECS service esistente (Lab 06/09).
- Permessi su CodeBuild/CodePipeline/IAM/ECR/ECS.

---

## Mini-project (ongoing)

Automatizza build e deploy del progetto “hello-api”.

Deliverable:

- una build produce un’immagine taggata con `sha-<commit>` (o equivalente) e la push-a in ECR
- fai redeploy del service ECS usando quella nuova versione (task definition revision)
- sai rispondere: “quale commit sta girando?” guardando tag/revision

---

## Step (numerati)

1) **Prepara repository sorgente**
   - Deve contenere `Dockerfile` e codice app

2) **Crea `buildspec.yml` (minimo)** 🎯 *Sfida*
   - Login su ECR
   - Build immagine
   - Tag con `latest` + `commit-sha` (o timestamp)
   - Push su ECR
   - *Sfida*: scrivi un buildspec.yml funzionante che tagga con `$CODEBUILD_RESOLVED_SOURCE_VERSION`.

3) **Crea o usa un progetto CodeBuild**
   - Abilita *privileged mode* per Docker build

4) **Esegui build** 🎯 *Sfida*
   - Start build
   - Verifica log e fase di push
   - *Sfida*: se la build fallisce, trova l'errore esatto nei log (quale fase? quale comando?).

5) **Verifica immagine su ECR**
   - Deve comparire un nuovo tag/digest

6) **Deploy su ECS (manuale nel lab)**
   - Aggiorna task definition con nuovo tag
   - ECS ──► Service ──► Update ──► force new deployment

---

## Output atteso

- Build automatizzata e push su ECR completato.
- ECS aggiornato e nuovo deployment riuscito.

## Checkpoint

- Sai spiegare perché taggare anche con **commit SHA** aumenta tracciabilità.
- Sai trovare il motivo di un build failure nei log CodeBuild.

---

## Troubleshooting rapido

- **Build Docker fallisce**: controlla `privileged mode`.
- **AccessDenied su ECR**: controlla role CodeBuild e policy.
- **Deploy non aggiorna**: task definition punta al tag corretto? forza deployment.

---

## Cleanup obbligatorio

- Elimina pipeline/progetto CodeBuild se creati solo per il lab.
- In ECR elimina immagini di test non necessarie.

---

## Parole chiave Google (screenshot/guide)

- AWS CodeBuild docker privileged mode screenshot
- CodeBuild buildspec.yml docker build push ECR example
- IAM policy for CodeBuild push to ECR
- CodePipeline ECS deploy action screenshot

---

## Tutorial consigliati

- [AWS CodeBuild Getting Started](https://docs.aws.amazon.com/codebuild/latest/userguide/getting-started.html)
- [Build Spec Reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)
- [CodePipeline with ECS](https://docs.aws.amazon.com/codepipeline/latest/userguide/ecs-cd-pipeline.html)

---

## Soluzioni

<details>
<summary>Sfida Step 2: buildspec.yml completo</summary>

**buildspec.yml funzionante**:

```yaml
version: 0.2

env:
  variables:
    AWS_DEFAULT_REGION: "eu-west-1"
    IMAGE_REPO_NAME: "hello-api"
    AWS_ACCOUNT_ID: "123456789012"  # Sostituisci

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}
  build:
    commands:
      - echo Build started on `date`
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest
  post_build:
    commands:
      - echo Build completed on `date`
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:latest
```

**Variabile chiave**: `$CODEBUILD_RESOLVED_SOURCE_VERSION` contiene il commit SHA completo.

</details>

<details>
<summary>Sfida Step 4: diagnosticare build failure</summary>

**Dove trovare l'errore**:

1. CodeBuild ──► Build history ──► [build fallita]
2. Apri "Build logs" (o "Tail logs")
3. Cerca la **fase** che ha fallito (PRE_BUILD, BUILD, POST_BUILD)
4. Leggi l'output del comando fallito

**Errori comuni**:

| Fase | Errore | Causa | Soluzione |
|------|--------|-------|-----------|
| PRE_BUILD | `denied: Your authorization token has expired` | Token ECR scaduto | Riesegui login |
| BUILD | `Cannot connect to Docker daemon` | Privileged mode non abilitato | Abilita in project settings |
| BUILD | `COPY failed: file not found` | Path errato nel Dockerfile | Verifica struttura cartelle |
| POST_BUILD | `AccessDeniedException` | IAM policy mancante | Aggiungi ecr:PutImage alla role |

**Tip**: la fase fallita è indicata con `[Container] Phase FAILED`.

</details>
