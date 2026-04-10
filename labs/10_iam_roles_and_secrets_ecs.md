# Lab 10 - IAM per ECS: role e secrets

## Obiettivo

- Distinguere `execution role` e `task role`.
- Salvare un segreto in SSM Parameter Store.
- Iniettare il segreto nella task definition senza metterlo in chiaro.

## Durata

30 minuti.

## Prerequisiti

- Service ECS attivo.
- Accesso a IAM, ECS e Systems Manager.
- `LabRole` disponibile come `execution role`.

## Guida del lab

1. **Crea un parametro SecureString**
   - Systems Manager -> `Parameter Store` -> `Create parameter`
   - Name: `/containersaws/lab/app_secret`
   - Type: `SecureString`
   - Value: scegli un valore di test

2. **Controlla `LabRole`**
   - IAM -> `Roles` -> `LabRole`
   - Verifica che serva alla piattaforma ECS per pull da ECR e push dei log.

3. **Prepara il task role**
   - Usa il task role pre-creato del corso, se disponibile.
   - Se il tuo lab permette un role `service-role/*`, concedi solo `ssm:GetParameter` sul parametro creato.
   - Domanda: perche qui non vuoi una wildcard su tutti i parametri?

4. **Crea una nuova task definition revision**
   - ECS -> `Task definitions` -> `hello-api` -> `Create new revision`
   - Container -> `Secrets`
   - Add secret:
     - Name: `APP_SECRET`
     - Value from: ARN del parametro SSM
   - `Task execution role`: `LabRole`
   - `Task role`: seleziona il ruolo applicativo del corso

5. **Redeploy del service**
   - ECS -> `Services` -> `hello-api-service` -> `Update`
   - Seleziona la nuova revision e abilita `Force new deployment`

6. **Verifica** 🎯 _Sfida_
   - Il task deve avviarsi senza `AccessDenied`.
   - Nei log non deve comparire il valore del secret in chiaro.

## Output atteso

- Parametro SecureString creato.
- Secret referenziato dalla task definition.
- Service aggiornato senza errori IAM.

## Checkpoint

- Sai spiegare differenza tra `execution role` e `task role`.
- Sai perche un segreto va in `Secrets` e non in `Environment variables`.
- Sai dove leggere un errore IAM: `Events`, `Stopped reason`, log.

## Troubleshooting

- **`AccessDenied` su SSM**: controlla il task role e l'ARN del parametro.
- **Secret non iniettato**: verifica di avere usato la sezione `Secrets`.
- **Task non parte**: ricontrolla `LabRole` come `execution role`.

## Cleanup

- Elimina il parametro `/containersaws/lab/app_secret`.
- Non eliminare `LabRole` o i ruoli condivisi del corso.
