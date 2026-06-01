# Overleaf Server Dokumentation
> Server: `164.30.68.85` | Domain: `https://overleaf-luc.duckdns.org`

---

## Übersicht

Wir haben die **Overleaf Community Edition** auf einem Ubuntu-Server installiert. Overleaf läuft in Docker-Containern und ist über Nginx mit HTTPS erreichbar.

### Komponenten
- **Overleaf (Sharelatex)** — die eigentliche LaTeX-Webanwendung
- **MongoDB** — Datenbank für Projekte und Nutzer
- **Redis** — Session-Verwaltung
- **Nginx** — Reverse Proxy, leitet HTTPS-Traffic an Overleaf weiter
- **Let's Encrypt** — kostenloses SSL-Zertifikat (automatische Erneuerung)

---

## Verzeichnisstruktur

```
/home/overleaf/
└── overleaf-toolkit/          # Hauptverzeichnis
    ├── bin/                   # Steuerungsskripte
    ├── config/
    │   └── overleaf.rc        # Hauptkonfiguration
    └── data/
        ├── overleaf/          # Overleaf-Daten
        ├── mongo/             # Datenbank
        └── redis/             # Redis-Daten
```

---

## Täglicher Betrieb

### SSH-Verbindung zum Server
```bash
ssh overleaf@164.30.68.85
```

### Overleaf starten
```bash
cd ~/overleaf-toolkit
./bin/up -d
```

### Overleaf stoppen
```bash
cd ~/overleaf-toolkit
./bin/stop
```

### Status prüfen
```bash
cd ~/overleaf-toolkit
./bin/docker-compose ps
```

Alle drei Container sollten `Up` zeigen:
```
mongo        Up (healthy)
redis        Up
sharelatex   Up
```

### Logs anschauen
```bash
# Alle Container
./bin/docker-compose logs

# Nur Overleaf
./bin/docker-compose logs sharelatex

# Live mitverfolgen
./bin/docker-compose logs -f sharelatex
```

---

## Overleaf aufrufen

Im Browser:
```
https://overleaf-luc.duckdns.org
```

### Admin-Bereich
```
https://overleaf-luc.duckdns.org/launchpad    # Ersteinrichtung
https://overleaf-luc.duckdns.org/admin        # Admin-Panel
```

---

## TeX Live Pakete nachinstallieren

Falls ein LaTeX-Paket fehlt (z.B. `File 'xyz.sty' not found`):

### In den Container wechseln
```bash
cd ~/overleaf-toolkit
./bin/shell
```

### Einzelnes Paket installieren
```bash
tlmgr option repository https://ftp.tu-chemnitz.de/pub/tug/historic/systems/texlive/2025/tlnet-final
tlmgr install paketname
```

### Alle Pakete installieren (scheme-full, ~4GB)
```bash
tlmgr option repository https://ftp.tu-chemnitz.de/pub/tug/historic/systems/texlive/2025/tlnet-final
tlmgr install scheme-full
```

### Container verlassen und neu starten
```bash
exit
cd ~/overleaf-toolkit
./bin/stop
./bin/up -d
```

---

## Konfiguration anpassen

```bash
nano ~/overleaf-toolkit/config/overleaf.rc
```

Wichtige Einstellungen:
```ini
OVERLEAF_SITE_URL=https://overleaf-luc.duckdns.org
OVERLEAF_LISTEN_IP=127.0.0.1
OVERLEAF_PORT=8081
```

Nach Änderungen neu starten:
```bash
cd ~/overleaf-toolkit
./bin/stop
./bin/up -d
```

---

## Overleaf aktualisieren

```bash
cd ~/overleaf-toolkit
./bin/stop
./bin/upgrade
./bin/up -d
```

---

## Nginx

### Status prüfen
```bash
sudo systemctl status nginx
```

### Konfiguration prüfen
```bash
sudo nginx -t
```

### Neu starten
```bash
sudo systemctl restart nginx
```

### Konfigurationsdatei
```bash
sudo nano /etc/nginx/sites-available/overleaf
```

---

## SSL-Zertifikat (Let's Encrypt)

Das Zertifikat wird automatisch erneuert. Manuell erneuern:
```bash
sudo certbot renew
```

Ablaufdatum prüfen:
```bash
sudo certbot certificates
```

---

## DuckDNS IP aktualisieren

Falls sich die Server-IP ändert, muss DuckDNS aktualisiert werden:
- [duckdns.org](https://www.duckdns.org) aufrufen und neue IP eintragen

Oder automatisch per Cronjob (empfohlen):
```bash
crontab -e
```
Folgende Zeile hinzufügen:
```
*/5 * * * * curl -s "https://www.duckdns.org/update?domains=overleaf-luc&token=DEIN_TOKEN&ip=" > /dev/null
```
Den Token findest du auf der DuckDNS-Webseite.

---

## Backup

Die gesamten Daten liegen in:
```bash
~/overleaf-toolkit/data/
```

Backup erstellen:
```bash
cd ~/overleaf-toolkit
./bin/stop
tar -czf overleaf-backup-$(date +%Y%m%d).tar.gz data/
./bin/up -d
```

---

## Firewall (UFW)

```bash
# Status
sudo ufw status

# Offene Ports
# 22   - SSH
# 443  - HTTPS (SSH-Fallback + Nginx)
# 80   - HTTP (wird zu HTTPS weitergeleitet)
```

---

## Troubleshooting

| Problem | Lösung |
|---|---|
| Seite nicht erreichbar | `./bin/docker-compose ps` prüfen, ggf. `./bin/up -d` |
| LaTeX-Paket fehlt | `./bin/shell` → `tlmgr install paketname` |
| Nginx-Fehler | `sudo nginx -t` → `sudo systemctl restart nginx` |
| Zertifikat abgelaufen | `sudo certbot renew` |
| Container startet nicht | `./bin/docker-compose logs sharelatex` |
