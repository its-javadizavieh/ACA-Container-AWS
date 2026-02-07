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

1) **Apri log group del service**
   - CloudWatch → Logs → Log groups
   - Apri stream e identifica:
     - startup logs
     - richieste
     - errori

2) **Correla con ECS events**
   - ECS → Service → Events
   - Obiettivo: evento ↔ log.

3) **Crea un allarme su CPU o Memory**
   - CloudWatch → Alarms → Create
   - Seleziona metrica ECS (ClusterName + ServiceName)
   - Soglia esempio: CPU > 70% per 5 minuti

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
