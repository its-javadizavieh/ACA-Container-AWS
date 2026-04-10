# Lab 03 - AWS: Region, identita e console

## Obiettivo

- Configurare le credenziali CLI del Learner Lab.
- Impostare la Region del corso.
- Trovare in console ECS, ECR, CloudWatch e IAM.

## Durata

30 minuti.

## Prerequisiti

- Account AWS del corso.
- AWS CLI v2 installata.
- Terminale.

## Guida del lab

1. **Copia le credenziali del lab**
   - Learner Lab -> **AWS Details** -> **Show** -> **AWS CLI**.
   - Copia il blocco che parte da `[default]`.
   - Linux / macOS:
     ```bash
     mkdir -p ~/.aws
     nano ~/.aws/credentials
     ```
   - PowerShell:
     ```powershell
     New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.aws" | Out-Null
     notepad "$env:USERPROFILE\.aws\credentials"
     ```
   - In Windows il file deve essere salvato esattamente come `credentials`, non come `credentials.txt`.
   - Se usi **File -> Save As** in Notepad, imposta:
     - `Save as type`: `All Files (*.*)`
     - `File name`: `credentials`
     - `Encoding`: `UTF-8`
   - Il percorso finale deve restare `%USERPROFILE%\.aws\credentials`.
   - `UTF-8` va bene. Anche `ANSI` funziona perche il contenuto e solo testo semplice.
   - Evita `Unicode` / `UTF-16` e qualunque formato non di testo semplice.
   - Il contenuto deve rimanere in formato AWS shared credentials, per esempio:
     ```ini
     [default]
     aws_access_key_id = ASIA...
     aws_secret_access_key = ...
     aws_session_token = ...
     ```
   - Se Notepad salva il file come `.txt` o cambia il formato, AWS CLI non riconosce correttamente le credenziali.

2. **Imposta la Region e verifica l'identita**

   ```bash
   aws configure set region us-east-1
   aws sts get-caller-identity
   ```

   Questo e il controllo piu importante: se fallisce qui, i lab successivi falliranno per lo stesso motivo.

3. **Fai il tour minimo della console**
   - **ECS**: individua `Clusters`, `Task definitions`, `Services`, `Tasks`.
   - **ECR**: individua `Repositories`, `Images`, `Image digest`.
   - **CloudWatch**: individua `Log groups`.
   - **IAM**: apri `Roles` e trova `LabRole`.

4. **Definisci convenzioni base** 🎯 _Sfida_
   - Prefisso risorse: `demo-<gruppo>-...`
   - Tag minimi: `Project=ContainersAWS`, `Owner=<gruppo>`
   - Scrivi la tua Region e il prefisso che userai nei lab.

## Output atteso

- Credenziali CLI configurate.
- Region del corso impostata.
- Sai dove trovare `Stopped reason`, immagini ECR e log CloudWatch.

## Checkpoint

- Sai verificare l'account attivo con `aws sts get-caller-identity`.
- Sai indicare dove leggere Events ECS e Log groups.
- Sai quale ruolo userai nei lab ECS: `LabRole`.

## Troubleshooting

- **`ExpiredToken`**: riapri `AWS Details` e ricopia le credenziali.
- **`AccessDenied`**: controlla di essere nel Learner Lab corretto.
- **Servizi mancanti**: verifica la Region in alto a destra nella console.

## Cleanup

Nessuna risorsa da eliminare.
