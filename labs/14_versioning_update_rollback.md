# Lab 14 - Versioning: tag, digest e rollback

## Obiettivo

- Aggiornare un service usando un nuovo riferimento immagine.
- Confrontare tag mutabile e digest immutabile.
- Fare rollback a una revision precedente.

## Durata

25 minuti.

## Prerequisiti

- Service ECS attivo.
- Repository ECR con almeno due versioni disponibili.

## Specifiche del laboratorio

- Questo lab riguarda il versionamento del deploy, non la build dell'immagine.
- Devi confrontare almeno un tag e un digest come riferimenti validi della stessa applicazione.
- Devi annotare quale revisione torna stabile dopo il rollback e perche la consideri affidabile.

## Guida del lab

1. **Guarda tag e digest in ECR**
   - ECR -> `demo-hello-api` -> `Images`
   - Annota un tag stabile e il relativo `Image digest`.

2. **Crea una nuova task definition revision**
   - ECS -> `Task definitions` -> `hello-api` -> `Create new revision`
   - Cambia l'immagine usando:
     - un nuovo tag, oppure
     - il digest completo `...@sha256:...`

3. **Aggiorna il service**
   - ECS -> `Services` -> `hello-api-service` -> `Update`
   - Seleziona la nuova revision
   - Abilita `Force new deployment`

4. **Osserva il rollout**
   - Apri `Deployments and events`
   - Annota quando il service torna `stable`.

5. **Rollback** 🎯 _Sfida_
   - Aggiorna di nuovo il service alla revision precedente.
   - Rispondi: quale e piu sicuro in produzione, tag o digest?

## Output atteso

- Nuova revision deployata.
- Rollback completato senza ricreare il service.

## Checkpoint

- Sai spiegare perche un tag e mutabile.
- Sai usare un digest come riferimento immagine.
- Sai leggere il tempo di deploy dai `Events`.

## Troubleshooting

- **Nessun cambiamento visibile**: verifica il riferimento immagine e forza il deploy.
- **`CannotPullContainerError`**: ricontrolla immagine ECR e permessi `LabRole`.

## Cleanup

- Lascia il service su una revision stabile e conosciuta.
