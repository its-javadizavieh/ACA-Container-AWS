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

1) **Crea una task definition minimale** 🎯 *Sfida*
   - Compatibilità Fargate
   - Immagine: `public.ecr.aws/docker/library/alpine:3.19`
   - Command: `sh -c "echo START; date; echo DONE"`
   - Logging: awslogs (se possibile)
   - *Sfida*: modifica il command per scrivere un file su /tmp e verificare che esiste.

2) **Run task (one-off)**
   - ECS → Run task
   - Desired tasks: 1

3) **Verifica esecuzione**
   - Output atteso: task passa RUNNING → STOPPED (exit code 0)

4) **Leggi i log (se configurati)**
   - CloudWatch Logs → log group/stream

5) **Discussione rapida** 🎯 *Sfida*
   - differenza: ECS Service (sempre acceso) vs RunTask (job)
   - *Sfida*: calcola il costo di eseguire questo job ogni giorno per un mese (Fargate pricing).

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

---

## Tutorial consigliati

- [Running Tasks with Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_run_task.html)
- [Scheduled Tasks with EventBridge](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/scheduled_tasks.html)
- [AWS Fargate Pricing](https://aws.amazon.com/fargate/pricing/)

---

## Soluzioni

<details>
<summary>Sfida Step 1: command con file su /tmp</summary>

**Command modificato**:

```
sh -c "echo START; date > /tmp/output.txt; cat /tmp/output.txt; echo DONE"
```

**Cosa fa**:

1. Stampa "START"
2. Scrive la data in `/tmp/output.txt`
3. Legge e stampa il contenuto del file
4. Stampa "DONE"

**Output atteso nei log**:

```
START
Thu Mar 20 14:30:00 UTC 2025
DONE
```

**Nota**: il file esiste solo durante l'esecuzione del task. Quando il container termina, tutto viene perso (ephemeral storage).

**Per persistere dati**: usa EFS o scrivi su S3.

</details>

<details>
<summary>Sfida Step 5: calcolo costo job giornaliero</summary>

**Dati per il calcolo** (Fargate eu-west-1, Marzo 2025 circa):

- vCPU: $0.04048 per vCPU-ora
- Memory: $0.004445 per GB-ora

**Configurazione minimale Fargate**:

- 0.25 vCPU + 0.5 GB RAM

**Durata job**: ~10 secondi = 0.00278 ore

**Costo per esecuzione**:

- vCPU: 0.25 × $0.04048 × 0.00278 = $0.000028
- Memory: 0.5 × $0.004445 × 0.00278 = $0.0000062
- **Totale: ~$0.000034** per esecuzione

**Costo mensile** (30 esecuzioni):

- $0.000034 × 30 = **~$0.001** (praticamente gratis)

**Confronto con Service 24/7**:

- Service: 0.25 vCPU × 730h = **~$7.40/mese**
- Job 1x/giorno: **~$0.001/mese**

**Conclusione**: per workload batch, RunTask è 1000x più economico di un Service sempre attivo.

</details>
