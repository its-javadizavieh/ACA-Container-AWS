# Lab 16 — Cost & governance: quick wins (log retention, lifecycle, tagging)

Riferimento lezione: `slides_deck/lecture_16_cost_optimization_governance_en.md`

## Obiettivo

- Applicare 3 “quick wins” di governance/costo:
  1) **Retention** CloudWatch Logs
  2) **Lifecycle policy** su ECR
  3) Tagging consistente

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

1) **CloudWatch Logs: imposta retention**
   - CloudWatch → Log groups
   - Actions → Edit retention
   - Valore didattico: 7 o 14 giorni

2) **ECR: lifecycle policy (semplice)**
   - ECR → repository → Lifecycle policy
   - Regola tipica: tieni solo le ultime N immagini non “release”

3) **Tagging (almeno su 1 risorsa)**
   - Aggiungi tag:
     - `Project=ContainersAWS`
     - `Owner=<gruppo>`

4) **Checklist: cosa elimina i costi più velocemente?**
   - ALB e target groups
   - NAT gateway
   - servizi ECS sempre accesi

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
