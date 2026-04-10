# Lab 06 - ECS: cluster, task definition e service

## Obiettivo

- Eseguire `hello-api` su ECS/Fargate.
- Configurare task definition con immagine ECR e CloudWatch Logs.
- Creare un service minimale con `1` task.

## Durata

35 minuti.

## Prerequisiti

- Repository ECR `demo-hello-api` disponibile.
- `LabRole` disponibile come `Task execution role`.
- Default VPC disponibile.

## Specifiche del laboratorio

- In questo lab porti `hello-api` da ECR a ECS/Fargate con una configurazione minima ma corretta.
- Devi configurare quattro elementi coerenti tra loro: image URI, porta `9090`, Security Group e CloudWatch Logs.
- Il risultato atteso non e solo la creazione delle risorse, ma un service con `1` task funzionante e log leggibili.

## Guida del lab

1. **Crea o riusa il cluster**
   - ECS -> `Clusters` -> `Create cluster`
   - Name: `demo-cluster`

2. **Crea il Security Group del task**
   - EC2 -> `Security Groups` -> `Create security group`
   - Name: `demo-task-sg`
   - Inbound rule: `Custom TCP`, porta `9090`, source `0.0.0.0/0`

3. **Crea la task definition `hello-api`**
   - ECS -> `Task definitions` -> `Create new task definition`
   - Family: `hello-api`
   - Launch type: `AWS Fargate`
   - Task execution role: `LabRole`
   - CPU: `0.25 vCPU` | Memory: `0.5 GB`
   - Container name: `hello-api`
   - Image URI: usa `Browse ECR images` e seleziona `demo-hello-api:1.0`
   - Container port: `9090`
   - Logging: `Use log collection` -> `Amazon CloudWatch`

4. **Crea il service**
   - ECS -> `Clusters` -> `demo-cluster` -> `Services` -> `Create`
   - `Application type`: `Service`
   - `Task definition family`: `hello-api`
   - `Revision`: `LATEST`
   - `Service name`: `hello-api-service`
   - `Service type`: `Replica`
   - `Desired tasks`: `1`
   - Networking: VPC default, subnet preselezionate, `Use an existing security group` -> `demo-task-sg`, `Public IP` -> `Turned on`
   - Clicca `Create`

5. **Verifica il deploy**
   - Controlla `Running count = 1`.
   - Apri la tab `Tasks`, copia il `Public IP` e prova:
     ```text
     http://<public-ip>:9090/health
     ```

6. **Events e log** 🎯 _Sfida_
   - Apri `Deployments and events`.
   - Apri il log group CloudWatch del task.
   - Scrivi la riga che conferma l'avvio dell'app.

## Output atteso

- Cluster e task definition creati.
- Service `hello-api-service` in esecuzione.
- Log disponibili in CloudWatch.

## Checkpoint

- Sai distinguere `task definition`, `task` e `service`.
- Sai dove trovare `Stopped reason` e `Events`.
- Sai verificare che l'immagine venga presa da ECR.

## Troubleshooting

- **`CannotPullContainerError`**: controlla immagine ECR e `LabRole`.
- **Connessione rifiutata**: verifica porta `9090`, `Public IP` e `demo-task-sg`.
- **Nessun log**: controlla la configurazione CloudWatch nella task definition.

## Cleanup

- Se passi al lab 07, lascia `hello-api-service` attivo.
- Altrimenti elimina il service e ferma i task.
