# Tamagotchi-Server-Scripts

Dieses Repository enthält alle Skripte, die für die Verwaltung und Einrichtung von Server-Services auf verschiedenen Maschinen benötigt werden. Es dient dazu, die Skripte remote verfügbar zu haben, sodass Services problemlos auf verschiedenen Servern eingerichtet und verwaltet werden können.

## Zweck

- Bereitstellung von Server-Skripten, die für verschiedene Konfigurationen und Automatisierungen erforderlich sind.
- Ermöglicht die Installation und Verwaltung von Services auf verschiedenen Servern.
- Zentralisierte Verwaltung der Server-Skripte für einfache Wartung und Aktualisierung.

## Verwendung

Service-Files --> /etc/systemd/system/name_der_app.service
\n
Nginx-Config-Files --> /etc/nginx/name_der_app
\n
Github Actions Script --> /projektordner/.github/workflows
\n
Backup-Script --> /projektordner/backup_sctipt.py
\n
Automatic-Push-Script (arbeitet mit Github Actions zusammen) --> ./webhook/webhook.sh
