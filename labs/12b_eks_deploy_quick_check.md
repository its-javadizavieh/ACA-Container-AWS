# Lab 12b - EKS quick check: deploy minimo e confronto con ECS

## Obiettivo

- Leggere il flusso minimo di deploy su EKS.
- Distinguere `Pod`, `Deployment` e `Service`.
- Capire quando scegliere ECS o EKS.

## Durata

20 minuti.

## Prerequisiti

- AWS CLI e `kubectl` installati.
- Nel Learner Lab `eks:*` e spesso bloccato: il lab e concettuale salvo cluster fornito dal docente.

## Guida del lab

1. **Verifica i tool locali**

   ```bash
   aws --version
   kubectl version --client
   ```

2. **Leggi il flusso base di comandi**

   ```bash
   aws eks update-kubeconfig --name <cluster-name> --region <region>
   kubectl create deployment web --image=nginx
   kubectl scale deployment web --replicas=2
   kubectl expose deployment web --type=LoadBalancer --port=80
   ```

   Scrivi accanto a ogni comando cosa crea o modifica.

3. **Se hai un cluster fornito dal docente, verifica lo stato**

   ```bash
   kubectl get nodes
   kubectl get deploy,po,svc
   ```

   Se non hai un cluster, limita il lavoro al walkthrough del demo 12.

4. **Compila il confronto ECS vs EKS** 🎯 _Sfida_
   - Unita di deploy
   - Complessita operativa
   - Quando lo useresti per `hello-api`

5. **Definizioni rapide**
   - `Pod`
   - `Deployment`
   - `Service`

## Output atteso

- Flusso minimo di deploy EKS compreso.
- Differenze base tra ECS ed EKS annotate.

## Checkpoint

- Sai cosa fanno `Pod`, `Deployment` e `Service`.
- Sai quando preferire ECS rispetto a EKS per un progetto piccolo.

## Cleanup

- Se hai usato un cluster reale, elimina solo le risorse create per il test.
- Se il lab e rimasto concettuale, nessun cleanup.
