# Lab 05 - ECR: repository, push, pull e scan

## Obiettivo

- Creare o riusare un repository Amazon ECR.
- Fare login Docker su ECR.
- Pubblicare e scaricare l'immagine `hello-api`.
- Controllare tag, digest e scan.

## Durata

30 minuti.

## Prerequisiti

- Docker funzionante.
- Image locale `hello-api:1.0` disponibile.
- AWS CLI configurata.

## Specifiche del laboratorio

- Devi usare la stessa image locale del corso e pubblicarla nel repository ECR del lab.
- Il risultato minimo e una image con tag leggibile e digest visibile in console.
- Se lo scan non e disponibile, devi comunque annotare la limitazione e spiegare a cosa servirebbe `Scan on push`.

## Guida del lab

1. **Imposta le variabili**
   - Bash:
     ```bash
     REGION=us-east-1
     ACCT=$(aws sts get-caller-identity --query Account --output text)
     ECR_REPO=$ACCT.dkr.ecr.$REGION.amazonaws.com/demo-hello-api
     ```
   - PowerShell:
     ```powershell
     $REGION = "us-east-1"
     $ACCT = aws sts get-caller-identity --query Account --output text
     $ECR_REPO = "$ACCT.dkr.ecr.$REGION.amazonaws.com/demo-hello-api"
     ```

2. **Crea il repository ECR**
   - Console -> `ECR` -> `Repositories` -> `Create repository`
   - Repository name: `demo-hello-api`
   - `Scan on push`: attivalo se disponibile nel tuo lab

3. **Login Docker su ECR**
   - Bash:
     ```bash
     aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ACCT.dkr.ecr.$REGION.amazonaws.com
     ```
   - PowerShell:
     ```powershell
     aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin "$ACCT.dkr.ecr.$REGION.amazonaws.com"
     ```

4. **Tag e push dell'immagine**

   ```bash
   docker tag hello-api:1.0 $ECR_REPO:1.0
   docker push $ECR_REPO:1.0
   ```

5. **Verifica in console**
   - ECR -> `demo-hello-api` -> `Images`
   - Annota `Image tag` e `Image digest`.

6. **Pull di test**

   ```bash
   docker rmi hello-api:1.0
   docker pull $ECR_REPO:1.0
   docker run --rm -p 9090:9090 $ECR_REPO:1.0
   ```

   In un secondo terminale verifica:

   ```bash
   curl http://localhost:9090/health
   ```

7. **Scan findings** 🎯 _Sfida_
   - Apri l'immagine in ECR e cerca `Scan results`.
   - Se lo scan non e disponibile, annota la limitazione del lab.

## Output atteso

- Repository `demo-hello-api` creato.
- Tag `1.0` presente in ECR.
- Pull di test completato.

## Checkpoint

- Sai spiegare differenza tra tag e digest.
- Sai quando usare `scan on push`.
- Sai fare login Docker su ECR.

## Troubleshooting

- **`AccessDenied`**: controlla account e Region.
- **`docker login` fallisce**: ripeti il comando `get-login-password`.
- **Repository non trovato**: verifica nome `demo-hello-api`.

## Cleanup

- Se passi al lab 06, mantieni il repository.
- Altrimenti: ECR -> `demo-hello-api` -> `Delete`, con `delete images` selezionato.
