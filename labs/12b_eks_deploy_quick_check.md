# Lab 12b — EKS intro: deploy applicazione + quick check

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

4) **Deploy applicazione demo** 🎯 *Sfida*
   - `kubectl create deployment web --image=nginx`
   - *Sfida*: modifica il deployment per usare 2 repliche invece di 1.

5) **Esponi applicazione**
   - `kubectl expose deployment web --type=LoadBalancer --port=80`

6) **Verifica** 🎯 *Sfida*
   - `kubectl get pods`
   - `kubectl get svc`
   - Test via URL del LoadBalancer (se disponibile)
   - *Sfida*: usa `kubectl describe pod <nome>` per trovare l'indirizzo IP del pod.

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

---

## Tutorial consigliati

- [Amazon EKS Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
- [Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

## Soluzioni

<details>
<summary>Sfida Step 4: deployment con 2 repliche</summary>

**Metodo 1 — Dopo la creazione**:
```bash
kubectl scale deployment web --replicas=2
```

**Metodo 2 — Con YAML (più corretto)**:
```bash
kubectl create deployment web --image=nginx --replicas=2 --dry-run=client -o yaml > deployment.yaml
kubectl apply -f deployment.yaml
```

**Verifica**:
```bash
kubectl get pods
# Dovresti vedere 2 pod con nome web-xxxxx-xxxxx
```

**Perché 2 repliche**:
- **Alta disponibilità**: se un pod crasha, l'altro continua a servire traffico
- **Zero-downtime updates**: puoi aggiornare un pod alla volta

</details>

<details>
<summary>Sfida Step 6: trovare IP del pod</summary>

**Comando**:
```bash
kubectl describe pod <nome-pod>
```

**Dove trovare l'IP**:
Cerca la riga `IP:` nell'output:
```
IP:           10.0.1.42
IPs:
  IP:  10.0.1.42
```

**Metodo alternativo (più veloce)**:
```bash
kubectl get pod <nome-pod> -o jsonpath='{.status.podIP}'
```

**Nota importante**: l'IP del pod è **interno** al cluster VPC. Non è raggiungibile da Internet direttamente.

Per accesso esterno servono:
- **Service type LoadBalancer** (crea un ELB AWS)
- **Ingress** (controller NGINX/ALB)
- **NodePort** (per test rapidi)

</details>

 
