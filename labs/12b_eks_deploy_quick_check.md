# Lab 12b — EKS intro: deploy applicazione + quick check

Riferimento lezione: `slides_deck/lecture_12_fargate_patterns_hybrid_en.md`

## Obiettivo

- Connettersi a un cluster **EKS** con `kubectl`.
- Deployare una applicazione semplice (Deployment + Service).
- Fare cleanup delle risorse Kubernetes create.

## Durata (timebox)

30 minuti.

## Prerequisiti

- Account AWS con permessi su EKS e accesso Kubernetes (cluster condiviso del docente consigliato).
- Postazione con:
  - AWS CLI
  - `kubectl`
- Attenzione ai costi: **non** creare cluster/nodegroup se non richiesto.

---

## Mini-project (ongoing)

Confronto ECS vs EKS per il tuo progetto.

Deliverable:

- sai motivare quando useresti ECS vs EKS per lo stesso scenario
- (stretch) fai deploy di un’app demo su EKS e fai cleanup completo

---

## Step (numerati)

1) **Verifica tool locali**
   - `aws --version`
   - `kubectl version --client`

2) **Configura kubeconfig**
   - `aws eks update-kubeconfig --name <cluster-name> --region <region>`

3) **Verifica connessione**
   - `kubectl get nodes`

4) **Deploy applicazione demo**
   - `kubectl create deployment web --image=nginx`

5) **Esponi applicazione**
   - `kubectl expose deployment web --type=LoadBalancer --port=80`

6) **Verifica**
   - `kubectl get pods`
   - `kubectl get svc`
   - Test via URL del LoadBalancer (se disponibile)

7) **Verifica competenze EKS (2–3 minuti)**
   - Sai spiegare:
     - differenza tra Pod, Deployment, Service
     - quando scegliere EKS vs ECS

---

## Output atteso

- Deployment e Service creati.
- Applicazione raggiungibile (se LB provisionato) o risorse visibili correttamente.

## Checkpoint

- Sai spiegare differenza tra **Pod**, **Deployment**, **Service**.
- Sai trovare eventi di un pod: `kubectl describe pod ...`.

---

## Troubleshooting rapido

- **Nodes non Ready**: attendi provisioning o verifica nodegroup.
- **Service LoadBalancer senza EXTERNAL-IP**: attendi e verifica subnet tag per ELB.
- **AccessDenied**: controlla IAM e contesto kubeconfig.

---

## Cleanup obbligatorio

1) Elimina risorse Kubernetes create:
   - `kubectl delete svc web`
   - `kubectl delete deployment web`
2) Se il cluster è condiviso: **non eliminarlo**.
3) Verifica che il LoadBalancer sia eliminato dopo `kubectl delete svc`.

---

## Parole chiave Google (screenshot/guide)

- site:docs.aws.amazon.com EKS getting started
- kubectl deployment service loadbalancer example
- EKS subnet tags for load balancer kubernetes.io/role/elb
- AWS EKS console create cluster screenshot
- EKS delete service load balancer cleanup
