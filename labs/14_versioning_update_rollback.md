# Lab 14 - Versioning: tag, digest e rollback

## Obiettivo

- Aggiornare un service usando un nuovo riferimento immagine.
- Confrontare tag mutabile e digest immutabile.
- Fare rollback a una revision precedente.

## Durata

25 minuti.

## Prerequisiti

- Service ECS attivo.
- Repository ECR con almeno due versioni disponibili.
- Se hai solo il tag `1.0`, crea la versione `1.1` seguendo il passo 0 qui sotto.

## Specifiche del laboratorio

- Questo lab riguarda il versionamento del deploy, non la build dell'immagine.
- Devi confrontare almeno un tag e un digest come riferimenti validi della stessa applicazione.
- Devi annotare quale revisione torna stabile dopo il rollback e perche la consideri affidabile.

## Guida del lab

0. **Prepara la versione 1.1** (salta se hai gia due tag in ECR)
   - Nella cartella `hello-api/`, apri `app.py` e cambia la versione di default:

     ```python
     version = os.environ.get("APP_VERSION", "1.1")   # era "1.0"
     ```

   - Imposta le variabili e pubblica:

     ```bash
     ACCT=$(aws sts get-caller-identity --query Account --output text)
     REGION=us-east-1
     REPO=$ACCT.dkr.ecr.$REGION.amazonaws.com/demo-hello-api

     docker build -t hello-api:1.1 .
     docker tag hello-api:1.1 ${REPO}:1.1
     docker push ${REPO}:1.1
     ```

   > **⚠️ Utenti zsh:** usare sempre `${REPO}:1.1` (con le graffe), **non** `$REPO:1.1`. In zsh `:1` non da problemi, ma `:latest` si (vedi troubleshooting lab 13). Per coerenza, usare sempre le graffe.

   - Verifica in ECR: `demo-hello-api` deve avere sia `1.0` che `1.1`.

1. **Guarda tag e digest in ECR**
   - ECR -> `demo-hello-api` -> `Images`
   - Annota un tag stabile e il relativo `Image digest`.

2. **Crea una nuova task definition revision**
   - ECS -> `Task definitions` -> `hello-api` -> `Create new revision`
   - Cambia l'immagine usando:
     - un nuovo tag, oppure
     - il digest completo `...@sha256:...`
   - Controlla la sezione **Environment variables**: se c'e `APP_VERSION` impostata da un lab precedente (ad esempio `2.0` dal lab 10), **rimuovila**. Altrimenti il valore della variabile d'ambiente sovrascrive il default in `app.py` e `/version` non riflette il cambio di immagine.

3. **Aggiorna il service**
   - ECS -> `Services` -> `hello-api-svc` -> `Update`
   - Seleziona la nuova revision
   - Abilita `Force new deployment`

4. **Verifica la nuova versione**
   - Copia il DNS dell'ALB da **EC2 → Load Balancers → `demo-alb` → DNS name**
   - Testa sulla **porta 80** (non aggiungere `:9090` — l'ALB riceve sulla 80 e inoltra al target group sulla 9090):

     ```bash
     curl http://<ALB-DNS>/version
     ```

   - Devi vedere `{"version":"1.1"}` (o il tag che hai appena deployato).

   > ⚠️ Non usare il Public IP del task direttamente — il security group permette traffico sulla porta 9090 solo dall'ALB. Non aggiungere `:9090` all'URL dell'ALB.

5. **Osserva il rollout**
   - Apri `Deployments and events`
   - Annota quando il service torna `stable`.

6. **Rollback** 🎯 _Sfida_
   - Aggiorna di nuovo il service alla revision precedente.
   - Rispondi: quale e piu sicuro in produzione, tag o digest?

## Output atteso

- Nuova revision deployata.
- Rollback completato senza ricreare il service.

## Checkpoint

- Sai spiegare perche un tag e mutabile.
- Sai usare un digest come riferimento immagine.
- Sai leggere il tempo di deploy dai `Events`.

## Troubleshooting

- **Nessun cambiamento visibile**: verifica il riferimento immagine e forza il deploy.
- **`CannotPullContainerError`**: ricontrolla immagine ECR e permessi `LabRole`.

## Cleanup

- Lascia il service su una revision stabile e conosciuta.
