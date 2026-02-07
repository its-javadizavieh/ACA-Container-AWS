# Lab 14 — Versioning: update immagine e rollback rapido

Riferimento lezione: `slides_deck/lecture_14_versioning_updates_en.md`

## Obiettivo

- Aggiornare una applicazione cambiando **tag** (o digest) immagine.
- Rilasciare una nuova task definition revision.
- Fare rollback alla revisione precedente.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Un service ECS esistente (1 task) oppure un task runnabile.
- Una image in ECR con almeno 2 tag (es. `1.0` e `1.1`).

---

## Mini-project (ongoing)

Rendi “hello-api” aggiornabile e rollbackabile in modo sicuro.

Deliverable:

- fai deploy di una nuova versione (tag o digest) con nuova task definition revision
- fai rollback alla revisione precedente e osserva gli Events
- annota: tag versione + task definition revision in esecuzione (opzionale: digest)

---

## Step (numerati)

1) **Verifica tag disponibili in ECR**
   - ECR → repository → Images

2) **Crea nuova task definition revision**
   - Cambia immagine da `:1.0` a `:1.1` (o digest)

3) **Update del service**
   - Service → Update → seleziona la nuova revision

4) **Osserva Events**
   - Output atteso: rollout e poi stable

5) **Rollback**
   - Aggiorna il service alla revision precedente

---

## Output atteso

- Deploy effettuato e rollback completato.

## Checkpoint

- Sai spiegare perché un tag può essere “mutabile”.
- Sai dire dove leggere gli eventi del cambiamento.

---

## Troubleshooting rapido

- **Non cambia nulla**: forza nuova revision e/o usa tag diversi.
- **CannotPullContainerError**: controlla execution role.

---

## Cleanup obbligatorio

- Riporta il service a una revisione stabile.
- (Opzionale) rimuovi tag temporanei in ECR.

---

## Parole chiave Google (screenshot/guide)

- ECS update service new task definition revision screenshot
- ECS rollback to previous task definition revision
- docker tag vs digest reproducibility
