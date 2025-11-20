# Satis Deployment Guide für Coolify

## ✅ Setup Status

Das Satis-Setup ist funktional und bereit für Deployment in Coolify. Die Tests haben gezeigt, dass das System korrekt arbeitet.

## 🔧 Verbesserungen im Setup

### 1. Persistent Volumes hinzugefügt
- **`satis`**: Speichert die Satis-Installation (vermeidet Neuinstallation)
- **`composer_cache`**: Cached Composer-Downloads für schnellere Builds

### 2. Optimiertes Logging
- Bessere Statusmeldungen mit ✓, →, ⚠ Symbolen
- Klarere Fehler- und Erfolgsmeldungen

### 3. Performance-Verbesserungen
- `--prefer-dist` Flag für schnellere Installation
- Persistente Volumes für Satis und Composer Cache
- `COMPOSER_HOME` Environment Variable explizit gesetzt

## 📋 Deployment-Schritte in Coolify

### 1. Repository in Coolify hinzufügen
1. In Coolify: **New Resource** → **Docker Compose**
2. Repository URL: `https://github.com/YOUR_USERNAME/satis-coolify.git`
3. Branch: `main`

### 2. **WICHTIG**: Environment Variables setzen

#### Erforderlich für private Repositories:
```bash
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
**Wie erstellen:**
1. Gehe zu: https://github.com/settings/tokens/new?scopes=repo
2. Scope: `repo` (Full control of private repositories)
3. Token generieren und in Coolify einfügen

#### Erforderlich für Token-Auth:
```bash
ACCESS_TOKEN=$(openssl rand -hex 32)
```
**Oder generiere manuell einen sicheren Token**

#### Optional:
```bash
SATIS_BUILD_INTERVAL=900  # Sekunden (default: 900 = 15 Minuten)
```

### 3. Domain konfigurieren
- In Coolify: Domain auf `packages.rosenheim-dev.de` setzen
- SSL-Zertifikat wird automatisch von Coolify verwaltet

### 4. Deploy & Warten
- Klicke auf **Deploy**
- **Erster Build dauert 3-5 Minuten** (Satis-Installation)
- Nachfolgende Builds sind schneller dank Caching

## 🧪 Lokales Testen (ohne echten GitHub Token)

Das System funktioniert auch ohne gültigen GitHub-Token, nutzt aber dann:
- Git-Source-Downloads (langsamer)
- Rate-Limits von GitHub (60 Requests/Stunde)

Für lokale Tests:
```bash
cp .env.example .env
# Editiere .env mit Test-Werten
docker compose up
```

## 📦 Verwendung in Composer-Projekten

Nach erfolgreichem Deployment:

### Methode 1: Query Parameter
```json
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.rosenheim-dev.de?token=DEIN_ACCESS_TOKEN"
    }
  ]
}
```

### Methode 2: Bearer Token (empfohlen)
```json
{
  "config": {
    "bearer": {
      "packages.rosenheim-dev.de": "DEIN_ACCESS_TOKEN"
    }
  },
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.rosenheim-dev.de"
    }
  ]
}
```

## 🐛 Troubleshooting

### Problem: "Could not authenticate against github.com"
**Ursache:** `GITHUB_TOKEN` fehlt oder ist ungültig
**Lösung:** 
1. Überprüfe Token in Coolify Environment Variables
2. Erstelle neuen Token mit `repo` Scope
3. Redeploy

### Problem: "Build failed" im Builder-Container
**Ursache:** Private Repositories nicht erreichbar
**Lösung:**
1. Prüfe ob Token gültig ist
2. Prüfe ob Token Zugriff auf die Repositories hat
3. Logs prüfen: `docker compose logs satis-builder`

### Problem: "Unauthorized" beim Zugriff
**Ursache:** `ACCESS_TOKEN` fehlt oder ist falsch
**Lösung:**
1. Token in Coolify Environment Variables prüfen
2. Token in `composer.json` muss identisch sein

### Problem: "404 Not Found" für Packages
**Ursache:** Erster Build noch nicht abgeschlossen
**Lösung:** 
1. Warte 3-5 Minuten nach Deployment
2. Prüfe Logs: `docker compose logs satis-builder`
3. Warte auf "Build completed successfully"

## 📊 Monitoring

### Logs anschauen
```bash
# Alle Logs
docker compose logs -f

# Nur Builder
docker compose logs -f satis-builder

# Nur Web Server
docker compose logs -f satis-web
```

### Status prüfen
```bash
docker compose ps
```

### Build-Output prüfen
```bash
docker compose exec satis-builder ls -lah /build/output/
```

## 🔄 Updates

### Repositories in satis.json aktualisieren
1. Editiere `docker-compose.yml` (Zeile 38-59)
2. Füge neue Repositories hinzu
3. Redeploy in Coolify

### Satis aktualisieren
```bash
# Container neu bauen (löscht Satis-Installation)
docker compose down -v
docker compose up
```

## ✨ Features

- ✅ Automatische Builds alle 15 Minuten (konfigurierbar)
- ✅ Token-basierte Authentifizierung
- ✅ Persistent Volumes für Performance
- ✅ Graceful Handling von Rate Limits
- ✅ Health Checks via Nginx
- ✅ Automatic SSL via Coolify
- ✅ Composer Cache für schnellere Builds

## 🔐 Sicherheit

- Token-Auth schützt vor unautorisierten Zugriffen
- HTTPS via Coolify SSL
- Nginx Security Headers aktiviert
- Private Repositories nur mit gültigem GitHub Token

## 📝 Hinweise

- **Erstes Deployment dauert länger**: Satis muss installiert werden
- **Nachfolgende Starts sind schnell**: Dank Volumes
- **Rate Limits**: GitHub API hat Limits - Token verwenden!
- **Caching**: Composer cached Dependencies für schnellere Builds
