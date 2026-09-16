# Hier alle im Video erwähnten Befehle:

## Docker installieren:

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

Falls kein curl installiert: `apt install curl`

Wenn man nicht mit dem Nutzer Root arbeitet, sollte man den aktuellen Benutzer berechtigen:

```
sudo usermod -aG docker $USER
```

## Mit Docker arbeiten

* Laufende Container auflisten: `docker ps`
* Alle Container auflisten (auch gestoppte): `docker ps -a`
* Einen Container anhalten: `docker stop <Containername>` (den Namen findet man mit `docker ps` heraus)
* Einen gestoppten Container endgültig löschen: `docker rm <Containername>`

## Einen simplen Webserver starten

Der Container aus dem Image nginx fährt mit folgendem Befehl hoch:

```
docker run -p 80:80 nginx
```
Die eigene IP-Adresse erhält man mit `ip a`

## Arbeiten mit Docker-Compose

Legt euch am besten einen eigenen Ordner für das Docker-Projekt an, um Ordnung zu halten. Die Datei docker-compose.yml enthält die Definition der Container.

Bearbeitet wird die Datei mit:

```
nano docker-compose.yml
```

Die Inhalte findet ihr unten in diesem GitHub-Gist. Den Texteditor Nano beendet man mit: `Strg+X`, dann `Y`

Die Compose-Zusammenstellung hochfahren:

```
docker compose up -d
```

Will man die Container updaten, lädt man die neuen Images mit 

```
docker compose pull
```

## Eine oder mehrere Docker-Compose-Dateien?

Das ist definitiv Geschmachssache und hängt von der Umgebung ab. Wenn man mehr als ein Projekt (zum Beispiel einen Blog und ein Pihole) auf einem Server betreibt, sollte man für jedes einen Ordner anlegen und darin eine Docker-Compose-Datei ablegen. Die nützlichen Helfer wie Portainer und Watchtower kommen zusammen in eine weitere Datei. Dann kann man mit `docker compose down`gezielt Teile der Umgebung herunterfahren.

---

## Service-Übersicht

* [Nginx](#nginx)
* [Portainer](#portainer)
* [Pi-hole](#pi-hole)
* [Watchtower](#watchtower)

---

## Nginx

### Aufgabe und Zweck
[Nginx](https://nginx.org/) ist ein leistungsstarker Webserver und Reverse Proxy. In diesem Projekt dient er zum Bereitstellen von statischem Web-Content und zum Testen des Port-Mappings in Docker Container-Netzwerken.

### Verwendetes Docker-Image
* `nginx:latest`

### Ports
* Extern auf dem Host: `8080`
* Intern im Container: `80` *(Weboberfläche erreichbar unter http://localhost:8080)*

### Befehle
* **Starten:** `docker compose -f nginx/nginx.yml up -d`
* **Beenden:** `docker compose -f nginx/nginx.yml down`

### Beobachtungen beim Test
* Nach dem Anpassen des Host-Ports von `80` auf `8080` konnte die Willkommensseite von Nginx erfolgreich im Browser über `http://localhost:8080` geladen werden.
* Das anfängliche Volume-Mapping auf `/etc/nginx` verhinderte das Laden der Standard-Konfiguration. Nach der Korrektur lief der Container stabil im Hintergrund (`Up`).

---

## Portainer

### Aufgabe und Zweck
[Portainer](https://www.portainer.io/) ist eine grafische Verwaltungsoberfläche (GUI) für Docker-Umgebungen. Es erleichtert das Verwalten von Containern, Images, Netzwerken und Volumes, ohne dass komplexe Terminal-Befehle erforderlich sind.

### Verwendetes Docker-Image
* `portainer/portainer-ce:latest`

### Ports
* Extern auf dem Host: `9000` (HTTP) / `9443` (HTTPS)
* Intern im Container: `9000` / `9443` *(Weboberfläche erreichbar unter https://localhost:9443)*

### Befehle
* **Starten:** `docker compose -f portainer/docker-compose.yml up -d`
* **Beenden:** `docker compose -f portainer/docker-compose.yml down`

### Beobachtungen beim Test
* Beim ersten Aufruf der Web-Oberfläche muss direkt ein Admin-Passwort vergeben werden.
* Das Mounting des Docker-Sockets (`/var/run/docker.sock`) ermöglicht Portainer die vollständige Echtzeit-Inspektion aller auf dem Mac laufenden Container.

---

## Pi-hole

### Aufgabe und Zweck
[Pi-hole](https://pi-hole.net/) agiert als lokaler DNS-Server mit integriertem Ad-Blocker. Es filtert unerwünschten Netzwerkverkehr und Werbung auf DNS-Ebene für alle verbundenen Geräte im Netzwerk heraus.

### Verwendetes Docker-Image
* `pihole/pihole:latest`

### Ports
* `53:53/tcp` und `53:53/udp` (DNS-Dienst)
* `80:80/tcp` oder `8081:80/tcp` *(Weboberfläche erreichbar unter http://localhost:8081/admin)*

### Befehle
* **Starten:** `docker compose -f pihole/docker-compose.yml up -d`
* **Beenden:** `docker compose -f pihole/docker-compose.yml down`

### Beobachtungen beim Test
* Beim Start legt Pi-hole automatische Konfigurationsdateien in den lokalen Ordnern (`etc-pihole` und `etc-dnsmasq.d`) an.
* Da diese Ordner permanente und temporäre Daten enthalten, wurden sie in der `.gitignore` eingetragen, um ein Verunreinigen des Git-Repositories zu verhindern.

---

## Watchtower

### Aufgabe und Zweck
[Watchtower](https://containrrr.dev/watchtower/) ist ein Automatisierungs-Tool für Docker. Es überwacht im Hintergrund laufende Container und aktualisiert diese automatisch auf die neueste Image-Version, sobald eine neue Version auf Docker Hub verfügbar ist.

### Verwendetes Docker-Image
* `containrrr/watchtower:latest`

### Ports
* **Keine Weboberfläche vorhanden.** Watchtower läuft als reiner Hintergrund-Daemon ohne exponierte HTTP-Ports.

### Befehle
* **Starten:** `docker compose -f watchtower/docker-compose.yml up -d`
* **Beenden:** `docker compose -f watchtower/docker-compose.yml down`


