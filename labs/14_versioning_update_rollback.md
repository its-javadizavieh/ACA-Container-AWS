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

2) **Crea nuova task definition revision** 🎯 *Sfida*
   - Cambia immagine da `:1.0` a `:1.1` (o digest)
   - *Sfida*: invece del tag, usa il digest SHA256 completo. Dove lo trovi?

3) **Update del service**
   - Service → Update → seleziona la nuova revision

4) **Osserva Events**
   - Output atteso: rollout e poi stable

5) **Rollback** 🎯 *Sfida*
   - Aggiorna il service alla revision precedente
   - *Sfida*: quanto tempo ci mette il rollback? Osserva il tempo tra "draining" e "stable".

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

---

## Tutorial consigliati

- [Amazon ECS — Updating a Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/update-service.html)
- [Amazon ECR — Image Tag Mutability](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)
- [Docker — Image Digests](https://docs.docker.com/reference/cli/docker/image/pull/#pull-an-image-by-digest-immutable-identifier)

---

## Soluzioni

<details>
<summary>Sfida Step 2: trovare e usare il digest SHA256</summary>

**Dove trovare il digest**:

1. **In Console ECR**:
   - ECR → repository → Images
   - Colonna "Image digest" (es. `sha256:abc123...`)

2. **Con AWS CLI**:

   ```bash
   aws ecr describe-images --repository-name hello-api \
     --query 'imageDetails[*].[imageTags,imageDigest]'
   ```

3. **Con Docker**:

   ```bash
   docker inspect --format='{{index .RepoDigests 0}}' hello-api:1.0
   ```

**Come usarlo nella task definition**:

```
123456789012.dkr.ecr.eu-west-1.amazonaws.com/hello-api@sha256:abc123def456...
```

**Nota**: usa `@sha256:` invece di `:tag`.

**Perché digest è meglio di tag**:

- **Immutabile**: garantisce esattamente quella versione
- **Riproducibile**: stesso digest = stesso contenuto sempre
- **Sicuro**: nessuno può "sovrascrivere" come con i tag

</details>

<details>
<summary>Sfida Step 5: tempo di rollback</summary>

**Cosa osservare negli Events**:

1. Evento "service has begun draining task..." → **T0**
2. Evento "service has started 1 tasks..." → task nuovi (vecchia revision)
3. Evento "service has stopped 1 running tasks..." → task nuovi fermati
4. Evento "service has reached a stable state" → **T1**

**Tempo tipico**: 2-5 minuti con configurazione default

**Fattori che influenzano il tempo**:

- `deregistration_delay` del target group (default 300s → **riduci a 30s per dev**)
- `healthCheckGracePeriodSeconds` del service
- Tempo di startup dell'app
- Configurazione `minimumHealthyPercent`

**Quick win per rollback veloci**:

```
# Riduci deregistration delay
aws elbv2 modify-target-group-attributes \
  --target-group-arn <arn> \
  --attributes Key=deregistration_delay.timeout_seconds,Value=30
```

</details>
