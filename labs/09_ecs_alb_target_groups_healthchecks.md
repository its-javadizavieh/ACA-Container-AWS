# Lab 09 - ECS + ALB: Target Group e health check

## Obiettivo

- Pubblicare `hello-api` dietro un Application Load Balancer.
- Configurare un Target Group corretto per Fargate.
- Verificare health check e raggiungibilita dell'app.

## Durata

30 minuti.

## Prerequisiti

- `hello-api-service` attivo.
- `demo-alb-sg` e `demo-task-sg` pronti dal lab 08.

## Specifiche del laboratorio

- Qui colleghi il service ECS a un ALB reale e verifichi il comportamento del Target Group.
- Devi controllare coerenza tra listener, target group, porta container e path `/health`.
- Se un target diventa `unhealthy`, il lavoro non e fermarsi: devi leggere il dettaglio del motivo.

## Guida del lab

1. **Crea il Target Group**
   - EC2 -> `Target Groups` -> `Create target group`
   - Target type: `IP`
   - Name: `demo-tg`
   - Protocol: `HTTP` | Port: `9090`
   - VPC: default
   - Health check path: `/health`
   - Clicca `Create`

2. **Crea l'Application Load Balancer**
   - EC2 -> `Load Balancers` -> `Create Load Balancer`
   - Type: `Application Load Balancer`
   - Name: `demo-alb`
   - Scheme: `internet-facing`
   - Subnets: almeno `2` subnet pubbliche
   - Security group: `demo-alb-sg`
   - Listener `HTTP:80` -> forward a `demo-tg`

3. **Collega l'ALB al service ECS**
   - ECS -> `Clusters` -> `demo-cluster` -> `Services` -> `hello-api-service` -> `Update`
   - In `Load balancing`, aggiungi l'ALB creato
   - Listener: `HTTP:80`
   - Container: `hello-api`, porta `9090`
   - Target group: `demo-tg`
   - Clicca `Update`

4. **Verifica health e DNS**
   - EC2 -> `Target Groups` -> `demo-tg` -> `Targets`
   - Attendi stato `healthy`
   - EC2 -> `Load Balancers` -> copia il `DNS name`
   - Browser: `http://<alb-dns>/health`

5. **Controllo finale** 🎯 _Sfida_
   - Se un target e `unhealthy`, apri `Health status details` e annota il motivo.

## Output atteso

- ALB creato e listener `80` attivo.
- Target Group con target `healthy`.
- App raggiungibile tramite DNS dell'ALB.

## Checkpoint

- Sai perche per Fargate il `Target type` deve essere `IP`.
- Sai dove leggere il motivo di un target `unhealthy`.
- Sai distinguere Security Group ALB e Security Group task.

## Troubleshooting

- **Target `unhealthy`**: controlla path `/health`, porta `9090` e source del `demo-task-sg`.
- **Browser in timeout**: verifica listener, subnet pubbliche e DNS dell'ALB.

## Cleanup

- Se passi al lab 15, mantieni ALB e Target Group.
- Altrimenti elimina ALB, poi Target Group, poi le regole SG non piu necessarie.
