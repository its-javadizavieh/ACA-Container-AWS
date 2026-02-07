# Lab 01 — Warm-up: concetti container + checklist ambiente

Riferimento lezione: `slides_deck/lecture_01_containers_fundamentals_en.md`

## Obiettivo

- Verificare che l’ambiente di lavoro sia pronto (Docker + terminale).
- Allineare il linguaggio tecnico della lezione: **container vs VM**, **immagine vs container**, **tag vs digest**, **registry**.

## Durata (timebox)

30 minuti.

## Prerequisiti

- PC con accesso Internet.
- Docker installato (Docker Desktop o Docker Engine).
- Terminale e editor (VS Code).

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

1) **Check Docker installato**
   - Comando: `docker version`
   - Output atteso: versione client (e server se disponibile).

2) **Check che puoi eseguire container**
   - Comando: `docker run --rm hello-world`
   - Output atteso: messaggio “Hello from Docker!” (o equivalente).

3) **Capire cosa è stato eseguito**
   - Domanda: `hello-world` è una *image* o un *container*?
   - Risposta attesa: `hello-world` è il nome dell’immagine; il run crea un container temporaneo.

4) **Elenca immagini e container**
   - `docker images | head`
   - `docker ps`
   - `docker ps -a | head`

5) **Tag vs digest (concetto)**
   - Esegui:
     - `docker pull alpine:3.19`
     - `docker image inspect alpine:3.19 | grep -i digest -n || true`
   - Nota: il digest è un riferimento immutabile (se presente nell’output).

6) **Mini-esercizio (5 minuti)**
   - Completa a voce:
     - “Un’immagine è ________”
     - “Un container è ________”
     - “Un registry serve per ________”
     - “Un tag è ________, un digest è ________”

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
