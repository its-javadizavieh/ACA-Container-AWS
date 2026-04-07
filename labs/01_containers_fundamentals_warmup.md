# Lab 01 - Warm-up: concetti container + checklist ambiente

## Obiettivo

- Verificare che l’ambiente di lavoro sia pronto (Docker + terminale).
- Allineare il linguaggio tecnico della lezione: **container vs VM**, **immagine vs container**, **tag vs digest**, **registry**.

## Durata (timebox)

30 minuti.

## Prerequisiti

- PC con accesso Internet.
- Terminale e editor (VS Code).
- Docker installato — segui le istruzioni per il tuo sistema operativo qui sotto.

---

## Step 0 - Installazione Docker

### Ubuntu (22.04 / 24.04)

```bash
# Rimuovi eventuali versioni vecchie
sudo apt-get remove -y docker docker-engine docker.io containerd runc 2>/dev/null

# Installa prerequisiti
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# Aggiungi la chiave GPG e il repository ufficiale Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installa Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Aggiungi il tuo utente al gruppo docker (evita sudo)
sudo usermod -aG docker $USER
newgrp docker
```

### macOS

```bash
# Opzione A: scarica Docker Desktop da https://www.docker.com/products/docker-desktop/
# Scegli "Mac with Apple chip" o "Mac with Intel chip", installa il .dmg, avvia Docker Desktop.

# Opzione B: installa via Homebrew
brew install --cask docker
# Poi avvia Docker Desktop da Applicazioni.
```

### Windows

```powershell
# 1. Abilita WSL 2 (apri PowerShell come Amministratore)
wsl --install
# Riavvia il PC se richiesto.

# 2. Scarica e installa Docker Desktop da:
#    https://www.docker.com/products/docker-desktop/
#    Durante l'installazione, spunta "Use WSL 2 instead of Hyper-V".

# 3. Avvia Docker Desktop e aspetta che il motore si avvii.
```

### Verifica (tutti i sistemi)

```bash
docker version
```

Deve mostrare sia **Client** che **Server**. Se manca il Server, Docker Desktop non è avviato.

---

## Scenario

Prima di iniziare la parte AWS, ci assicuriamo che tutti abbiano un ambiente funzionante e la stessa terminologia.

---

## Mini-project (ongoing)

Oggi definisci lo scope del progetto (senza coding): una piccola HTTP API (“hello-api”).

Deliverable (nota breve, 5 minuti):

- endpoint da implementare: `/`, `/health`, `/version`
- cosa significa “healthy” per la tua app

---

## Step (numerati)

1. **Check Docker installato**
   - Comando: `docker version`
   - Output atteso: versione client (e server se disponibile).

2. **Check che puoi eseguire container**
   - Comando: `docker run --rm hello-world`
   - Output atteso: messaggio “Hello from Docker!” (o equivalente).

3. **Capire cosa è stato eseguito** 🎯 _Sfida_
   - Domanda: `hello-world` è una _image_ o un _container_?
   - Risposta attesa: `hello-world` è il nome dell'immagine; il run crea un container temporaneo.
   - _Sfida_: spiega la differenza tra image e container.

4. **Elenca immagini e container**
   - `docker images | head`
   - `docker ps`
   - `docker ps -a | head`

5. **Tag vs digest (concetto)** 🎯 _Sfida_
   - Esegui:
     - `docker pull alpine:3.19`
     - `docker image inspect alpine:3.19 | grep -i digest -n || true`
   - Nota: il digest è un riferimento immutabile (se presente nell'output).
   - _Sfida_: spiega perché il digest è più sicuro del tag.

---

## Output atteso

- Docker funzionante su tutte le postazioni.
- Esecuzione di un container di prova riuscita.
- Concetti minimi condivisi.

## Checkpoint

- Sai distinguere **image** vs **container**.
- Sai dire perché `latest` è rischioso.

---

## Troubleshooting rapido

- **Permission denied / docker.sock**: su Linux verifica gruppo `docker` o usa `sudo` (solo se consentito dal docente).
- **Docker daemon non avviato**: avvia Docker Desktop o `systemctl start docker`.
- **Download lento**: verifica rete/proxy.

---

## Cleanup obbligatorio

- Nessuna risorsa AWS.
- (Opzionale) Se vuoi pulire immagini scaricate:
  - `docker image rm hello-world alpine:3.19` (solo se non servono).

---

## Parole chiave Google (screenshot/guide)

- docker run hello-world output screenshot
- docker images vs docker ps difference
- docker tag vs digest explanation
- OCI image digest meaning
- docker permission denied docker.sock fix
- docker daemon not running fix

---

## Tutorial consigliati

- [Docker Getting Started - What is a container?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)
- [Docker Getting Started - What is an image?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)
- [Docker Getting Started - What is a registry?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-registry/)

---

## Soluzioni

<details>
<summary>🎯 Sfida 3: hello-world è image o container?</summary>

`hello-world` è il **nome dell'immagine**. Quando esegui `docker run`, Docker:

1. Scarica l'immagine (se non presente)
2. Crea un **container** (istanza) da quell'immagine
3. Esegue il container (che stampa il messaggio e termina)

Il container è temporaneo (`--rm` lo rimuove automaticamente).

</details>

<details>
<summary>🎯 Sfida 5: Perché il digest è più sicuro del tag?</summary>

- **Tag** (es. `alpine:3.19`): può essere spostato su un'immagine diversa. Qualcuno potrebbe fare push di una nuova versione con lo stesso tag.
- **Digest** (es. `sha256:abc123...`): hash crittografico del contenuto. È **immutabile** - se il contenuto cambia, il digest cambia.

In production, usa il digest per garantire che stai eseguendo esattamente l'immagine testata.

</details>
