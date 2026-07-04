# Docker-Lab

Ein Beispiel-Setup für ein eigenes Docker-Lab im lokalen Netzwerk mit eigener CA (Certificate Authority).
Alle Dienste laufen hinter [Traefik](https://traefik.io/) als Reverse-Proxy und sind per HTTPS über interne Domainnamen erreichbar.

## Voraussetzungen

* Linux-Server mit installiertem [Docker](https://docs.docker.com/engine/install/) und Docker Compose
* Eigene CA bzw. selbst-signiertes Wildcard-Zertifikat (z. B. mit `mkcert`)
* Interner DNS-Server oder Einträge in `/etc/hosts`

## Enthaltene Dienste

| Dienst | Beschreibung | URL | Interner Port |
|---|---|---|---|
| [Traefik](https://traefik.io/) | Reverse-Proxy mit Dashboard | `https://traefik.intern.local` | 8080 |
| [Dockge](https://github.com/louislam/dockge) | Docker-Compose-Stack-Manager | `https://dockge.intern.local` | 5001 |
| [Portainer](https://www.portainer.io/) | Docker-Verwaltung per Web-UI | `https://portainer.intern.local` | 8000 |
| [Nginx](https://nginx.org/) | Test-Webseite (HTML5UP-Template) | `https://app.intern.local` | 80 |
| [IT-Tools](https://github.com/CorentinTh/it-tools) | Sammlung nützlicher IT-Werkzeuge | `https://tools.intern.local` | 80 |
| [Stirling-PDF](https://github.com/Stirling-Tools/Stirling-PDF) | PDF-Werkzeuge im Browser | `https://pdf.intern.local` | 8080 |
| [Vaultwarden](https://github.com/dani-garcia/vaultwarden) | Passwort-Manager (Bitwarden-kompatibel) | `https://vault.intern.lan` | 80 |
| [NetBox](https://github.com/netbox-community/netbox-docker) | IPAM/DCIM (Verzeichnis als Platzhalter) | – | – |

## Verzeichnisstruktur

```
docker-lab/
├── dockge/            # Dockge Stack-Manager
├── it-tools/          # IT-Tools
├── netbox-docker/     # Platzhalter für netbox-docker
├── nginx/             # Nginx mit Test-Webseite (html5up-stellar)
├── portainer/         # Portainer CE
├── stirling-pdf/      # Stirling-PDF
├── traefik/
│   ├── certs/         # Zertifikat + Private Key (cert.crt / private.key)
│   ├── config/
│   │   ├── traefik.yml    # Statische Traefik-Konfiguration
│   │   └── dynamic.yml    # TLS-Zertifikate & TLS-Optionen
│   └── docker-compose.yml
└── vault/             # Vaultwarden
```

## Installation

### 1. Repository klonen

Eigenes Docker-Verzeichnis anlegen und das Repository klonen:

```bash
mkdir /opt/docker && cd $_

git clone https://github.com/mr-jo-reese/docker-lab.git
```

> **Hinweis:** Dockge erwartet die Stacks unter `/opt/docker` (siehe Volume-Mapping in `dockge/docker-compose.yml`). Wird ein anderer Pfad verwendet, muss das Mapping angepasst werden.

### 2. Docker-Netzwerk anlegen

Alle Stacks nutzen das externe Docker-Netzwerk `proxy`. Dieses muss einmalig erstellt werden:

```bash
docker network create proxy
```

### 3. Eigene CA und Zertifikat erstellen

Für die eigene CA kann z. B. [mkcert](https://github.com/FiloSottile/mkcert) oder [openssl](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-ca-on-ubuntu-20-04) verwendet werden.

```bash
# mkcert unter Ubuntu installieren
apt install mkcert

# Eigene CA erstellen und installieren
mkcert --install

# Wildcard-Zertifikat für Traefik erstellen
mkcert "*.intern.local"
```

Die erzeugten Dateien liegen im aktuellen Verzeichnis. Anschließend Zertifikat und Private Key in das Traefik-Verzeichnis kopieren — die Dateinamen müssen zu `traefik/config/dynamic.yml` passen:

```bash
cp _wildcard.intern.local.pem       /opt/docker/docker-lab/traefik/certs/cert.crt
cp _wildcard.intern.local-key.pem   /opt/docker/docker-lab/traefik/certs/private.key
```

Damit der Browser den Zertifikaten vertraut, muss die Root-CA (`mkcert -CAROOT`) auf den Client-Geräten importiert werden.

### 4. DNS-Einträge anlegen

Damit die Dienste im Browser erreichbar sind, die Domainnamen im internen DNS-Server eintragen oder in die `/etc/hosts` der Clients aufnehmen. Die IP-Adresse ist die des Docker-Hosts (im Beispiel `192.168.1.99`):

```
192.168.1.99    traefik.intern.local
192.168.1.99    dockge.intern.local
192.168.1.99    portainer.intern.local
192.168.1.99    app.intern.local
192.168.1.99    tools.intern.local
192.168.1.99    pdf.intern.local
192.168.1.99    vault.intern.lan
```

### 5. Container starten

Zuerst muss Traefik gestartet werden:

```bash
cd /opt/docker/docker-lab/traefik
docker compose up -d
```

Danach können die übrigen Dienste gestartet werden, z. B.:

```bash
# IT-Tools
cd /opt/docker/docker-lab/it-tools
docker compose up -d

# Portainer
cd /opt/docker/docker-lab/portainer
docker compose up -d
```

## Funktionsweise

* **Traefik** lauscht auf Port 80 und 443. HTTP-Anfragen werden automatisch auf HTTPS umgeleitet (siehe `traefik/config/traefik.yml`).
* Die Container werden über **Docker-Labels** bei Traefik registriert (`exposedByDefault: false` — nur Container mit `traefik.enable: true` werden veröffentlicht).
* Das Wildcard-Zertifikat und die TLS-Optionen (min. TLS 1.2, sichere Cipher-Suites, SNI strict) sind in `traefik/config/dynamic.yml` konfiguriert.
* Alle Dienste kommunizieren über das gemeinsame externe Netzwerk `proxy`; nach außen sind nur die Traefik-Ports 80/443 geöffnet.

## Hinweise

* **Vaultwarden** verwendet abweichend die Domain `vault.intern.lan` (siehe `vault/docker-compose.yml`). Dafür wird ein passendes Zertifikat bzw. ein zusätzlicher DNS-Eintrag benötigt. Nach der Einrichtung der eigenen Konten sollte `SIGNUPS_ALLOWED=false` gesetzt werden.
* **NetBox**: Das Verzeichnis `netbox-docker/` ist aktuell ein Platzhalter — die Konfiguration folgt dem Projekt [netbox-docker](https://github.com/netbox-community/netbox-docker).
* Das Traefik-Log-Level steht auf `DEBUG` und kann für den Produktivbetrieb in `traefik/config/traefik.yml` auf `INFO` reduziert werden.
