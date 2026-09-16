# Docker Compose Netzwerke – Dokumentation

Diese Datei beschreibt die Grundlagen der Netzwerkkonfiguration in Docker Compose basierend auf der offiziellen Docker-Dokumentation.

---

## 1. Standardnetzwerk von Docker Compose (Default Network)

Wenn in einer `docker-compose.yml` kein separates Netzwerk definiert wird, erstellt Docker Compose automatisch ein Standardnetzwerk (Bridge-Netzwerk). 
* **Namensgebung:** Der Name setzt sich aus dem Projektnamen (Ordnernamen) und dem Suffix `_default` zusammen (z. B. `nginx_default`).
* **Funktionsweise:** Alle in der Datei definierten Container treten diesem gemeinsamen Netzwerk automatisch bei. Sie können direkt miteinander kommunizieren, sind jedoch nach außen hin isoliert, solange keine Ports freigegeben werden.

---

## 2. Kommunikation über Servicenamen (DNS-Auflösung)

Innerhalb desselben Docker Compose Netzwerks agiert der **Service-Name** (der Schlüssel unter `services:` in der YAML-Datei) als DNS-Hostname.
* **Beispiel:** Ein Service namens `web` kann einen Datenbank-Service namens `db` einfach über das Kürzel `http://db:5432` erreichen.
* **Vorteil:** Es müssen keine dynamischen IP-Adressen hart in Anwendungscodes oder Konfigurationsdateien hinterlegt werden.

---

## 3. Benutzerdefinierte Netzwerke (Custom Networks)

Unter dem Top-Level-Schlüssel `networks:` können eigene Netzwerke definiert werden, um Container gezielt zu isolieren oder zu gruppieren.
* **Sicherheits-Vorteil (Netzwerktrennung):** Container kommunizieren nur dann miteinander, wenn sie im *selben* Netzwerk Mitglied sind. 
* **Mehrfachmitgliedschaft:** Ein Container kann mehreren Netzwerken gleichzeitig angehören (z. B. ein Backend-Service im `frontend-net` und im `db-net`, während die Datenbank nur im `db-net` erreichbar ist).

---

## 4. Besondere Netzwerkmodi (`host` und `none`)

Neben dem Standard-Bridge-Modus unterstützen Container spezielle Netzwerkmodi:
* **`network_mode: host`:** Der Container teilt sich den Netzwerk-Stack direkt mit dem Host-System (Mac/Linux). Es gibt keine Netzwerktrennung mehr; der Container nutzt direkt die IP-Adresse und Ports des Host-Rechners. *(Port-Mappings via `ports:` sind hier wirkungslos).*
* **`network_mode: none`:** Schaltet jegliche Netzwerkfunktionalität für den Container komplett ab. Der Container hat nur das Loopback-Interface (`127.0.0.1`) und ist weder nach außen noch für andere Container erreichbar.

---

## 5. Unterschied zwischen `docker compose stop` und `docker compose down`

* **`docker compose stop`:** Stoppt lediglich die laufenden Container. Die Container-Instanzen, erstellten Netzwerke und Volumes bleiben erhalten.
* **`docker compose down`:** Stoppt die Container **und entfernt** die Container-Instanzen sowie alle von Compose automatisch erstellten Netzwerke vollständig. *(Daten in benannten Volumes bleiben erhalten).*

---

## 6. Kommunikation zwischen verschiedenen Compose-Projekten

Standardmäßig sind zwei getrennte Compose-Projekte strikt voneinander isoliert. Soll Service A aus Projekt 1 mit Service B aus Projekt 2 sprechen, nutzt man ein **externes Netzwerk** (`external: true`):

1. **Netzwerk manuell erstellen:**
   ```bash
   docker network create mein-geteiltes-netz
