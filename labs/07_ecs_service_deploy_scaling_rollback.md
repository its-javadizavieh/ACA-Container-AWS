# Lab 07 - ECS Service: update, scaling e rollback

## Obiettivo

- Aggiornare un service ECS con una nuova task definition revision.
- Osservare il rolling update negli `Events`.
- Fare scaling manuale e rollback.

## Durata

30 minuti.

## Prerequisiti

- `hello-api-service` attivo dal lab 06.
- Task definition `hello-api` gia registrata.

## Guida del lab

1. **Controlla lo stato iniziale**
   - ECS -> `Clusters` -> `demo-cluster` -> `Services` -> `hello-api-service`
   - Apri `Deployments and events` e `Tasks`.
   - Annota: revision attuale, `Desired count`, `Running count`.

2. **Crea una nuova task definition revision**
   - ECS -> `Task definitions` -> `hello-api` -> `Create new revision`
   - Container -> `Environment variables` -> aggiungi `APP_VERSION = 2.0`
   - Clicca `Create`

3. **Aggiorna il service**
   - ECS -> `Services` -> `hello-api-service` -> `Update`
   - `Revision`: seleziona la nuova revision
   - Abilita `Force new deployment`
   - Clicca `Update`

4. **Osserva il rolling update**
   - Torna in `Deployments and events`.
   - Attendi lo stato `Completed`.
   - Se il task ha `Public IP`, verifica `/version` dal browser o da `curl`.

5. **Scaling manuale**
   - `Update` -> cambia `Desired tasks` da `1` a `2`
   - Attendi `Running count = 2`

6. **Rollback** 🎯 _Sfida_
   - `Update` -> seleziona la revision precedente
   - Abilita `Force new deployment`
   - Durante il rollback osserva quanti task `old` e `new` compaiono nella tab `Tasks`.

7. **(Opzionale) Auto scaling**
   - `Update` -> `Service auto scaling - optional`
   - Se il lab non permette `application-autoscaling:RegisterScalableTarget`, limita il lavoro al walkthrough concettuale.

## Output atteso

- Service aggiornato con una nuova revision.
- Scaling a `2` task riuscito.
- Rollback completato correttamente.

## Checkpoint

- Sai distinguere `service` e `task definition revision`.
- Sai leggere gli `Events` durante un deploy.
- Sai fare rollback senza ricreare il service.

## Troubleshooting

- **Deploy non stabile**: controlla `Events`, `Stopped reason` e log CloudWatch.
- **`CannotPullContainerError`**: verifica immagine ECR e `LabRole`.
- **`/version` non cambia**: assicurati di avere selezionato la revision corretta e forzato il deploy.

## Cleanup

- Se passi al lab 09, lascia il service in una revision stabile.
- Altrimenti riporta `Desired tasks` a `1`.
