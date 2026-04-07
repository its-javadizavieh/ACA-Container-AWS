# Lab 05 - Amazon ECR: repository + push/pull + scan

## Obiettivo

- Creare un repository **Amazon ECR**.
- Autenticare Docker verso ECR.
- Fare push/pull di un’immagine con tagging tracciabile.
- Verificare funzionalità di **scan** (se disponibile nel tuo account).

## Durata (timebox)

30 minuti.

## Prerequisiti

- Account AWS con permessi su ECR.
- AWS CLI configurata (oppure uso Console).
- Docker installato.
- Una image locale da pubblicare (es. dal Lab 02) oppure una image fornita dal docente.

---

## Mini-project (ongoing)

Crea il perimetro di registry privato per il progetto.

Deliverable:

- repository ECR per “hello-api”
- push di almeno un tag tracciabile (preferibile `sha-<commit>`)
- se scan è disponibile: verifica risultati e annota l’esito

---

## Step (numerati)

1. **Scegli la Region e imposta le variabili**

   **Bash (Linux / macOS)**
   ```bash
   REGION=us-east-1
   ACCT=$(aws sts get-caller-identity --query Account --output text)
   ECR_REPO=$ACCT.dkr.ecr.$REGION.amazonaws.com/demo-hello-api
   ```

   **PowerShell (Windows)**
   ```powershell
   $REGION = "us-east-1"
   $ACCT = aws sts get-caller-identity --query Account --output text
   $ECR_REPO = "$ACCT.dkr.ecr.$REGION.amazonaws.com/demo-hello-api"
   ```

2. **Crea repository ECR**
   - Console ──► ECR ──► Repositories ──► Create repository
   - Nome consigliato: `hello-api`

3. **Login Docker su ECR**

   **Bash**
   ```bash
   aws ecr get-login-password --region $REGION | \
     docker login --username AWS --password-stdin $ACCT.dkr.ecr.$REGION.amazonaws.com
   ```

   **PowerShell**
   ```powershell
   aws ecr get-login-password --region $REGION |
     docker login --username AWS --password-stdin "$ACCT.dkr.ecr.$REGION.amazonaws.com"
   ```

4. **Tag dell'immagine locale verso ECR** 🎯 _Sfida_

   ```bash
   docker tag hello-api:1.0 $ECR_REPO:1.0
   ```
   - _Sfida_: spiega cosa succede se usi lo stesso tag `1.0` per due immagini diverse.

5. **Push su ECR**

   ```bash
   docker push $ECR_REPO:1.0
   ```
   > I comandi `docker tag` e `docker push` sono identici in Bash e PowerShell (le variabili impostate allo Step 1 funzionano in entrambi).

6. **Verifica in Console**
   - ECR ──► repository ──► Images: deve comparire il tag `1.0`.

7. **Pull (test)**

   ```bash
   docker rmi hello-api:1.0          # rimuove solo il tag locale
   docker pull $ECR_REPO:1.0
   docker run --rm -p 9090:9090 $ECR_REPO:1.0
   curl http://localhost:9090/       # verifica che risponde
   ```

8. **(Opzionale) Scan findings** 🎯 _Sfida_
   - ECR ──► Image ──► Scan results
   - _Sfida_: se trovi vulnerabilità CRITICAL o HIGH, cerca la CVE e spiega cosa rischi.

---

## Output atteso

- Repository ECR creato.
- Immagine pubblicata con tag `1.0`.
- Pull riuscito.

## Checkpoint

- Sai spiegare differenza tra **tag** e **digest**.
- Sai dire perché ECR è parte del perimetro di sicurezza.

---

## Troubleshooting rapido

- **AccessDenied**: controlla policy IAM e Region.
- **docker login fails**: controlla Region e ora del sistema.
- **push lento**: verifica dimensione immagine e rete.

---

## Cleanup obbligatorio

1. ECR ──► Repository ──► Delete (spunta “delete images”).
2. Verifica che non restino repository inutili.

---

## Parole chiave Google (screenshot/guide)

- Amazon ECR create repository console screenshot
- site:docs.aws.amazon.com Amazon ECR get-login-password
- docker login ecr get-login-password example
- Amazon ECR scan on push enable
- ECR repository policy examples

---

## Tutorial consigliati

- [Amazon ECR User Guide - Getting Started](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-console.html)
- [ECR Image Scanning](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html)
- [Docker Docs - Build and Push Image](https://docs.docker.com/get-started/introduction/build-and-push-first-image/)


