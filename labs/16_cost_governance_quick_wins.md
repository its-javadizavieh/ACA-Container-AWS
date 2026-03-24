# Lab 16 — Cost & governance: quick wins (log retention, lifecycle, tagging)

## Obiettivo

- Applicare 3 “quick wins” di governance/costo:
  1. **Retention** CloudWatch Logs
  2. **Lifecycle policy** su ECR
  3. Tagging consistente

## Durata (timebox)

30 minuti.

## Prerequisiti

- Accesso Console AWS.
- Almeno 1 log group (da ECS) e 1 repository ECR.

---

## Mini-project (ongoing)

Metti guardrail minimi (costo + governance) sul progetto “hello-api”.

Deliverable:

- log retention impostata sui log group del progetto (non “never expire”)
- lifecycle policy ECR impostata per non accumulare immagini inutili
- tagging minimo applicato ad almeno 1 risorsa (es. `Project`, `Owner`)

---

## Step (numerati)

1. **CloudWatch Logs: imposta retention**
   - CloudWatch ──► Log groups
   - Actions ──► Edit retention
   - Valore didattico: 7 o 14 giorni

2. **ECR: lifecycle policy (semplice)** 🎯 _Sfida_
   - ECR ──► repository ──► Lifecycle policy
   - Regola tipica: tieni solo le ultime N immagini non "release"
   - _Sfida_: scrivi una policy che mantiene solo le ultime 5 immagini e cancella quelle untagged dopo 1 giorno.

3. **Tagging (almeno su 1 risorsa)**
   - Aggiungi tag:
     - `Project=ContainersAWS`
     - `Owner=<gruppo>`

4. **Checklist: cosa elimina i costi più velocemente?** 🎯 _Sfida_
   - ALB e target groups
   - NAT gateway
   - servizi ECS sempre accesi
   - _Sfida_: ordina queste risorse per costo orario (dalla più cara alla meno cara).

---

## Output atteso

- Retention log impostata.
- Lifecycle policy ECR configurata.
- Tagging applicato.

## Checkpoint

- Sai dire perché log retention e lifecycle policy sono fondamentali.
- Sai riconoscere un “orphan resource”.

---

## Troubleshooting rapido

- **Non posso cambiare retention**: permessi limitati.
- **Lifecycle policy non disponibile**: verifica permessi ECR.

---

## Cleanup obbligatorio

- Se hai creato policy di test non desiderate: rimuovile.

---

## Parole chiave Google (screenshot/guide)

- CloudWatch Logs retention set screenshot
- ECR lifecycle policy keep last images example
- AWS cost allocation tags setup
- NAT gateway cost biggest surprise

---

## Tutorial consigliati

- [CloudWatch Logs Retention and Archiving](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
- [ECR Lifecycle Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/LifecyclePolicies.html)
- [AWS Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)

---

## Soluzioni

<details>
<summary>🎯 Sfida Step 1: calcolo risparmio CloudWatch Logs retention</summary>

**Pricing CloudWatch Logs** (eu-west-1, Febbraio 2026 circa):

- **Storage**: ~$0.03/GB-mese
- **Never expire**: accumuli log all'infinito

**Scenario tipo** (service con 2 task, log moderati):

- Log giornalieri: ~500 MB
- Dopo 1 mese: ~15 GB
- Dopo 6 mesi: ~90 GB
- Dopo 1 anno: ~180 GB

**Costo storage**:

- 1 anno senza retention: 180 GB × $0.03 = **$5.40/mese** (e cresce)
- Con retention 7 giorni: ~3.5 GB × $0.03 = **$0.10/mese**

**Risparmio**: $5.30/mese per **un solo service** ──► **98% di risparmio**

**Best practice**:

- Dev/test: 7 giorni
- Staging: 14-30 giorni
- Produzione: 30-90 giorni (o archivia su S3)

</details>

<details>
<summary>🎯 Sfida Step 2: ECR lifecycle policy (ultime 5 immagini)</summary>

**Policy JSON**:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep only last 5 images",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 5
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

**Cosa fa**:

- Mantiene le **5 immagini più recenti** (tagged o untagged)
- Elimina automaticamente quelle più vecchie

**Varianti utili**:

**Mantieni immagini "release" per sempre**:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep release tags forever",
      "selection": {
        "tagStatus": "tagged",
        "tagPrefixList": ["release"],
        "countType": "imageCountMoreThan",
        "countNumber": 999
      },
      "action": {
        "type": "expire"
      }
    },
    {
      "rulePriority": 2,
      "description": "Keep only last 5 dev images",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 5
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

**Tip**: testa con "Dry run" prima di applicare.

</details>

---

## Tutorial consigliati

- [CloudWatch Logs Retention](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html)
- [ECR Lifecycle Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/LifecyclePolicies.html)
- [AWS Cost Allocation Tags](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-alloc-tags.html)

---

## Soluzioni

<details>
<summary>🎯 Sfida Step 2: ECR lifecycle policy JSON</summary>

**Policy completa** (2 regole):

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Delete untagged images after 1 day",
      "selection": {
        "tagStatus": "untagged",
        "countType": "sinceImagePushed",
        "countUnit": "days",
        "countNumber": 1
      },
      "action": {
        "type": "expire"
      }
    },
    {
      "rulePriority": 2,
      "description": "Keep only last 5 images",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 5
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

**Cosa fa**:

1. Regola 1: immagini senza tag ──► cancella dopo 1 giorno
2. Regola 2: se hai più di 5 immagini ──► cancella le più vecchie

**Tip**: per proteggere release, aggiungi regola che **esclude** tag che matchano `release-*` o `v*`.

</details>

<details>
<summary>🎯 Sfida Step 4: classifica risorse per costo orario</summary>

**Classifica costi orari** (eu-west-1, Marzo 2025 circa):

| Risorsa                              | Costo/ora             | Costo/mese 24/7     |
| ------------------------------------ | --------------------- | ------------------- |
| **NAT Gateway**                      | ~$0.045/h + $0.045/GB | ~$32 + traffico     |
| **ALB**                              | ~$0.0225/h + LCU      | ~$16 + traffico     |
| **ECS Fargate (0.25 vCPU + 0.5 GB)** | ~$0.012/h             | ~$9                 |
| **Target Group**                     | $0                    | $0 (incluso in ALB) |

**Ordine dal più caro**:

1. 🥇 **NAT Gateway** — il killer silenzioso!
2. 🥈 **ALB** — fisso + LCU (Load Balancer Capacity Units)
3. 🥉 **ECS Service Fargate** — dipende da CPU/mem
4. **Target Group** — gratuito

**Quick wins per lab/dev**:

- Elimina NAT Gateway appena finito il lab
- Riduci `desired_count` a 0 quando non serve
- Usa VPC endpoints invece di NAT se hai traffico verso servizi AWS

</details>
