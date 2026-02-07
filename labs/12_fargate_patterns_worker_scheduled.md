# Lab 12 — Fargate pattern: worker (o task schedulato) in 30 minuti

Riferimento lezione: `slides_deck/lecture_12_fargate_patterns_hybrid_en.md`

## Obiettivo

- Capire un pattern Fargate diverso dal classico “web behind ALB”.
- Eseguire una **one-off task** (job) che termina.
- Ragionare su scaling e costi.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Cluster ECS disponibile.
- Permessi ECS.
- (Opzionale) Accesso a CloudWatch Logs.

---

## Mini-project (ongoing)

Esegui almeno un workload “non-web” legato al progetto.

Deliverable (scegli uno):

- esegui una one-off task usando la tua image “hello-api” (se pronta)
- oppure esegui il job con `alpine` come nel lab e descrivi come lo renderesti un task schedulato/worker su ECS
- in entrambi i casi: verifica exit code e “stopped reason” e (se possibile) i log

## Scenario

Eseguiamo un job containerizzato (es. `alpine`) che fa un’azione e termina.

---

## Step (numerati)

1) **Crea una task definition minimale**
   - Compatibilità Fargate
   - Immagine: `public.ecr.aws/docker/library/alpine:3.19`
   - Command: `sh -c "echo START; date; echo DONE"`
   - Logging: awslogs (se possibile)

2) **Run task (one-off)**
   - ECS → Run task
   - Desired tasks: 1

3) **Verifica esecuzione**
   - Output atteso: task passa RUNNING → STOPPED (exit code 0)

4) **Leggi i log (se configurati)**
   - CloudWatch Logs → log group/stream

5) **Discussione rapida**
   - differenza: ECS Service (sempre acceso) vs RunTask (job)

---

## Output atteso

- Task one-off eseguita correttamente e terminata.
- Log letti o “stopped reason” verificato.

## Checkpoint

- Sai distinguere: service vs job.
- Sai spiegare perché è spesso più economico.

---

## Troubleshooting rapido

- **Task non parte**: controlla subnet/SG e “stopped reason”.
- **No logs**: awslogs non configurato o permessi mancanti.

---

## Cleanup obbligatorio

- Stop task se rimane RUNNING.
- (Opzionale) deregister task definition revision se creata solo per prova.

---

## Parole chiave Google (screenshot/guide)

- ECS RunTask Fargate one-off task screenshot
- ECS scheduled task EventBridge RunTask
- ECS service vs task run differences
- ECS task stopped reason exit code 0
