# Lab 12 - Fargate pattern: worker o job schedulato

## Obiettivo

- Eseguire una task `one-off` che termina da sola.
- Leggere `exit code`, `Stopped reason` e log.
- Capire quando usare `Run task` invece di un service sempre acceso.

## Durata

30 minuti.

## Prerequisiti

- Cluster ECS disponibile.
- `LabRole` disponibile.

## Specifiche del laboratorio

- Questo e il lab principale della Lesson 12 sul lato ECS/Fargate.
- Questo lab mostra un workload non web: una task che parte, fa lavoro e termina.
- Devi verificare `Exit code`, `Stopped reason` e output del job.
- L'obiettivo concettuale e saper dire quando un `Run task` e piu adatto di un `Service` sempre attivo.

## Guida del lab

1. **Crea la task definition del job**
   - ECS -> `Task definitions` -> `Create new task definition`
   - Family: `demo-job`
   - Launch type: `AWS Fargate`
   - `Task execution role`: `LabRole`
   - CPU: `0.25 vCPU` | Memory: `0.5 GB`
   - Container image: `public.ecr.aws/docker/library/alpine:3.19`
   - Command (formato ECS = token separati da virgola):

      ```text
      sh,-c,echo START; date; echo DONE
      ```

   - Logging: attiva `Use log collection` -> `Amazon CloudWatch`
   - Inserisci questi valori nella tabella di logging:

      | Key                     | Value           |
      | ----------------------- | --------------- |
      | `awslogs-group`         | `/ecs/demo-job` |
      | `awslogs-region`        | `us-east-1`     |
      | `awslogs-stream-prefix` | `ecs`           |
      | `awslogs-create-group`  | `true`          |

   - Lascia `Value type = Value` per tutte le righe.

2. **Esegui la task**
   - ECS -> `Clusters` -> `demo-cluster` -> `Run new task`
   - Family: `demo-job`
   - Desired tasks: `1`
  - Networking: default VPC, 2 subnet, security group di default, **Public IP = ON**

  > Il task ha bisogno di accesso in uscita verso CloudWatch Logs. Senza Public IP (e senza NAT Gateway o VPC endpoint), il task fallisce subito con `ResourceInitializationError`.

3. **Verifica il ciclo di vita**
   - Attendi `RUNNING`, poi `STOPPED`
   - Apri il task: il banner in alto nella pagina mostra lo **Stopped reason**
     - **Banner giallo** = uscita normale (es. `Essential container in task exited`)
     - **Banner rosso** = errore (es. `ResourceInitializationError`)
   - Per un job riuscito: banner giallo con `Exit code = 0` nella riga del container

4. **Leggi log e risultato**
   - CloudWatch -> log group del job
   - Cerca `START` e `DONE`

5. **Passo successivo** 🎯 _Sfida_
   - Spiega quando useresti `Run task` invece di un `Service`.
   - Descrivi come lo scheduleresti con EventBridge.

## Output atteso

- Task `demo-job` eseguita e terminata con successo.
- Log del job leggibili in CloudWatch.

## Checkpoint

- Sai distinguere workload `web` e workload `job`.
- Sai leggere `Exit code` e `Stopped reason`.
- Sai descrivere a parole un job schedulato con EventBridge.

## Troubleshooting

- **Task si ferma subito con `ResourceInitializationError: failed to validate logger args… connection issue`**: il task non riesce a raggiungere CloudWatch Logs. Imposta **Public IP = ON** quando lanci il task, oppure verifica che la subnet abbia un NAT Gateway.
- **`executable file not found in $PATH`**: il comando e stato scritto come stringa shell. Usa il formato ECS a token separati da virgola: `sh,-c,echo START; date; echo DONE` (senza virgolette).
- **Task non parte**: controlla rete, `LabRole` e `Stopped reason`.
- **Nessun log**: ricontrolla la raccolta CloudWatch e verifica che Public IP sia attivo.

## Cleanup

- Se il job e ancora `RUNNING`, fermalo.
- Elimina la revision della task definition solo se non ti serve piu.
- Se il docente ha previsto anche il confronto Kubernetes, continua con il Lab 12b.
