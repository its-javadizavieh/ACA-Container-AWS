# Lab 05 — Amazon ECR: repository + push/pull + scan

Riferimento lezione: `slides_deck/lecture_05_ecr_en.md`

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

1) **Scegli la Region del corso**

2) **Crea repository ECR**
   - Console → ECR → Repositories → Create repository
   - Nome consigliato: `hello-api`

3) **Login Docker su ECR**
   - Pattern: `aws ecr get-login-password ... | docker login ...`

4) **Tag dell’immagine locale verso ECR**
   - `docker tag hello-api:1.0 <account>.dkr.ecr.<region>.amazonaws.com/hello-api:1.0`

5) **Push su ECR**
   - `docker push <...>/hello-api:1.0`

6) **Verifica in Console**
   - ECR → repository → Images: deve comparire il tag `1.0`.

7) **Pull (test)**
   - `docker rmi hello-api:1.0` (solo locale)
   - `docker pull <...>/hello-api:1.0`

8) **(Opzionale) Scan findings**
   - ECR → Image → Scan results

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

1) ECR → Repository → Delete (spunta “delete images”).
2) Verifica che non restino repository inutili.

---

## Parole chiave Google (screenshot/guide)

- Amazon ECR create repository console screenshot
- site:docs.aws.amazon.com Amazon ECR get-login-password
- docker login ecr get-login-password example
- Amazon ECR scan on push enable
- ECR repository policy examples
