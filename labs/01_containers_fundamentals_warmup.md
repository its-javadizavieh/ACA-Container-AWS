# Lab 01 - Warm-up: container e immagini

## Obiettivo

- Verificare che Docker funzioni.
- Distinguere image, container, tag e digest.
- Preparare l'ambiente per i lab successivi.

## Durata

20 minuti.

## Prerequisiti

- Docker installato.
- Terminale e accesso Internet.
- Docker avviato correttamente.

## Specifiche del laboratorio

- In questo lab non devi scrivere codice: devi osservare e descrivere i concetti base.
- Ogni comando va usato per distinguere chiaramente `image`, `container`, `tag` e `digest`.
- Alla fine devi saper spiegare con un esempio perche `latest` e meno affidabile di un digest.

## Guida del lab

1. **Verifica Docker**

   ```bash
   docker version
   ```

   Atteso: sezione Client e Server visibili.

2. **Esegui il test base**

   ```bash
   docker run --rm hello-world
   ```

   Questo passaggio ti fa vedere il ciclo completo: download della image, creazione del container temporaneo ed esecuzione.

3. **Guarda image e container**

   ```bash
   docker images | head
   docker ps -a | head
   ```

   Rispondi: `hello-world` e il nome della image o del container?

4. **Tag vs digest**

   ```bash
   docker pull alpine:3.19
   docker image inspect alpine:3.19 --format '{{index .RepoDigests 0}}'
   ```

   Qui non devi solo copiare il digest: devi capire perche il tag e comodo per lavorare, mentre il digest e piu affidabile per ambienti riproducibili.

5. **Domande rapide** 🎯 _Sfida_
   - Perche `latest` e rischioso?
   - Quando useresti un digest invece di un tag?

## Output atteso

- Docker operativo sulla macchina.
- Test `hello-world` completato.
- Digest di `alpine:3.19` identificato.

## Checkpoint

- Sai distinguere image e container.
- Sai spiegare tag vs digest.
- Sai leggere `docker images` e `docker ps -a`.

## Troubleshooting

- **Permission denied su `docker.sock`**: su Linux aggiungi l'utente al gruppo `docker` o usa `sudo` se il docente lo consente.
- **Server Docker assente**: avvia Docker Desktop oppure il servizio Docker.

## Cleanup

```bash
docker image rm hello-world alpine:3.19
```
