# Lab 02 - Docker: build & run (locale)

## Obiettivo

- Comprendere differenza tra **immagine** e **container**.
- Creare una **Docker image** da Dockerfile.
- Eseguire il container, esporre una porta, leggere i log e fare troubleshooting base.

## Durata (timebox)

30 minuti.

## Prerequisiti

- PC con Docker installato (vedi **Lab 01, Step 0** per Ubuntu/Mac/Windows).
- Terminale e editor (VS Code).
- Connessione Internet.

## Scenario

Costruiamo una piccola app HTTP “hello” e la eseguiamo in container in locale.

---

## Mini-project (ongoing)

Implementa la prima versione eseguibile in locale.

Deliverable:

- la tua “hello-api” gira localmente in un container
- puoi chiamare `/health` dal browser
- Dockerfile + `.dockerignore` presenti (no secrets)

---

## Step (numerati)

1. **Verifica installazione Docker**
   - Comando: `docker version`

2. **Crea una cartella di lavoro**
   - Esempio: `containers-lab-02/`

3. **Crea un file app minimale**
   - Opzione A (Python): un server HTTP minimale.
   - Opzione B (Node): un server HTTP minimale.
   - Nota: se sei in ritardo, usa il template del docente.
   - ⚠️ **Importante**: l'app DEVE loggare su stdout usando `logging` o `console.log()` per essere visibile in `docker logs`

4. **Crea un file `.env`** 🎯 _Sfida env_
   - Formato: `KEY=VALUE` (una variabile per riga)
   - Esempio:
     ```
     APP_VERSION=2.5
     APP_ENV=development
     ```
   - **NOTA:** Aggiungi `.env` a `.gitignore` (secrets!)

5. **Scrivi un Dockerfile** 🎯 _Sfida_
   - Base image "slim"
   - Copia dei file
   - Porta esposta
   - Comando di avvio
   - _Sfida_: usa multi-stage build se possibile (o almeno minimizza i layer).

6. **Build dell'immagine**
   - Comando: `docker build -t hello-api:1.0 .`

7. **Run del container (carica .env)** 🎯 _Sfida env_
   - Comando: `docker run -d --name hello-api --env-file .env -p 8080:8080 hello-api:1.0`
   - Oppure: `docker run -d --name hello-api --env APP_VERSION=2.5 -p 8080:8080 hello-api:1.0`

8. **Verifica da browser**
   - URL: `http://localhost:8080`

9. **Controlla log e env var** 🎯 _Sfida_
   - `docker ps`
   - `docker logs hello-api` → deve mostrare i log di startup
   - `docker logs -f hello-api` → segui i log in tempo reale
   - `docker exec hello-api env | grep APP` → verifica che la env var è stata caricata
   - _Sfida_: trova una riga nei log che conferma che il server è pronto E che mostra il valore di APP_VERSION.

---

## Output atteso

- Un container in esecuzione che risponde su `localhost:8080`.
- Log visibili dal terminale o via `docker logs`.
- Env var `APP_VERSION` visibile nei log di startup.
- Comando `docker exec hello-api env` mostra `APP_VERSION=2.5`.

## Checkpoint

- Sai spiegare: _cos'è un'immagine? cos'è un container? cos'è un tag?_
- Sai ricostruire l'immagine da Dockerfile.
- Sai passare una `.env` file al container: `--env-file .env`
- Sai verificare che i log appaiono via `docker logs` e che le env var sono caricate.

---

## Troubleshooting rapido

- **Porta occupata**: cambia mapping (`-p 8081:8080`) oppure chiudi il processo che usa la porta.
- **Errore “command not found”**: controlla `CMD`/`ENTRYPOINT` e path.
- **Build lenta**: verifica connessione e usa base image più piccola.

---

## Cleanup obbligatorio

(Nessuna risorsa AWS. Cleanup solo locale.)

- Ferma il container (CTRL+C) oppure lascia `--rm` per auto-rimozione.
- Rimuovi immagini inutili:
  - `docker images`
  - `docker rmi hello-api:1.0` (se serve)

---

## Parole chiave Google (screenshot/guide)

- docker build run tutorial
- dockerfile best practices multi stage
- docker logs docker exec troubleshooting
- .dockerignore example
- docker expose vs publish port explanation

---

## Tutorial consigliati

