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

## Specifiche del laboratorio

- In questo lab il punto principale e la separazione delle responsabilita tra piattaforma ECS e applicazione.
- Devi creare o usare un secret vero in SSM e referenziarlo senza esporlo in chiaro nella task definition.
- La parte importante non e solo far partire il task, ma spiegare perche `execution role` e `task role` non sono la stessa cosa.

## Guida del lab

1. **Crea un parametro SecureString**
   - Systems Manager -> `Parameter Store` -> `Create parameter`
   - Name: `/containersaws/lab/app_secret`
   - Type: `SecureString`
   - Value: scegli un valore di test

2. **Controlla `LabRole`**
   - IAM -> `Roles` -> `LabRole`
   - Tab **Trust relationships**: apri il JSON e verifica che `ecs-tasks.amazonaws.com` sia nella lista dei servizi. La lista e lunga perche `LabRole` e condiviso nel Learner Lab.
   - Tab **Permissions**: tra le 7 policy collegate, cerca **AmazonSSMManagedInstanceCore**. Questa policy include `ssm:GetParameters`, necessario per l'injection dei secret.
   - Domanda: in produzione, perche non useresti un ruolo cosi ampio?

3. **Prepara il task role**
   - Usa il task role pre-creato del corso, se disponibile.
   - Se il tuo lab permette un role `service-role/*`, concedi solo `ssm:GetParameter` sul parametro creato.
   - Domanda: perche qui non vuoi una wildcard su tutti i parametri?

4. **Crea una nuova task definition revision**
   - ECS -> `Task definitions` -> `hello-api` -> `Create new revision`
   - Container -> `Environment variables` -> `Add environment variable`
   - Compila:
     - Key: `APP_SECRET`
     - Value type: cambia da `Value` a **`ValueFrom`**
     - Value: ARN del parametro SSM
   - Per trovare l'ARN: vai su Systems Manager -> `Parameter Store` -> clicca sul parametro `/containersaws/lab/app_secret` -> copia il valore del campo **ARN** in alto (formato: `arn:aws:ssm:<region>:<account-id>:parameter/containersaws/lab/app_secret`)
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
