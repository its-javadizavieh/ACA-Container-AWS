# Lab 13 — CI/CD: CodeBuild ──► ECR ──► deploy ECS

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

> **⚠️ AWS Academy Lab Environment**
>
> Nel lab _Microservices and CI/CD Pipeline Builder_:
>
> - **`codebuild:CreateProject`** non è incluso nella lab policy.
> - `codebuild:CreateProject` è **bloccato** — non è possibile creare né usare progetti CodeBuild in questo lab.
> - Lo step CodeBuild è un **walkthrough concettuale**: mostra la schermata di creazione, spiega i campi, ma non cliccare Create.
> - In alternativa: build e push manuale da Cloud9/terminale (stessi comandi del buildspec.yml).

---

## Mini-project (ongoing)

Automatizza build e deploy del progetto “hello-api”.

Deliverable:

- scrivi un `buildspec.yml` funzionante (anche se non puoi eseguirlo in CodeBuild)
- build manuale da terminale: `docker build` + `docker tag sha-<commit>` + `docker push` su ECR
- fai redeploy del service ECS usando la nuova immagine (task definition revision)
- sai rispondere: "quale commit sta girando?" guardando tag/revision
- ⚠️ `codebuild:CreateProject` bloccato — walkthrough concettuale della schermata CodeBuild

---

## Step (numerati)

1. **Prepara repository sorgente**
   - Deve contenere `Dockerfile` e codice app

2. **Crea `buildspec.yml` (minimo)** 🎯 _Sfida_
   - Login su ECR
   - Build immagine
   - Tag con `latest` + `commit-sha` (o timestamp)
   - Push su ECR
   - _Sfida_: scrivi un buildspec.yml funzionante che tagga con `$CODEBUILD_RESOLVED_SOURCE_VERSION`.

3. **Usa il progetto CodeBuild pre-creato**
   - ⚠️ **Lab AWS Academy**: `codebuild:CreateProject` è bloccato. Questo step è un **walkthrough concettuale**: apri CodeBuild ──► Create build project, mostra i campi, ma non cliccare Create. In alternativa: esegui build e push manualmente da terminale.
   - Verifica che _privileged mode_ sia abilitato (necessario per Docker build)
   - Esamina la configurazione: source, environment, service role

4. **Esegui build** 🎯 _Sfida_
   - Start build
   - Verifica log e fase di push
   - _Sfida_: se la build fallisce, trova l'errore esatto nei log (quale fase? quale comando?).

5. **Verifica immagine su ECR**
   - Deve comparire un nuovo tag/digest

6. **Deploy su ECS (manuale nel lab)**
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

- ⚠️ **Lab AWS Academy**: non eliminare il progetto CodeBuild pre-creato — potrebbe servire ad altri studenti.
- In ECR elimina immagini di test non necessarie.
- Se hai creato pipeline CodePipeline: disabilita o elimina.

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
<summary>🎯 Sfida Step 2: buildspec.yml completo</summary>

**buildspec.yml funzionante**:

```yaml
version: 0.2

env:
  variables:
    AWS_DEFAULT_REGION: "eu-west-1"
    IMAGE_REPO_NAME: "hello-api"
    AWS_ACCOUNT_ID: "123456789012" # Sostituisci

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
<summary>🎯 Sfida Step 4: diagnosticare build failure</summary>

**Dove trovare l'errore**:

1. CodeBuild ──► Build history ──► [build fallita]
2. Apri "Build logs" (o "Tail logs")
3. Cerca la **fase** che ha fallito (PRE_BUILD, BUILD, POST_BUILD)
4. Leggi l'output del comando fallito

**Errori comuni**:

| Fase       | Errore                                         | Causa                         | Soluzione                       |
| ---------- | ---------------------------------------------- | ----------------------------- | ------------------------------- |
| PRE_BUILD  | `denied: Your authorization token has expired` | Token ECR scaduto             | Riesegui login                  |
| BUILD      | `Cannot connect to Docker daemon`              | Privileged mode non abilitato | Abilita in project settings     |
| BUILD      | `COPY failed: file not found`                  | Path errato nel Dockerfile    | Verifica struttura cartelle     |
| POST_BUILD | `AccessDeniedException`                        | IAM policy mancante           | Aggiungi ecr:PutImage alla role |

**Tip**: la fase fallita è indicata con `[Container] Phase FAILED`.

</details>
