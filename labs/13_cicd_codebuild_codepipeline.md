# Lab 13 — CI/CD: CodeBuild → ECR → deploy ECS

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

2) **Crea `buildspec.yml` (minimo)**
   - Login su ECR
   - Build immagine
   - Tag con `latest` + `commit-sha` (o timestamp)
   - Push su ECR

3) **Crea o usa un progetto CodeBuild**
   - Abilita *privileged mode* per Docker build

4) **Esegui build**
   - Start build
   - Verifica log e fase di push

5) **Verifica immagine su ECR**
   - Deve comparire un nuovo tag/digest

6) **Deploy su ECS (manuale nel lab)**
   - Aggiorna task definition con nuovo tag
   - ECS → Service → Update → force new deployment

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
