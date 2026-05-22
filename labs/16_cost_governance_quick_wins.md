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

## Specifiche del laboratorio

- Questo lab non crea nuove funzionalita applicative: serve a ridurre sprechi e disordine operativo.
- Devi scegliere o impostare valori concreti per retention, lifecycle policy e tagging.
- Alla fine devi saper indicare quali risorse controlleresti per prime durante il cleanup del corso.

## Guida del lab

1. **Retention dei log**
   - CloudWatch -> `Log groups`
   - Apri il log group dell'app -> `Actions` -> `Edit retention setting`
   - Scegli `7` o `14` giorni
   - Se il lab blocca `logs:PutRetentionPolicy`, annota solo il valore scelto.

2. **Lifecycle policy ECR**
   - ECR -> `Repositories` -> `demo-hello-api` -> `Lifecycle policy` -> `Create rule`

   > AWS puo mostrare un avviso sulle azioni distruttive. In un account lab con poche immagini puoi procedere. In produzione usa **Lifecycle Policy Preview** prima di applicare regole che eliminano immagini.

   Crea **due regole** (una alla volta):

   **Regola 1 — elimina immagini senza tag dopo 1 giorno**

   | Campo                              | Valore                               |
   | ---------------------------------- | ------------------------------------ |
   | **Rule priority**                  | `1`                                  |
   | **Rule description** _(opzionale)_ | `Expire untagged images after 1 day` |
   | **Image tag status**               | **Untagged**                         |
   | **Match criteria**                 | **Days since image created**         |
   | **Days before action**             | `1`                                  |
   | **Rule action**                    | **Expire** _(Deletes the image)_     |

   > Dopo **Untagged**, la console **non mostra Image tag filters** (compare solo con **Tagged**). Vedi subito **Match criteria**, **Days before action** e **Rule action**.

   Clicca **Save** (o aggiungi la regola successiva).

   **Regola 2 — mantieni solo le ultime 5 immagini**

   | Campo                              | Valore                           |
   | ---------------------------------- | -------------------------------- |
   | **Rule priority**                  | `2`                              |
   | **Rule description** _(opzionale)_ | `Keep only last 5 images`        |
   | **Image tag status**               | **Any**                          |
   | **Match criteria**                 | **Image count**                  |
   | **Image count**                    | `5`                              |
   | **Rule action**                    | **Expire** _(Deletes the image)_ |

   > Con **Image count**, la console mostra **Image count** al posto di **Days before action**.

   > **Opzioni Image tag status (riferimento):** Tagged (wildcard matching) · Tagged (prefix matching) · Untagged · Any  
   > **Opzioni Match criteria (riferimento):** Days since image created · Days since last recorded pull time · Days since image archived · Image count  
   > **Opzioni Rule action:** **Expire** (elimina) · **Archive** (sposta in archive storage class — non serve in questo lab)

3. **Tagging minimo**
   - ECR -> `Repositories` -> `demo-hello-api` -> tab **Repository tags** -> **Edit**
   - Aggiungi:
     - **Key** `Project` → **Value** `ContainersAWS`
     - **Key** `Owner` → **Value** nome del gruppo, es. `GruppoA` (non usare `<gruppo>` — caratteri `<` `>` non sono validi nei tag AWS)
   - Clicca **Save changes**
   - _(Su security group o risorse ECS la tab si chiama di solito **Tags**.)_

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
- **Errore `Tag parameters are invalid`**: non copiare `<gruppo>` o `<your-group>` — usa un valore reale senza parentesi angolari, es. `GruppoA`.
- **Policy ECR non chiara**: crea prima una regola semplice e poi la migliori.
- **Non vedi Image tag filters**: normale se hai scelto **Untagged** o **Any**; i filtri compaiono soprattutto con **Tagged (wildcard/prefix matching)**.

## Cleanup

- Mantieni queste impostazioni se il progetto continua.
- Rimuovi solo regole di test create per errore.
