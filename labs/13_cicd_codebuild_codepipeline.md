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

## Specifiche del laboratorio

- In questo lab devi capire il flusso CI/CD anche se il Learner Lab non permette tutta l'automazione.
- Devi produrre almeno due cose tracciabili: un tag basato sul commit e una nuova revision ECS che lo usa.
- La parte concettuale su CodeBuild serve a spiegare i campi essenziali, ma la verifica pratica passa dal push manuale su ECR.

## Guida del lab

1. **Leggi `buildspec.yml`**
   - Il file e gia presente in `hello-api/buildspec.yml` (stessa cartella di `app.py` e `Dockerfile`).
   - Aprilo e identifica le tre fasi: `pre_build`, `build`, `post_build`.
   - Deve fare:
     - login su ECR
     - build dell'immagine
     - tag con il commit SHA
     - push del tag su ECR

2. **Walkthrough CodeBuild (concettuale — console bloccata)**
   - La pagina CodeBuild → `Create build project` mostra un errore rosso subito (`codebuild:ListCuratedEnvironmentImages` non autorizzato). Non e possibile vedere i campi dalla console.
   - Spiega questi campi a voce o dalla slide:
     - `Source`: CodeCommit / GitHub / S3
     - `Environment`: Amazon Linux 2023, runtime Standard
     - `Privileged mode = ON`: serve per eseguire `docker build` dentro CodeBuild
     - `Service role`: in produzione si userebbe un ruolo dedicato, qui `LabRole`
     - `Buildspec`: "Use a buildspec file" → usa il `buildspec.yml` dalla root
   - Non aprire la pagina — il Learner Lab blocca tutto CodeBuild.

3. **Build e push manuale**
   - Riusa `REGION`, `ACCT` e `ECR_REPO` del lab 05.

   ```bash
   COMMIT=$(git rev-parse --short HEAD)
   docker build -t hello-api:$COMMIT .
   docker tag hello-api:$COMMIT ${ECR_REPO}:$COMMIT
   docker tag hello-api:$COMMIT ${ECR_REPO}:latest
   docker push ${ECR_REPO}:$COMMIT
   docker push ${ECR_REPO}:latest
   ```

   > **⚠️ Utenti zsh:** scrivere sempre `${ECR_REPO}:latest` (con le graffe), **non** `$ECR_REPO:latest`. In zsh `:l` e un modificatore (lowercase) e il tag risulta corrotto (es. `demo-hello-apiatest`).

   Se il push fallisce con `name unknown: The repository with name '...' does not exist`, esegui questi comandi di debug:

   ```bash
   # 1. Verifica che il repository ECR esista
   aws --no-cli-pager ecr describe-repositories --region us-east-1 \
     --query 'repositories[].repositoryName' --output text

   # 2. Controlla il valore effettivo della variabile (nessun carattere nascosto)
   echo -n "$ECR_REPO" | xxd | head -5

   # 3. Confronta le due sintassi (zsh tratta :l come modificatore lowercase)
   echo "Con graffe: ${ECR_REPO}:latest"
   echo "Senza graffe: $ECR_REPO:latest"    # ← in zsh produce un tag corrotto!

   # 4. Controlla cosa ha taggato Docker
   docker images --format '{{.Repository}}:{{.Tag}}' | grep hello

   # 5. Se vedi un tag corrotto (es. demo-hello-apiatest), rimuovilo e ri-tagga
   docker rmi "$(docker images --format '{{.Repository}}:{{.Tag}}' | grep apiatest)" 2>/dev/null
   docker tag hello-api:$COMMIT ${ECR_REPO}:latest
   docker push ${ECR_REPO}:latest
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

- **`name unknown: The repository with name '...' does not exist`**: quasi sempre un problema di zsh. Controlla `docker images | grep hello` — se vedi un tag corrotto come `demo-hello-apiatest` significa che hai usato `$REPO:latest` senza graffe. In zsh `:l` e un modificatore lowercase, quindi il tag risulta "$REPO in minuscolo" + "atest". Soluzione: usa sempre `${REPO}:latest` e ri-tagga l'immagine.
- **Build Docker fallita**: controlla il `Dockerfile` e il contesto di build.
- **Push negato (`denied` o `authorization`)**: ripeti il login ECR (`aws ecr get-login-password ...`) e verifica che `ECR_REPO` punti al repository corretto.
- **Service non si aggiorna**: crea una nuova revision della task definition e forza il deploy con **Force new deployment**.

## Cleanup

- Mantieni i tag utili se passi al lab 14.
- Elimina immagini di test inutili solo se non ti servono piu.
