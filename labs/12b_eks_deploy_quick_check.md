# Lab 12b - EKS quick check: estensione opzionale e confronto con ECS

## Obiettivo

- Leggere il flusso minimo di deploy su EKS.
- Distinguere `Pod`, `Deployment` e `Service`.
- Capire quando scegliere ECS o EKS.

## Durata

20 minuti.

## Prerequisiti

- AWS CLI installata.
- Terminale con permessi per installare o usare `kubectl`.
- `LabEksClusterRole` e `LabEksNodeRole` disponibili in IAM (creati dal template del Learner Lab).
- Questo lab viene dopo il Lab 12 ed e pensato come estensione della stessa lezione.

## Specifiche del laboratorio

- Questo e il secondo esercizio della Lesson 12 ed e separato dal Lab 12 perche cambia orchestratore e obiettivo didattico.
- In questo lab crei un cluster EKS da zero, fai un deploy e poi confronti l'esperienza con ECS.
- Devi associare i comandi ai relativi oggetti (`Deployment`, `Pod`, `Service`).
- Il risultato richiesto e una spiegazione chiara di quando useresti ECS o EKS per lo stesso progetto.
- **Attenzione ai costi**: il control plane EKS costa ~$0.10/ora e i nodi EC2 si aggiungono. Completa il cleanup appena finisci.

## Guida del lab

1. **Installa `kubectl` se manca**

   Se `kubectl` e gia presente, puoi verificarlo con:

   ```bash
   kubectl version --client
   ```

   Se invece manca, usa il comando adatto al tuo sistema operativo.

   **Ubuntu / Debian**

   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   kubectl version --client
   ```

   **macOS (Homebrew)**

   ```bash
   brew install kubectl
   kubectl version --client
   ```

   **Windows (PowerShell)**

   ```powershell
   winget install -e --id Kubernetes.kubectl
   kubectl version --client
   ```

   In alternativa, puoi seguire la guida ufficiale di download per Windows:
   <https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/>

   Se il terminale Windows era gia aperto, chiudilo e riaprilo prima della verifica.

2. **Verifica i tool locali**

   ```bash
   aws --version
   kubectl version --client
   ```

3. **Crea il cluster EKS dalla console**

   - AWS Console → **EKS** → **Clusters** → **Create cluster**
   - **Configuration options**: lascia **Quick configuration (with EKS Auto Mode)**
   - **Cluster name**: `demo-eks`
   - **Kubernetes version**: lascia la versione proposta (es. `1.35`)
   - **Cluster IAM role**: apri il dropdown, filtra per `LabEksClusterRole` e seleziona il ruolo che contiene quel nome (ha un prefisso/suffisso CloudFormation)
   - **VPC**: seleziona la default VPC
   - **Subnets**: lascia almeno 2 subnet selezionate (es. `us-east-1a`, `us-east-1b`)
   - Scorri fino a **Auto Mode Compute**:
     - **Built-in node pools**: lascia `general-purpose` e `system` gia selezionati
     - **Node IAM role**: apri il dropdown — vedrai tre ruoli: uno con `LabEksNodeRole` nel nome, `EMR_EC2_DefaultRole` e `LabRole`. Seleziona quello che contiene **`LabEksNodeRole`** (inizia con un prefisso tipo `c208...LabEksNodeRole-QqwP...`)
   - Clicca **Create**

   > Con Quick configuration non serve creare un node group a parte: EKS Auto Mode gestisce i nodi automaticamente.
   >
   > La creazione del cluster richiede circa **15-20 minuti**. Aspetta che lo stato diventi **Active** prima di proseguire.

4. **Collega kubectl e verifica i nodi**

   ```bash
   aws eks update-kubeconfig --name demo-eks --region us-east-1
   kubectl get nodes
   ```

   Devi vedere 2 nodi in stato `Ready`.

5. **Deploy e expose**

   ```bash
   kubectl create deployment web --image=nginx --replicas=2
   kubectl get pods
   kubectl expose deployment web --type=LoadBalancer --port=80
   kubectl get svc
   ```

   Scrivi accanto a ogni comando cosa crea o modifica.

6. **Compila il confronto ECS vs EKS** 🎯 _Sfida_
   - Unita di deploy
   - Complessita operativa
   - Quando lo useresti per `hello-api`

7. **Definizioni rapide**
   - `Pod`
   - `Deployment`
   - `Service`

## Output atteso

- Cluster EKS creato e funzionante con 2 nodi.
- Deploy nginx eseguito e raggiungibile tramite LoadBalancer.
- Differenze base tra ECS ed EKS annotate.

## Cleanup (obbligatorio per non pagare)

1. Elimina le risorse Kubernetes:

   ```bash
   kubectl delete svc web
   kubectl delete deployment web
   ```

2. In console, elimina il cluster:
   - EKS → **demo-eks** → **Delete cluster** (Auto Mode elimina automaticamente i nodi)

## Checkpoint

- Sai cosa fanno `Pod`, `Deployment` e `Service`.
- Sai quando preferire ECS rispetto a EKS per un progetto piccolo.

## Cleanup

- Se hai usato un cluster reale, elimina solo le risorse create per il test.
- Se il lab e rimasto concettuale, nessun cleanup.