- [Docker Workshop - Our application](https://docs.docker.com/get-started/workshop/02_our_app/)
- [Build and push your first image](https://docs.docker.com/get-started/introduction/build-and-push-first-image/)
- [Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/)
- [Build, tag, and publish an image](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/)

---

## Soluzioni

<details>
<summary>🎯 Sfida 3: Esempio app.py (Python con logging e env)</summary>

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json, os, sys, logging

# Configure logging to ensure all output goes to stdout/stderr
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    stream=sys.stdout  # Explicitly send logs to stdout so docker logs captures them
)
logger = logging.getLogger(__name__)

class H(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/health":
            body = json.dumps({"status": "ok"})
            logger.info(f"GET /health from {self.client_address[0]}")
        else:
            body = "Hello from container!"
            logger.info(f"GET {self.path} from {self.client_address[0]}")

        self.send_response(200)
        self.send_header('Content-Type', 'application/json' if self.path in ['/health', '/version'] else 'text/plain')
        self.end_headers()
        self.wfile.write(body.encode())

    def log_message(self, format, *args):
        # Override to ensure HTTP logs go to logger (captured by docker logs)
        logger.info(f"HTTP: {format % args}")

try:
    app_version = os.environ.get('APP_VERSION', '1.0')
    logger.info(f"Starting server - APP_VERSION={app_version}")
    logger.info("Server listening on port 8080")
    HTTPServer(("", 8080), H).serve_forever()
except Exception as e:
    logger.error(f"Server error: {e}", exc_info=True)  # exc_info=True includes full traceback
    sys.exit(1)
```

**Punti chiave per stdout/stderr:**

- `logging.basicConfig(stream=sys.stdout)`: Dirige TUTTI i log a stdout
- `exc_info=True`: Include traceback nei log di errore (visibile via `docker logs`)
- Questo assicura che `docker logs` catturi OGNI messaggio

</details>

<details>
<summary>🎯 Sfida 4: Esempio Dockerfile (Python)</summary>

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app.py .

EXPOSE 8080

CMD ["python", "app.py"]
```

Punti chiave:

- `FROM`: base image leggera (`-slim`)
- `WORKDIR`: directory di lavoro
- `COPY app.py`: copia il file app
- `EXPOSE`: documenta la porta (non la pubblica)
- `CMD`: comando di avvio in exec form (consigliato per logs)

> **Nota su stdout/stderr:** Il comando in forma `["python", "app.py"]` (exec form) preserva stdout/stderr direttamente. Evita `/bin/sh -c` che potrebbe intercettarli.

</details>

<details>
<summary>🎯 Sfida 8: Verifica env variables e logs</summary>

### ✅ Verifica che i log vengono catturati:

```bash
# Costruisci l'immagine
docker build -t hello-api:1.0 .

# Avvia il container
docker run -d --name hello-api -p 8080:8080 hello-api:1.0

# Leggi i log (dovrai vedere il startup message!)
docker logs hello-api
# Output atteso:
# 2024-03-17 10:15:32,123 - INFO - Starting server - APP_VERSION=1.0
# 2024-03-17 10:15:32,124 - INFO - Server listening on port 8080

# Segui i log in tempo reale
docker logs -f hello-api
```

### ✅ Verifica che le env variables vengono passate:

```bash
# Crea un .env file (NON lo aggiungere a git!)
cat > .env << EOF
APP_VERSION=2.5
APP_ENV=development
EOF

# Ferma il container precedente
docker stop hello-api && docker rm hello-api

# Avvia il container CON il .env file
docker run -d --name hello-api --env-file .env -p 8080:8080 hello-api:1.0

# Verifica che la env var è stata caricata
docker logs hello-api
# Output atteso:
# APP_VERSION=2.5

# Verifica che il container ha effettivamente la env var
docker exec hello-api env | grep APP_VERSION
# Output: APP_VERSION=2.5
```

### ✅ Debugging:

- **I log non appaiono?** Riavvia il container e usa `docker logs -f hello-api`
- **Env var vuota?** Verifica che `.env` non sia vuoto e usi formato `KEY=VALUE`
- **Container crasha?** Usa `docker logs hello-api` per vedere l'errore completo

</details>

<details>
<summary>🎯 Sfida 6: Differenza tra -d e --rm</summary>

- `--rm`: rimuove il container automaticamente quando termina (usa in sviluppo)
- `-d` (detached): esegue in background, il container rimane attivo

Per vedere i log di un container in background:

```bash
docker ps                    # trova CONTAINER ID
docker logs <container_id>   # leggi i log
docker logs -f <container_id>  # segui i log in tempo reale
```

</details>

---

## 📚 Guida Completa: Stdout/Stderr e Variabili d'Ambiente

### 🔴 Problema: i log non appaiono in `docker logs`

#### Cause comuni:

1. **App usa `print()` senza flush**
   - `print()` potrebbe essere bufferato
   - Soluzione: usa `print(..., flush=True)` oppure configura logging

2. **App non loga affatto**
   - Soluzione: aggiungi `logging` con `stream=sys.stdout`

3. **CMD non è in exec form**
   - ❌ Errato: `CMD "python app.py"` (shell form)
   - ✅ Corretto: `CMD ["python", "app.py"]` (exec form)

#### ✅ Soluzione - app.py completo:

```python
import logging, sys, os

# Setup logging to stdout so docker logs can capture everything
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s',
    stream=sys.stdout,  # ← IMPORTANTE: indirizza a stdout
    force=True  # ← Sovrascrive handler precedenti
)
logger = logging.getLogger(__name__)

class MyApp:
    def __init__(self):
        logger.info(f"App started - ENV: {os.environ.get('APP_ENV', 'unknown')}")

    def process(self, request):
        logger.info(f"Processing: {request}")
        return "OK"

app = MyApp()
app.process("test")
```

#### ✅ Soluzione - Dockerfile:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
# ← NO /bin/sh -c intercept, use exec form:
CMD ["python", "app.py"]
```

#### ✅ Verifica:

```bash
docker build -t myapp .
docker run -d --name myapp myapp
docker logs -f myapp
# Dovresti vedere subito il log "App started"
```

---

### 🔐 Problema: Environment variables non vengono caricate

#### Cause comuni:

1. **Container lanciato SENZA `--env-file`**

   ```bash
   ❌ docker run myapp  # .env non viene caricato!
   ✅ docker run --env-file .env myapp
   ```

2. **File `.env` non esiste o ha percorso sbagliato**

   ```bash
   # Verifica che .env esiste nel directory di lavoro:
   ls -la .env
   ```

3. **Sintassi `.env` sbagliata**

   ```bash
   ❌ Sbagliato:  APP_VERSION = 1.0  (spazi!)
   ✅ Corretto:   APP_VERSION=1.0

   ❌ Sbagliato:  export APP_VERSION=1.0
   ✅ Corretto:   APP_VERSION=1.0
   ```

4. **File `.env` è nella `.dockerignore` o nel build context**
   ```dockerfile
   # ❌ SBAGLIATO: aggiungere .env in COPY
   # COPY .env .  ← NO! .env dovrebbe restare fuori dall'immagine
   # .env è un file locale per lo sviluppo, non per il container
   ```

#### ✅ Workflow Completo:

**Step 1: Crea `.env`**

```bash
cat > .env << 'EOF'
APP_VERSION=2.0
DATABASE_URL=localhost:5432
DEBUG=true
EOF
```

**Step 2: Verifica formato**

```bash
cat .env
# Output:
# APP_VERSION=2.0
# DATABASE_URL=localhost:5432
# DEBUG=true
```

**Step 3: Aggiungi a `.gitignore`**

```bash
echo ".env" >> .gitignore
```

**Step 4: App legge env var**

```python
import os
version = os.environ.get('APP_VERSION', 'default')
logger.info(f"Running version: {version}")
```

**Step 5: Build & Run con --env-file**

```bash
docker build -t myapp .
docker run -d --name myapp --env-file .env -p 8080:8080 myapp
```

**Step 6: Verifica caricamento**

```bash
# Metodo 1: Leggi i log
docker logs myapp
# Atteso: "Running version: 2.0"

# Metodo 2: Elenca tutte le env var
docker exec myapp env | sort
# Dovresti vedere APP_VERSION=2.0 tra le righe

# Metodo 3: Verifica una specifica var
docker exec myapp printenv APP_VERSION
# Output: 2.0
```

---

### 📊 Tabella Riepilogativa: Passare Env Var al Container

| Metodo                      | Comando                          | Uso Ideale             | Nota                      |
| --------------------------- | -------------------------------- | ---------------------- | ------------------------- |
| **File `.env`**             | `docker run --env-file .env IMG` | Sviluppo locale        | Tutte le var in una volta |
| **Single `--env`**          | `docker run --env KEY=VAL IMG`   | Override singolo       | Utile in CI/CD            |
| **Hardcoded in Dockerfile** | `ENV KEY=value` in Dockerfile    | Defaults di produzione | Visibility nei logs       |
| **Secrets (AWS/Docker)**    | `docker run --secret SEC IMG`    | Produzione con secrets | Sicuro, non nei logs      |

---

### 🧠 Checklist: Debugging Log & Env Var

- [ ] App usa `logging.basicConfig(stream=sys.stdout)`
- [ ] Dockerfile usa exec form: `CMD ["python", "app.py"]`
- [ ] `.env` file esiste e ha formato `KEY=VALUE`
- [ ] Container avviato con `--env-file .env`
- [ ] `docker logs <container>` mostra startup message
- [ ] `docker exec <container> env | grep APP` mostra le var caricate
- [ ] `docker logs -f <container>` segue i log in tempo reale
