# Lab 02 - Docker: build e run locale

## Obiettivo

- Costruire una image Docker dal progetto `hello-api`.
- Avviare il container in locale sulla porta `9090`.
- Verificare endpoint, log e variabili ambiente.

## Durata

30 minuti.

## Prerequisiti

- Docker funzionante.
- Editor di testo o VS Code.
- Puoi usare la cartella `hello-api/` gia presente oppure ricrearla durante il lab.

## Specifiche del laboratorio

- Devi avere a disposizione il codice completo di `hello-api`, non inventarlo durante il lab.
- L'obiettivo e ottenere un container realmente funzionante con gli endpoint `/`, `/health` e `/version`.
- Le verifiche obbligatorie sono tre: risposta HTTP, log visibili e variabile ambiente letta dal container.

## Guida del lab

1. **Crea o verifica i file del progetto**

   ```bash
   mkdir -p hello-api
   cd hello-api
   ls -la
   ```

   Se i file non esistono ancora, crea `app.py` con questo contenuto:

   ```python
   from http.server import HTTPServer, BaseHTTPRequestHandler
   import json, os, sys, logging

   logging.basicConfig(
       level=logging.INFO,
       format="%(asctime)s - %(levelname)s - %(message)s",
       stream=sys.stdout,
   )
   logger = logging.getLogger(__name__)


   class H(BaseHTTPRequestHandler):
       def do_GET(self):
           if self.path == "/health":
               body = json.dumps({"status": "ok"})
               logger.info(f"GET /health from {self.client_address[0]}")
           elif self.path == "/version":
               version = os.environ.get("APP_VERSION", "1.0")
               body = json.dumps({"version": version})
               logger.info(f"GET /version - {version}")
           else:
               body = "Hello from container!"
               logger.info(f"GET {self.path} from {self.client_address[0]}")

           self.send_response(200)
           self.send_header(
               "Content-Type",
               (
                   "application/json"
                   if self.path.startswith("/") and self.path != "/"
                   else "text/plain"
               ),
           )
           self.end_headers()
           self.wfile.write(body.encode())

       def log_message(self, format, *args):
           logger.info(f"HTTP {self.address_string()}: {format % args}")


   try:
       logger.info(f"APP_VERSION={os.environ.get('APP_VERSION', '1.0')}")
       logger.info("Server listening on port 9090")
       HTTPServer(("", 9090), H).serve_forever()
   except Exception as e:
       logger.error(f"Server error: {e}", exc_info=True)
       sys.exit(1)
   ```

   Crea anche `Dockerfile` con questo contenuto:

   ```dockerfile
   FROM python:3.11-slim
   WORKDIR /app
   COPY app.py .
   EXPOSE 9090
   CMD ["python", "app.py"]
   ```

   Prima della build verifica tre punti:
   - l'app ascolta sulla porta `9090`
   - il `Dockerfile` copia `app.py`
   - il `CMD` avvia `python app.py`

2. **Build della image**

   ```bash
   docker build -t hello-api:1.0 .
   ```

   Il punto finale indica che il contesto di build e la cartella corrente. Se sbagli cartella, Docker puo non trovare i file giusti.

3. **Avvio del container**

   ```bash
   docker run -d --name hello-api -p 9090:9090 -e APP_VERSION=1.0 hello-api:1.0
   ```

   In questo comando stai facendo tre cose: avvii il container in background, esponi la porta dell'app e passi una variabile ambiente utile per `/version`.

4. **Verifica applicazione**

   ```bash
   curl http://localhost:9090/
   curl http://localhost:9090/health
   curl http://localhost:9090/version
   ```

   Non limitarti a vedere che risponde: confronta i tre endpoint e annota quale informazione restituisce ciascuno.

5. **Controlla stato e log**

   ```bash
   docker ps
   docker logs hello-api
   docker logs -f hello-api
   docker exec hello-api printenv APP_VERSION
   ```

   `docker ps` conferma che il container e vivo. `docker logs` ti dice se il processo e partito bene. `docker exec` ti permette di verificare l'ambiente reale del container.

   Rispondi: dove vedi che l'app e in ascolto? dove vedi `APP_VERSION`?

6. **Modifica rapida** 🎯 _Sfida_
   - Ferma e rimuovi il container:
     ```bash
     docker stop hello-api && docker rm hello-api
     ```
   - Riavvialo con `APP_VERSION=2.0`:
     ```bash
     docker run -d --name hello-api -p 9090:9090 -e APP_VERSION=2.0 hello-api:1.0
     ```
     Oppure, usando un file `.env`:
     ```bash
     echo "APP_VERSION=2.0" > .env
     docker run -d --name hello-api -p 9090:9090 --env-file .env hello-api:1.0
     ```
   - Verifica che `/version` risponda con il nuovo valore.

   L'obiettivo non e solo cambiare numero: devi capire che la configurazione del container si applica alla creazione, quindi il container va ricreato.

## Output atteso

- Image `hello-api:1.0` costruita correttamente.
- Container in stato `RUNNING`.
- Endpoint raggiungibili su `localhost:9090`.
- Log visibili con `docker logs`.

## Checkpoint

- Sai distinguere image, container e tag.
- Sai ricostruire una image dal Dockerfile.
- Sai verificare variabili ambiente dentro il container.

## Troubleshooting

- **Porta occupata**: usa `-p 9091:9090` oppure libera la porta.
- **Container fermo subito**: controlla `docker logs hello-api`.
- **Versione non cambia**: ricrea il container dopo avere cambiato `APP_VERSION`.

## Cleanup

```bash
docker stop hello-api && docker rm hello-api
```
