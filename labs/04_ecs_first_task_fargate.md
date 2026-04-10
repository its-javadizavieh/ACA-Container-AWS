# Lab 04 - ECS: primo task su Fargate

## Obiettivo

- Avviare un task ECS su Fargate dalla console.
- Verificare stato task, `Stopped reason` e log.
- Testare il task via `Public IP`.

## Durata

30 minuti.

## Prerequisiti

- Account AWS con permessi ECS e CloudWatch.
- Default VPC disponibile.
- `LabRole` disponibile come `Task execution role`.

## Guida del lab

1. **Crea o riusa il cluster**
   - ECS -> `Clusters` -> `Create cluster`
   - Name: `demo-cluster`

2. **Crea la task definition**
   - ECS -> `Task definitions` -> `Create new task definition`
   - Family: `demo-nginx`
   - Launch type: `AWS Fargate`
   - Task execution role: `LabRole`
   - CPU: `0.25 vCPU` | Memory: `0.5 GB`
   - Container name: `web`
   - Image URI: `public.ecr.aws/docker/library/nginx:alpine`
   - Container port: `80`
   - Logging: `Use log collection` -> `Amazon CloudWatch` -> auto-configure

3. **Crea il Security Group per il test**
   - EC2 -> `Security Groups` -> `Create security group`
   - Name: `demo-nginx-sg`
   - Inbound rule: `HTTP`, porta `80`, source `0.0.0.0/0`

4. **Esegui il task**
   - ECS -> `Clusters` -> `demo-cluster` -> `Tasks` -> `Run new task`
   - Launch type: `Fargate`
   - Task definition family: `demo-nginx`
   - Desired tasks: `1`
   - VPC: default
   - Subnets: seleziona almeno `2`
   - Security group: `demo-nginx-sg`
   - Public IP: `Turned on`

5. **Verifica**
   - Attendi stato `RUNNING`.
   - Apri il task e copia il `Public IP`.
   - Browser: `http://<public-ip>`

6. **Debug rapido** 🎯 _Sfida_
   - Se il task va in `STOPPED`, leggi `Stopped reason`.
   - Apri il log group CloudWatch creato automaticamente.
   - Scrivi il primo indizio utile che trovi.

## Output atteso

- Task in stato `RUNNING`.
- Pagina nginx raggiungibile via browser.
- Sai trovare `Stopped reason` e log.

## Checkpoint

- Sai la differenza tra task definition e task.
- Sai quali campi di rete servono per avviare un task Fargate.
- Sai dove leggere i log del task.

## Troubleshooting

- **Task `STOPPED` subito**: controlla `Stopped reason`.
- **Nessuna pagina via browser**: verifica `Public IP` e regola inbound `80`.
- **Nessun log**: controlla che la raccolta CloudWatch sia abilitata.

## Cleanup

- Ferma il task.
- Elimina `demo-nginx-sg` se non ti serve.
- Mantieni `demo-cluster` se passi subito al lab 06.
