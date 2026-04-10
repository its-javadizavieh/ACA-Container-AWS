# Lab 11 - CloudWatch: log, metriche e allarmi

## Obiettivo

- Leggere i log di `hello-api` in CloudWatch.
- Correlare log ed eventi ECS.
- Impostare o descrivere un allarme semplice su CPU o memoria.

## Durata

30 minuti.

## Prerequisiti

- `hello-api-service` attivo.
- Log CloudWatch gia presenti dalla task definition ECS.

## Guida del lab

1. **Apri il log group dell'app**
   - CloudWatch -> `Logs` -> `Log groups`
   - Apri il log group usato da `hello-api`
   - Apri lo stream piu recente

2. **Filtra i log**
   - Cerca manualmente una riga di startup e una richiesta `/health`.
   - Se vuoi usare `Logs Insights`, prova:
     ```text
     fields @timestamp, @message
     | filter @message like /health|error/i
     | sort @timestamp desc
     | limit 20
     ```

3. **Correla con ECS**
   - ECS -> `Services` -> `hello-api-service` -> `Deployments and events`
   - Associa almeno un evento ECS a una riga di log.

4. **Walkthrough allarme**
   - CloudWatch -> `Alarms` -> `Create alarm`
   - `Select metric` -> `ECS` -> `ClusterName, ServiceName`
   - Scegli `CPUUtilization` oppure `MemoryUtilization`
   - Esempio soglia: `> 70` per `5 minutes`
   - Se `cloudwatch:PutMetricAlarm` e bloccato nel lab, fermati prima di `Create alarm` e annota i campi.

5. **Walkthrough retention** 🎯 _Sfida_
   - CloudWatch -> `Log groups` -> `Actions` -> `Edit retention setting`
   - Valore consigliato per dev/test: `7` o `14` giorni
   - Se `logs:PutRetentionPolicy` e bloccato, annota solo il valore scelto.

## Output atteso

- Log di startup e richieste identificati.
- Almeno una correlazione tra evento ECS e log.
- Soglia allarme definita o annotata.

## Checkpoint

- Sai la differenza tra log e metrica.
- Sai dove cercare se un task si riavvia o fallisce.
- Sai quale retention useresti in ambiente dev/test.

## Troubleshooting

- **Nessun log**: verifica la configurazione CloudWatch nella task definition.
- **Metrica assente**: controlla di essere nella Region corretta e sul service giusto.
- **Permessi bloccati**: trasforma il punto in walkthrough concettuale e annota i campi.

## Cleanup

- Elimina l'allarme solo se lo hai creato davvero.
