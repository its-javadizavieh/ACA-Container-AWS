# Lab 13 - CI/CD: build, push e deploy

## Obiettivo

- Preparare il flusso minimo `build -> push -> deploy`.
- Taggare l'immagine con un riferimento tracciabile.
- Aggiornare ECS con una nuova versione.

## Durata

30 minuti.

## Prerequisiti

- Repository sorgente con `Dockerfile`.
- Repository ECR `demo-hello-api` disponibile.
- Service ECS attivo.

## Guida del lab

1. **Prepara `buildspec.yml`**
   - Crea un file con tre fasi: `pre_build`, `build`, `post_build`.
   - Deve fare:
     - login su ECR
     - build dell'immagine
     - tag con `latest` e con `commit-sha`
     - push dei tag su ECR

2. **Walkthrough CodeBuild**
   - CodeBuild -> `Create build project`
   - Controlla questi campi:
     - `Source`
     - `Environment`
     - `Privileged mode = ON`
     - `Service role`
   - Nel Learner Lab `codebuild:CreateProject` puo essere bloccato: se succede, non creare il progetto.

3. **Build e push manuale**
   - Riusa `REGION`, `ACCT` e `ECR_REPO` del lab 05.

   ```bash
   COMMIT=$(git rev-parse --short HEAD)
   docker build -t hello-api:$COMMIT .
   docker tag hello-api:$COMMIT $ECR_REPO:$COMMIT
   docker tag hello-api:$COMMIT $ECR_REPO:latest
   docker push $ECR_REPO:$COMMIT
   docker push $ECR_REPO:latest
   ```

4. **Verifica in ECR**
   - ECR -> `demo-hello-api` -> `Images`
   - Controlla la presenza del tag `latest` e del tag `commit-sha`.

5. **Aggiorna ECS**
   - Crea una nuova task definition revision con l'immagine taggata dal commit.
   - Aggiorna il service con `Force new deployment`.

## Output atteso

- Nuova immagine pubblicata in ECR.
- Tag tracciabile basato sul commit disponibile.
- Service ECS aggiornato.

## Checkpoint

- Sai perche un tag `commit-sha` e piu utile di `latest` da solo.
- Sai cosa serve in CodeBuild per eseguire una Docker build.
- Sai tracciare quale versione sta girando nel service.

## Troubleshooting

- **Build Docker fallita**: controlla il `Dockerfile` e il contesto di build.
- **Push negato**: ripeti il login ECR e verifica `ECR_REPO`.
- **Service non si aggiorna**: crea una nuova revision e forza il deploy.

## Cleanup

- Mantieni i tag utili se passi al lab 14.
- Elimina immagini di test inutili solo se non ti servono piu.
