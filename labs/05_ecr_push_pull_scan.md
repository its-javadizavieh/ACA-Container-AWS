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

4) **Tag dell'immagine locale verso ECR** 🎯 *Sfida*
   - `docker tag hello-api:1.0 <account>.dkr.ecr.<region>.amazonaws.com/hello-api:1.0`
   - *Sfida*: spiega cosa succede se usi lo stesso tag `1.0` per due immagini diverse.

5) **Push su ECR**
   - `docker push <...>/hello-api:1.0`

6) **Verifica in Console**
   - ECR → repository → Images: deve comparire il tag `1.0`.

7) **Pull (test)**
   - `docker rmi hello-api:1.0` (solo locale)
   - `docker pull <...>/hello-api:1.0`

8) **(Opzionale) Scan findings** 🎯 *Sfida*
   - ECR → Image → Scan results
   - *Sfida*: se trovi vulnerabilità CRITICAL o HIGH, cerca la CVE e spiega cosa rischi.

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

---

## Tutorial consigliati

- [Amazon ECR User Guide — Getting Started](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-console.html)
- [ECR Image Scanning](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html)
- [Docker Docs — Build and Push Image](https://docs.docker.com/get-started/introduction/build-and-push-first-image/)

---

## Soluzioni

<details>
<summary>Sfida Step 4: stesso tag per due immagini diverse</summary>

**Risposta**: il nuovo push **sovrascrive** l'immagine precedente associata a quel tag.

Il vecchio image digest rimane in ECR ma **senza tag** (diventa "untagged").

**Problema**: se un service ECS usa `hello-api:1.0`, potrebbe non aggiornarsi automaticamente (Fargate cacherà l'immagine per qualche tempo).

**Best practice**:

- Usa tag immutabili basati su commit SHA: `hello-api:sha-abc1234`
- Abilita "Tag immutability" nel repository ECR per evitare sovrascritture

</details>

<details>
<summary>Sfida Step 8: interpretare scan findings</summary>

Esempio di vulnerabilità tipiche:

| Severity | Esempio CVE | Cosa rischi |
|----------|-------------|-------------|
| CRITICAL | CVE-2021-44228 (Log4Shell) | Remote Code Execution |
| HIGH | CVE-2022-0778 (OpenSSL) | Denial of Service |
| MEDIUM | Librerie outdated | Potenziale exploitation futura |

**Azioni da fare**:

1. Per CRITICAL/HIGH: aggiorna la base image immediatamente
2. Cerca la CVE su [NVD](https://nvd.nist.gov/) o [Snyk](https://security.snyk.io/)
3. Se non puoi aggiornare: documenta il rischio accettato

**Tip**: abilita "Scan on push" per controllo automatico.

</details>
