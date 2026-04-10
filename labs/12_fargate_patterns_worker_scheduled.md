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
   - Command:
     ```text
     sh -c "echo START; date; echo DONE"
     ```
   - Logging: `Use log collection` -> `Amazon CloudWatch`

2. **Esegui la task**
   - ECS -> `Clusters` -> `demo-cluster` -> `Run new task`
   - Family: `demo-job`
   - Desired tasks: `1`

3. **Verifica il ciclo di vita**
   - Attendi `RUNNING`, poi `STOPPED`
   - Controlla `Exit code = 0`

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

- **Task non parte**: controlla rete, `LabRole` e `Stopped reason`.
- **Nessun log**: ricontrolla la raccolta CloudWatch.

## Cleanup

- Se il job e ancora `RUNNING`, fermalo.
- Elimina la revision della task definition solo se non ti serve piu.
