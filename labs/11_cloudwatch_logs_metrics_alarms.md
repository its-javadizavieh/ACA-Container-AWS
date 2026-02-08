# Lab 11 — CloudWatch per container: log, metriche e allarmi

Riferimento lezione: `slides_deck/lecture_11_cloudwatch_for_containers_en.md`

## Obiettivo

- Leggere e filtrare log di container su **CloudWatch Logs**.
- Creare una metrica (es. CPU service) e un **allarme**.
- (Opzionale) costruire una dashboard minimale.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Un service ECS attivo (da Lab 06/09).
- Permessi su CloudWatch e Logs.

---

## Mini-project (ongoing)

Rendi “hello-api” osservabile e sostenibile nei costi.

Deliverable:

- trovi e filtri i log dell’app in CloudWatch Logs
- crei almeno 1 allarme (CPU o memory; opzionale: unhealthy targets se disponibile)
- imposti (o proponi) una retention dei log per evitare costi inattesi

---

## Step (numerati)

1) **Apri log group del service** 🎯 *Sfida*
   - CloudWatch → Logs → Log groups
   - Apri stream e identifica:
     - startup logs
     - richieste
     - errori
   - *Sfida*: usa il filtro per trovare solo le righe che contengono "error" o "ERROR".

2) **Correla con ECS events**
   - ECS → Service → Events
   - Obiettivo: evento ↔ log.

3) **Crea un allarme su CPU o Memory** 🎯 *Sfida*
   - CloudWatch → Alarms → Create
   - Seleziona metrica ECS (ClusterName + ServiceName)
   - Soglia esempio: CPU > 70% per 5 minuti
   - *Sfida*: configura un'azione SNS (anche solo un topic vuoto) per ricevere notifiche.

4) **(Opzionale) Crea una dashboard minimale**
   - Aggiungi widget CPU/memory e stato allarme.

---

## Output atteso

- Log consultabili e filtrabili.
- Allarme creato e visibile.

## Checkpoint

- Sai dire la differenza tra **logs** e **metrics**.
- Sai trovare rapidamente la causa di un task “crash loop”.

---

## Troubleshooting rapido

- **Non vedo log**: controlla `awslogs` nel task definition e permessi dell’execution role.
- **Metrica non disponibile**: verifica namespace ECS e che il service stia inviando metriche.
- **Allarme non scatta**: controlla periodo/statistica e soglia.

---

## Cleanup obbligatorio

- Elimina allarmi e dashboard creati per il lab.

---

## Parole chiave Google (screenshot/guide)

- CloudWatch Logs view log streams screenshot
- Amazon ECS CloudWatch metrics ClusterName ServiceName
- CloudWatch alarm create from metric screenshot
- CloudWatch dashboard create widget screenshot

---

## Tutorial consigliati

- [CloudWatch Logs — Getting Started](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
- [Using Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html)
- [Creating CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

---

## Soluzioni

<details>
<summary>Sfida Step 1: filtrare log per "error"</summary>

**Metodo 1 — Filter pattern nel log group**:

1. CloudWatch → Logs → Log groups → [tuo log group]
2. Nella barra "Filter events", scrivi: `?error ?ERROR ?Error`
3. Premi Enter

**Metodo 2 — Logs Insights (più potente)**:

1. CloudWatch → Logs → Logs Insights
2. Seleziona il log group
3. Query:

```
fields @timestamp, @message
| filter @message like /(?i)error/
| sort @timestamp desc
| limit 50
```

**Tip**: `(?i)` rende la ricerca case-insensitive.

**Varianti utili**:

- `?error ?exception ?failed` — cerca più pattern
- `ERROR -"health check"` — escludi health check noise

</details>

<details>
<summary>Sfida Step 3: configurare azione SNS per allarme</summary>

**Passo per passo**:

1. **Crea topic SNS** (se non esiste):
   - Amazon SNS → Topics → Create topic
   - Type: Standard
   - Name: `ecs-alerts`

2. **Crea subscription** (opzionale, per ricevere email):
   - SNS → Topics → ecs-alerts → Create subscription
   - Protocol: Email
   - Endpoint: tua email
   - Conferma l'email ricevuta

3. **Collega all'allarme**:
   - CloudWatch → Alarms → Create alarm
   - Dopo aver scelto metrica e soglia:
   - "Notification" → In alarm → Select SNS topic → `ecs-alerts`

**Risultato**: quando CPU > 70% per 5 min, ricevi email.

**Costo**: SNS è quasi gratuito per volumi bassi (prime 1M richieste gratis).

</details>
