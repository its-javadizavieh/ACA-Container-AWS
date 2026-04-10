# Lab 16 - Costi e governance: quick wins

## Obiettivo

- Definire retention dei log.
- Limitare la crescita delle immagini in ECR.
- Applicare tagging coerente alle risorse del progetto.

## Durata

20 minuti.

## Prerequisiti

- Almeno un log group CloudWatch e un repository ECR esistenti.
- Accesso alla console AWS.

## Guida del lab

1. **Retention dei log**
   - CloudWatch -> `Log groups`
   - Apri il log group dell'app -> `Actions` -> `Edit retention setting`
   - Scegli `7` o `14` giorni
   - Se il lab blocca `logs:PutRetentionPolicy`, annota solo il valore scelto.

2. **Lifecycle policy ECR**
   - ECR -> `demo-hello-api` -> `Lifecycle policy` -> `Create rule`
   - Regola 1: elimina immagini `untagged` vecchie di `1` giorno
   - Regola 2: mantieni solo le ultime `5` immagini usate per test

3. **Tagging minimo**
   - Applica almeno questi tag a una risorsa del progetto:
     - `Project=ContainersAWS`
     - `Owner=<gruppo>`

4. **Checklist costi** 🎯 _Sfida_
   - Ordina per urgenza di cleanup:
     - ALB inutilizzato
     - NAT Gateway inutilizzato
     - service ECS sempre acceso
     - vecchie immagini ECR

## Output atteso

- Retention log scelta o impostata.
- Lifecycle policy ECR configurata.
- Tag minimi applicati.

## Checkpoint

- Sai perche log retention e lifecycle policy riducono costi e disordine.
- Sai riconoscere una `orphan resource`.
- Sai quali risorse controllare per prime alla fine del corso.

## Troubleshooting

- **Permesso bloccato**: trasforma il punto in walkthrough concettuale e annota la scelta corretta.
- **Policy ECR non chiara**: crea prima una regola semplice e poi la migliori.

## Cleanup

- Mantieni queste impostazioni se il progetto continua.
- Rimuovi solo regole di test create per errore.
