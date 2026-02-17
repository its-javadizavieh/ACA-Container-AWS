# Lab 02 — Docker: build & run (locale)

## Obiettivo

- Comprendere differenza tra **immagine** e **container**.
- Creare una **Docker image** da Dockerfile.
- Eseguire il container, esporre una porta, leggere i log e fare troubleshooting base.

## Durata (timebox)

30 minuti.

## Prerequisiti

- PC con Docker installato (Docker Desktop o Docker Engine).
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

1) **Verifica installazione Docker**
   - Comando: `docker version`

2) **Crea una cartella di lavoro**
   - Esempio: `containers-lab-02/`

3) **Crea un file app minimale**
   - Opzione A (Python): un server HTTP minimale.
   - Opzione B (Node): un server HTTP minimale.
   - Nota: se sei in ritardo, usa il template del docente.

4) **Scrivi un Dockerfile** 🎯 *Sfida*
   - Base image "slim"
   - Copia dei file
   - Porta esposta
   - Comando di avvio
   - *Sfida*: usa multi-stage build se possibile (o almeno minimizza i layer).

5) **Build dell'immagine**
   - Comando: `docker build -t hello-api:1.0 .`

6) **Run del container**
   - Comando: `docker run --rm -p 8080:8080 hello-api:1.0`

7) **Verifica da browser**
   - URL: `http://localhost:8080`

8) **Controlla log e stato** 🎯 *Sfida*
   - `docker ps`
   - `docker logs <container_id>`
   - *Sfida*: trova una riga nei log che conferma che il server è pronto.

---

## Output atteso

- Un container in esecuzione che risponde su `localhost:8080`.
- Log visibili dal terminale o via `docker logs`.

## Checkpoint

- Sai spiegare: *cos’è un’immagine? cos’è un container? cos’è un tag?*
- Sai ricostruire l’immagine da Dockerfile.

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

- [Docker Workshop — Our application](https://docs.docker.com/get-started/workshop/02_our_app/)
- [Build and push your first image](https://docs.docker.com/get-started/introduction/build-and-push-first-image/)
- [Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/)
- [Build, tag, and publish an image](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/)

---

## Soluzioni

<details>
<summary>🎯 Sfida 4: Esempio Dockerfile (Python)</summary>

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["python", "app.py"]
```

Punti chiave:

- `FROM`: base image leggera (`-slim`)
- `WORKDIR`: directory di lavoro
- `COPY` + `RUN pip install`: prima le dipendenze (cache layer)
- `EXPOSE`: documenta la porta (non la pubblica)
- `CMD`: comando di avvio

</details>

<details>
<summary>🎯 Sfida 8: Differenza tra -d e --rm</summary>

- `--rm`: rimuove il container automaticamente quando termina (usa in sviluppo)
- `-d` (detached): esegue in background, il container rimane attivo

Per vedere i log di un container in background:

```bash
docker ps                    # trova CONTAINER ID
docker logs <container_id>   # leggi i log
docker logs -f <container_id>  # segui i log in tempo reale
```

</details>
