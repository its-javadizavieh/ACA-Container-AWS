# Lab 08 - Networking: VPC, subnet e Security Groups

## Obiettivo

- Riconoscere subnet pubbliche e private.
- Preparare i Security Groups per il flusso `Internet -> ALB -> task`.
- Capire quando servono NAT Gateway o VPC endpoints.

## Durata

30 minuti.

## Prerequisiti

- Accesso alla console AWS.
- VPC default o VPC del corso disponibile.

## Guida del lab

1. **Identifica VPC e subnet**
   - VPC -> `Your VPCs` -> apri la VPC del lab.
   - VPC -> `Subnets` -> apri almeno `2` subnet.
   - VPC -> `Route tables` -> verifica quale subnet ha route `0.0.0.0/0 -> igw`.

2. **Crea il Security Group dell'ALB**
   - EC2 -> `Security Groups` -> `Create security group`
   - Name: `demo-alb-sg`
   - Inbound rule: `HTTP`, porta `80`, source `0.0.0.0/0`

3. **Aggiorna il Security Group del task**
   - Apri `demo-task-sg` creato nel lab 06.
   - Modifica `Inbound rules`.
   - Aggiungi o sostituisci la regola con:
     - Type: `Custom TCP`
     - Port range: `9090`
     - Source: `demo-alb-sg`

4. **Disegna il flusso minimo** 🎯 _Sfida_
   - Scrivi su una riga: `Internet -> ALB -> Target Group -> ECS task`.
   - Rispondi: perche come source usi un Security Group e non `0.0.0.0/0`?

5. **Ragiona sull'uscita verso AWS services**
   - Se i task vanno in subnet private, come raggiungono ECR e CloudWatch?
   - Opzioni: `NAT Gateway` oppure `VPC endpoints`.

## Output atteso

- `demo-alb-sg` creato.
- `demo-task-sg` pronto per ricevere traffico solo dall'ALB.
- Differenza tra subnet pubblica e privata chiara.

## Checkpoint

- Sai riconoscere una subnet pubblica dalla route table.
- Sai spiegare il path del traffico fino al container.
- Sai perche il task SG non dovrebbe esporre `9090` a Internet quando usi un ALB.

## Troubleshooting

- **Traffico non passa**: verifica source del `demo-task-sg`.
- **Subnet confusa**: ricontrolla route table e Internet Gateway.

## Cleanup

- Se passi al lab 09, mantieni `demo-alb-sg` e `demo-task-sg`.
- Altrimenti elimina prima `demo-task-sg`, poi `demo-alb-sg`.
