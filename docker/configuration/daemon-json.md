# Docker Konfigurationsdateien & daemon.json Referenz

## Konfigurationsdateien

### Docker Daemon
| Datei | Zweck |
|-------|-------|
| `/etc/docker/daemon.json` | Haupt-Daemon-Config |
| `/lib/systemd/system/docker.service` | systemd Unit (Startparameter) |
| `/etc/systemd/system/docker.service.d/*.conf` | systemd Overrides |

### CLI / Client
| Datei | Zweck |
|-------|-------|
| `~/.docker/config.json` | Registry-Auth, Proxies, Default-Context |

### Compose
| Datei | Zweck |
|-------|-------|
| `docker-compose.yml` / `compose.yaml` | Stack-Definition |
| `.env` | Variablen für Compose |

### TLS / Registry-Zertifikate
```bash
/etc/docker/certs.d/
└── registry.example.com:5000/
    └── ca.crt    # CA-Zertifikat (selbstsigniert oder eigene CA)
```

```bash
mkdir -p /etc/docker/certs.d/registry.example.com:5000
cp ca.crt /etc/docker/certs.d/registry.example.com:5000/ca.crt
```

> Kein Docker-Neustart nötig – wird beim nächsten Pull/Push automatisch gelesen.

---

## daemon.json – Konfigurationsoptionen

### Logging
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```
> ⚠️ **Kaputt:** `"log-driver": "inexistent"` → Docker startet nicht mehr. `dockerd` versucht beim Start den Log-Driver zu initialisieren – ist er unbekannt, bricht der Daemon komplett ab (auch `docker ps` funktioniert nicht mehr).

**Gültige Log-Driver:** `json-file`, `syslog`, `journald`, `gelf`, `fluentd`, `awslogs`, `splunk`, `gcplogs`, `logentries`, `local`, `none`

---

### Registry Mirror
```json
{
  "registry-mirrors": ["https://mirror.example.com"]
}
```
> ⚠️ **Kaputt:** Falscher Mirror-URL → alle Pulls schlagen still fehl

---

### Insecure Registry
```json
{
  "insecure-registries": ["192.168.1.100:5000"]
}
```
**Typische Verwendung:** Lokale Entwicklung mit selbst gehosteter Registry ohne TLS – z.B. Harbor oder Plain-Docker-Registry im Heimnetz oder CI-Lab. Nur für private IPs sinnvoll (`192.168.x.x`, `10.x.x.x`, `localhost`).

> ⚠️ **Kaputt:** Produktions-Registry als insecure → gesamte Kommunikation läuft unverschlüsselt über HTTP, Images und Zugangsdaten im Klartext übertragbar

---

### Storage Driver
```json
{
  "storage-driver": "overlay2"
}
```
> ⚠️ **Kaputt:** Wechsel des Storage-Drivers → alle bestehenden Images/Container sind weg (anderes Verzeichnislayout)

---

### Data Root
```json
{
  "data-root": "/mnt/docker-data"
}
```
> ⚠️ **Kaputt:** Pfad existiert nicht oder falsches Filesystem (kein overlay2-Support) → Docker startet nicht

---

### Proxy
```json
{
  "proxies": {
    "http-proxy": "http://proxy:3128",
    "no-proxy": "localhost,127.0.0.1"
  }
}
```
> ⚠️ **Kaputt:** Proxy falsch oder nicht erreichbar → alle Pulls/Pushes hängen

---

### DNS
```json
{
  "dns": ["8.8.8.8", "1.1.1.1"]
}
```
> ⚠️ **Kaputt:** Falsche DNS-IP → Container-interne Namensauflösung bricht komplett

---

### Experimental Features
```json
{
  "experimental": true
}
```
> ⚠️ **Kaputt:** In Produktion ungetestete Features aktiv

---

### IP-Tables
```json
{
  "iptables": false
}
```
> ⚠️ **Kaputt:** Container-Networking funktioniert nicht mehr, kein Port-Forwarding

---

### JSON validieren (vor Neustart!)
```bash
dockerd --validate --config-file /etc/docker/daemon.json
# oder
cat /etc/docker/daemon.json | python3 -m json.tool
```
> ⚠️ **Generelle Falle:** Ungültiges JSON (fehlendes Komma, Tippfehler) → `dockerd` startet nicht

---

## overlay2 – Filesystem-Voraussetzungen

### Unterstützte Filesysteme
| Filesystem | overlay2 | Anmerkung |
|------------|----------|-----------|
| **ext4** | ✅ | Empfohlen, out-of-the-box |
| **xfs** | ✅ | Nur mit `ftype=1` (d_type) |
| **btrfs** | ❌ | Docker blockt overlay2 aktiv – eigener `btrfs`-Driver nötig |
| **tmpfs** | ❌ | Kein d_type-Support |
| **NFS** | ❌ | Kein overlay-Support |
| **FUSE-Varianten** | ❌ | Meist nicht unterstützt |

### xfs: ftype=1 prüfen
```bash
xfs_info /dev/sda1 | grep ftype
# ftype=1 → OK
# ftype=0 → overlay2 nicht möglich
```

> `ftype=1` muss beim **Formatieren** gesetzt werden – nachträglich nicht änderbar:
```bash
mkfs.xfs -n ftype=1 /dev/sda1
```

### Mount-Optionen

**ext4** – keine speziellen Mount-Optionen nötig:
```
/dev/sda1 /var/lib/docker ext4 defaults 0 0
```

**xfs** – ebenfalls `defaults` ausreichend, solange `ftype=1` beim Formatieren gesetzt wurde:
```
/dev/sda1 /var/lib/docker xfs defaults 0 0
```

> `defaults` reicht in beiden Fällen – overlay2 ist keine Mount-Option, sondern eine Kernel-Funktion die Docker intern nutzt.

### btrfs nativer Driver
```json
{
  "storage-driver": "btrfs"
}
```

### Kernel & Storage Driver prüfen
```bash
# overlay2 im Kernel vorhanden?
grep overlay /proc/filesystems

# Docker-Status
docker info | grep -E "Storage Driver|Backing Filesystem"
```
