# Server Installation

## Inhaltsverzeichnis
- [Server Installation](#server-installation)
  - [Inhaltsverzeichnis](#inhaltsverzeichnis)
  - [Hardware installieren und vorbereiten](#hardware-installieren-und-vorbereiten)
    - [Ziel](#ziel)
    - [Zutaten](#zutaten)
    - [Arbeitsschritte](#arbeitsschritte)
  - [Betriebssystem installieren und konfigurieren](#betriebssystem-installieren-und-konfigurieren)
    - [Ziel](#ziel-1)
    - [Zutaten](#zutaten-1)
    - [Arbeitsschritte](#arbeitsschritte-1)
      - [1. Bootfähigen USB-Stick erstellen](#1-bootfähigen-usb-stick-erstellen)
      - [2. Server booten](#2-server-booten)
      - [3. Betriebssystem installieren](#3-betriebssystem-installieren)
  - [Remote SSH-Zugriff einrichten](#remote-ssh-zugriff-einrichten)
    - [Ziel](#ziel-2)
    - [Arbeitsschritte](#arbeitsschritte-2)
  - [Deployment](#deployment)
    - [Ziel](#ziel-3)
    - [Zutaten](#zutaten-2)
    - [Arbeitsschritte](#arbeitsschritte-3)
  - [Nginx-Konfiguration](#nginx-konfiguration)
    - [Statische Webseiten](#statische-webseiten)
    - [Dynamische Webseiten](#dynamische-webseiten)
  - [Domainnamen (DNS-Verbindung)](#domainnamen-dns-verbindung)
  - [SSL Zertifikate (HTTPS einrichten)](#ssl-zertifikate-https-einrichten)
  - [Service-Konfiguration (Gunicorn)](#service-konfiguration-gunicorn)
  - [Datenbank-Backups](#datenbank-backups)
    - [Ziel](#ziel-4)
    - [Arbeitsschritte](#arbeitsschritte-4)
  - [Mail-Server (Sendmail)](#mail-server-sendmail)
    - [Ziel](#ziel-5)
  - [Automatisierte Git-Pull-Action](#automatisierte-git-pull-action)
    - [Ziel](#ziel-6)
    - [Arbeitsschritte](#arbeitsschritte-5)
  - [Debugging](#debugging)
  - [Zusammenfassung](#zusammenfassung)
  - [Theorie](#theorie)
  - [Links](#links)
  - [Fragen](#fragen)
  - [To Do](#to-do)

---

## Hardware installieren und vorbereiten

### Ziel
Den Server für die Installation eines Betriebssystems vorbereiten.

### Zutaten
- Monitor (an Strom angeschlossen und mit dem Server verbunden)
- Server (Modell: ThinkStation D20, am Strom und Netzwerk angeschlossen)
- Zwei leere oder überschreibbare Festplatten/SSDs
- Mindestens 40 GB RAM
- Funktionierende Batterie

### Arbeitsschritte
1. Komponenten im Serverraum installieren:
   - SSDs einbauen und RAM erweitern.
   - Batterie prüfen und bei Bedarf ersetzen (Signalbereich prüfen: grün = gut).

2. Netzwerk-Interfaces identifizieren:
   - Führe den Befehl ip addr show aus, um die aktiven Ethernet-Anschlüsse zu finden.
   - Ein aktiver Anschluss wird als "UP" angezeigt. Merke dir den Namen des Interfaces! (den bracht man später für der Netzwerkkonfiguration)

---

## Betriebssystem installieren und konfigurieren

### Ziel
Ein Betriebssystem installieren und den Server bereit für Speicherung und Internetzugang machen.

### Zutaten
- USB-Stick
- Remote-PC oder Laptop mit Linux und Internetzugang
- Server
- BIOS-Zugang (hier: Taste F1) wissen

### Arbeitsschritte
#### 1. Bootfähigen USB-Stick erstellen
- Lade das gewünschte Betriebssystem (z. B. [Ubuntu Server](https://ubuntu.com/download/server)) herunter.
- Stecke den USB-Stick ein und identifiziere den USB-Stick mit sudo lsblk. Aus folgendem Output kann man sehen dass der Stick den Path /dev/sda hat mit einer Partition /dev/sda/sda1!
  ```bash
  sda           8:0    1 114.6G  0 disk 
  └─sda1        8:1    1 114.6G  0 part /media/zoe/E71C-66DE
  ```
- Kopiere das ISO-Image auf den Stick, wir überschrieben nun die KOMPLETTE Disk und nicht nur eine Partition! Die Partitionstabelle ist bereits auf dem ISO-Image:
  ```bash
  sudo dd if=/path/to/filename.iso of=/dev/sda bs=4M status=progress oflag=sync
  ```
#### 2. Server booten
- Schließe den bootfähigen USB-Stick an den ausgeschalteten Server an.
- Starte den Server und drücke sofort wiederholt die BIOS-Taste, um ins BIOS (hier UEFI) zu gelangen.
- Gehe ins StartUp-Menu. Hier werden alle angeschlossenen Geräte aufgelistet. Das oberste davon wir dausgewählt beim Bootvorgang um zu booten!
- Passe die Bootreihenfolge an (USB-Stick an erster Stelle). Die SSDs müssen ebenfalls in die Bootreihenfolge!
- Save and Exit. Nun bootet das System vom Memory Stick!

#### 3. Betriebssystem installieren
- Folge der [Ubuntu Server Installationsanleitung](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview).
- Beachte spezifische Einstellungen wie:
  - Sprache und Tastatur (z. B. Deutsch, Schweizer Layout).
  - Netzwerkeinstellungen: Hier geht man von einer statischen IP-Adresse aus!
  - Partitionierung enthält: SWAP-Partition, Boot-Partition, RAID1-Partition.
  
Mit sudo lsblk kannst du im nachinein dann die Partitionstabelle ansehen im Terminal.
```bash
sda       8:0    0   1,8T  0 disk  
├─sda1    8:1    0     1M  0 part  
├─sda2    8:2    0     2G  0 part  
│ └─md0   9:0    0     2G  0 raid1 /boot
├─sda3    8:3    0     2G  0 part  [SWAP]
└─sda4    8:4    0   1,8T  0 part  
  └─md1   9:1    0   1,8T  0 raid1 /
sdb       8:16   0   1,8T  0 disk  
├─sdb1    8:17   0     1M  0 part  
├─sdb2    8:18   0     2G  0 part  
│ └─md0   9:0    0     2G  0 raid1 /boot
├─sdb3    8:19   0     2G  0 part  [SWAP]
└─sdb4    8:20   0   1,8T  0 part  
  └─md1   9:1    0   1,8T  0 raid1 /
```
---

## Remote SSH-Zugriff einrichten

### Ziel
Einen sicheren Zugriff auf den Server von einem externen Gerät ermöglichen.

### Arbeitsschritte
1. Installiere openssh auf Server und Client.
2. Erstelle SSH-Keys:
   ```bash
   ssh-keygen -t rsa -b 4096
   ```
3. Finde die IP-Adresse des Servers mit ip addr show.
4. Kopiere den Public Key vom Client auf den Server:
   ```bash
   ssh-copy-id username@ip-address
   ```

---

## Deployment

### Ziel
Webseiten bereitstellen, die über einen Browser zugänglich sind.

### Zutaten
- nginx
- Projektdateien
- gunicorn (als Systemd-Service)

### Arbeitsschritte
1. Richte nginx für statische und dynamische Webseiten ein.
2. Konfiguriere Services für dynamische Applikationen.
3. Verbinde Domainnamen mit der Server-IP.
4. Stelle SSL-Zertifikate bereit.

---

## Nginx-Konfiguration

### Statische Webseiten
1. Erstelle eine Konfigurationsdatei:
   
   sudo nano /etc/nginx/sites-available/nginx_config
   
2. Füge die Einstellungen hinzu (z. B. listen 80, root-Verzeichnis).
3. Verlinke die Datei in den sites-enabled-Ordner:
   
   sudo ln -s /etc/nginx/sites-available/nginx_config /etc/nginx/sites-enabled/
   
4. Lade die Konfiguration neu:
   
   sudo systemctl restart nginx
   

### Dynamische Webseiten
- Führe ähnliche Schritte wie oben aus, aber passe die proxy_pass-Einstellungen für den WSGI-Server an.

---

## Domainnamen (DNS-Verbindung)
1. Kaufe eine Domain (z. B. bei GoDaddy).
2. Richte DNS-Records ein, um Domains mit dem Server zu verbinden.

---

## SSL Zertifikate (HTTPS einrichten)
1. Installiere Certbot:
   
   sudo apt install certbot
   
2. Hole ein Zertifikat:
   
   sudo certbot --nginx -d yourdomain.com
   
3. Konfiguriere automatisches Erneuern der Zertifikate.

---

## Service-Konfiguration (Gunicorn)
1. Klone das Repository und installiere die Abhängigkeiten.
2. Starte die App mit Gunicorn:
   
   gunicorn --workers 3 --bind 0.0.0.0:8000 app:app
   
3. Richte einen Systemd-Service für Gunicorn ein.

---

## Datenbank-Backups

### Ziel
Automatisierte Backups der Datenbank erstellen.

### Arbeitsschritte
- Installiere SQLite3, um die Datenbank manuell zu testen.
- Erstelle ein Backup-Skript und richte es als Service ein.

---

## Mail-Server (Sendmail)

### Ziel
Einen Mail-Server einrichten (z. B. mit Sendmail).

---

## Automatisierte Git-Pull-Action

### Ziel
Automatische Synchronisation bei Pushes auf den Deployment-Branch.

### Arbeitsschritte
1. Installiere Git und konfiguriere die SSH-Keys.
2. Erstelle ein GitHub-Workflow-Skript für automatische Pull-Aktionen.

---
## Debugging
- Überprüfe Firewalls und laufende Services.
- Stelle sicher, dass alle notwendigen Ports geöffnet sind (z. B. 80, 443 für HTTP/HTTPS, 22 für SSH).
- Prüfe die Logs der Services:
  
  sudo journalctl -u <service-name>
  
- Teste die Konnektivität zwischen Geräten mit ping und telnet.
- Stelle sicher, dass die Systemressourcen ausreichend sind (RAM, CPU, Speicherplatz).
- Überprüfe die Syntax und Integrität von Konfigurationsdateien, z. B. für nginx:
  
  sudo nginx -t
  
- Nutze Debugging-Werkzeuge wie strace, tcpdump oder netstat, um detaillierte Einblicke in Probleme zu erhalten.
- Überprüfe, ob die Dienste korrekt gestartet sind und im Hintergrund laufen:
  
  sudo systemctl status <service-name>
  

---

## Zusammenfassung
- Ergebnis: Ein funktionierender Server mit gehosteten Webseiten.
- Wichtige Daten:
  - Statische IP: 195.234.163.110
  - Domains: tamagotchi.werft22.net, flumini.ch

---

## Theorie
- Details zu RAID1, SSH, DNS und SSL/TLS.

---

## Links
- [Bootable USB Stick erstellen](https://pendrivelinux.com/create-bootable-usb-from-iso-using-dd/)
- [Let’s Encrypt Anleitung](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-22-04)

---

## Fragen
- Wie kann man Notifications für Serverausfälle einrichten?
- Warum erfordert GitHub keine Passwortabfrage?

---

## To Do
- Applikationen hochladen
- Backups und GitHub Actions testen
- Serverdokumentation abschließen
