# Cheat Sheet — Docker, ECS & Kubernetes Formats

**ITS Academy — Containers on AWS** | Keep this sheet for reference

---

## Dockerfile (how to build an image)

```dockerfile
FROM python:3.11-slim        # base image (OS + runtime)
WORKDIR /app                 # working dir inside container
COPY requirements.txt .      # copy deps list (cached layer)
RUN pip install -r requirements.txt  # install deps
COPY . .                     # copy app source
EXPOSE 9090                  # document the port (metadata only)
CMD ["python", "app.py"]     # default start command
```

| Instruction  | What it does         | Example                    |
| ------------ | -------------------- | -------------------------- |
| `FROM`       | Base image           | `FROM node:20-alpine`      |
| `WORKDIR`    | Set working dir      | `WORKDIR /app`             |
| `COPY`       | Copy files → image   | `COPY . .`                 |
| `RUN`        | Run during **build** | `RUN npm install`          |
| `EXPOSE`     | Document port        | `EXPOSE 9090`              |
| `ENV`        | Set env variable     | `ENV NODE_ENV=production`  |
| `CMD`        | Default run cmd      | `CMD ["node","server.js"]` |
| `ENTRYPOINT` | Fixed run cmd        | `ENTRYPOINT ["python"]`    |

**Build:** `docker build -t my-app:1.0 .`  
**Run:** `docker run -d -p 9090:9090 my-app:1.0`

---

## Image layers (how caching works)

```
┌──────────────────────────┐
│ CMD ["python","app.py"]  │ ← metadata (no layer)
├──────────────────────────┤
│ COPY . .                 │ ← Layer 4: your code (changes often)
├──────────────────────────┤
│ RUN pip install ...      │ ← Layer 3: deps (cached if unchanged)
├──────────────────────────┤
│ COPY requirements.txt    │ ← Layer 2: dep file
├──────────────────────────┤
│ python:3.11-slim         │ ← Layer 1: base image
└──────────────────────────┘
```

- Only **changed layers** rebuild → put things that change least at the top.
- Images are **immutable** → never edit, always rebuild.

---

## ECS Task Definition (JSON)

```json
{
  "family": "hello-api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256", // 0.25 vCPU
  "memory": "512", // 0.5 GB
  "executionRoleArn": "arn:aws:iam::<acct>:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "hello-api",
      "image": "<acct>.dkr.ecr.<region>.amazonaws.com/hello-api:1.0",
      "portMappings": [{ "containerPort": 9090 }],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/hello-api",
          "awslogs-region": "eu-west-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

> You can also create this via the **ECS Console wizard** (no JSON needed).

---

## Kubernetes manifests (YAML)

### Deployment (keeps N pods running — like ECS Service)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-api
spec:
  replicas: 2
  selector:
    matchLabels: { app: hello-api }
  template:
    metadata:
      labels: { app: hello-api }
    spec:
      containers:
        - name: hello-api
          image: hello-api:1.0
          ports: [{ containerPort: 9090 }]
          resources:
            requests: { cpu: "250m", memory: "512Mi" }
```

### Service (expose pods — like ALB + Target Group)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-api-svc
spec:
  selector: { app: hello-api }
  ports:
    - port: 80
      targetPort: 9090
  type: LoadBalancer
```

---

## ECS vs Kubernetes — side by side

| Concept        | ECS (Fargate)              | Kubernetes (EKS)            |
| -------------- | -------------------------- | --------------------------- |
| Image recipe   | Dockerfile                 | Dockerfile (**same**)       |
| Image format   | OCI layers                 | OCI layers (**same**)       |
| Run unit       | **Task**                   | **Pod**                     |
| Keep N running | **Service** (Console/JSON) | **Deployment** (YAML)       |
| Expose traffic | **ALB + Target Group**     | **Service + Ingress**       |
| Config format  | **JSON**                   | **YAML**                    |
| Registry       | ECR / Docker Hub           | ECR / Docker Hub (**same**) |

> **Key insight:** the Docker image is the same everywhere. Only the orchestrator config changes.

---

## Essential commands

```bash
# --- Docker ---
docker build -t app:1.0 .              # build image from Dockerfile
docker run -d -p 9090:9090 app:1.0     # run container (background)
docker ps                               # list running containers
docker logs <name>                      # read container logs
docker exec -it <name> sh               # shell into container
docker stop <name> && docker rm <name>  # stop + remove
docker push <registry>/app:1.0          # push to registry

# --- AWS ECR ---
aws ecr get-login-password --region $R | docker login --username AWS --password-stdin $ACCT.dkr.ecr.$R.amazonaws.com
docker tag app:1.0 $ACCT.dkr.ecr.$R.amazonaws.com/app:1.0
docker push $ACCT.dkr.ecr.$R.amazonaws.com/app:1.0

# --- Kubernetes (kubectl) ---
kubectl apply -f deployment.yaml        # create/update resources
kubectl get pods                         # list pods
kubectl logs <pod-name>                  # read logs
kubectl delete -f deployment.yaml        # cleanup
```

---

## Fargate CPU/Memory combos (valid pairs)

| CPU (vCPU) | Memory options |
| ---------- | -------------- |
| 0.25       | 0.5, 1, 2 GB   |
| 0.5        | 1, 2, 3, 4 GB  |
| 1          | 2–8 GB         |
| 2          | 4–16 GB        |
| 4          | 8–30 GB        |

---

## Debug checklist (when things don't work)

1. **ECS Events tab** → was the task placed?
2. **Task → Stopped reason** → why did it fail?
3. **CloudWatch Logs** → what did the app print?
4. **Security Group** → is the port open? Is the source correct?
5. **Health check** → is the path right? Is the app listening on that port?

> Debug order: **Events → Stopped reason → Logs → SG → Health check**
