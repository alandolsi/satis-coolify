# Satis on Coolify

Private Composer Package Repository mit automatischen Builds und Token-Authentifizierung.

## 🚀 Quick Start

1. **Coolify**: Erstelle neue "Docker Compose" Anwendung
2. **Repository**: Verknüpfe dieses Git-Repository
3. **Domain**: `packages.your-domain.com`
4. **Environment Variables** (siehe unten)
5. **Deploy** → Warte 3-5 Minuten für ersten Build

## 🔑 Required Environment Variables

```bash
# GitHub Token (ERFORDERLICH für private Repos!)
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Access Token für Repository (generiere mit: openssl rand -hex 32)
ACCESS_TOKEN=your-secure-random-token-here

# Build Interval in Sekunden (optional, default: 900 = 15 Minuten)
SATIS_BUILD_INTERVAL=900

# Satis Konfiguration
SATIS_NAME=your-vendor/packages
SATIS_HOMEPAGE=https://packages.your-domain.com
SATIS_DESCRIPTION=Private Composer Package Repository

# Repositories (JSON Array Format - WICHTIG!)
# Füge deine GitHub Repositories hinzu (kompaktes JSON ohne Zeilenumbrüche)
SATIS_REPOSITORIES=[{"type":"vcs","url":"https://github.com/YOUR-USERNAME/package-one.git"},{"type":"vcs","url":"https://github.com/YOUR-USERNAME/package-two.git"}]
```

> **💡 Tipp:** Coolify erlaubt Zeilenumbrüche in Environment Variables mit "Is Multiline". Das Script entfernt automatisch alle Whitespace, so dass auch formatiertes JSON funktioniert!

### GitHub Token erstellen:
1. https://github.com/settings/tokens/new?scopes=repo
2. Scope: `repo` (Full control of private repositories)
3. Token generieren und in Coolify einfügen

## 📦 Verwendung in Projekten

### Empfohlen: Bearer Token
```json
{
  "config": {
    "bearer": {
      "packages.your-domain.com": "DEIN_ACCESS_TOKEN"
    }
  },
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.your-domain.com"
    }
  ]
}
```

Dann Packages installieren:
```bash
composer require vendor/package-name
```

### Alternative: Query Parameter
```json
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.your-domain.com?token=DEIN_ACCESS_TOKEN"
    }
  ]
}
```

## ➕ Neues Package hinzufügen

1. **Editiere `SATIS_REPOSITORIES` in Coolify Environment Variables**:
   
   **Option A - Kompakt (eine Zeile):**
   ```json
   [{"type":"vcs","url":"https://github.com/USERNAME/package-one.git"},{"type":"vcs","url":"https://github.com/USERNAME/package-two.git"},{"type":"vcs","url":"https://github.com/USERNAME/NEUES-PACKAGE.git"}]
   ```
   
   **Option B - Formatiert (mit "Is Multiline" in Coolify):**
   ```json
   [
     {"type":"vcs","url":"https://github.com/USERNAME/package-one.git"},
     {"type":"vcs","url":"https://github.com/USERNAME/package-two.git"},
     {"type":"vcs","url":"https://github.com/USERNAME/NEUES-PACKAGE.git"}
   ]
   ```

2. **Speichern und Redeploy** in Coolify

3. **Warte 2-3 Minuten** → Package ist verfügbar!

4. **Installation**: `composer require vendor/package-name`

## 🔄 Package-Versionen aktualisieren

Satis scannt alle **15 Minuten** automatisch nach neuen Versionen/Tags.

**Wenn neue Version nicht erscheint:**
1. Prüfe ob Git-Tag existiert: `git tag -l` im Repository
2. Prüfe ob Tag gepusht wurde: `git push origin --tags`
3. Warte max. 15 Minuten (Build-Intervall)
4. Oder manuell Build triggern (in Coolify Container Console):
   ```bash
   /satis/bin/satis build -vvv /config/satis.json /build/output
   ```

## 📋 Files

- `docker-compose.yml` – Services: Builder + Nginx
- `.env.example` – Environment Variable Template
- `DEPLOYMENT_GUIDE.md` – Detaillierte Deployment-Anleitung

## ✨ Features

- ✅ Automatische Builds alle 15 Minuten (konfigurierbar)
- ✅ Token-basierte Authentifizierung (Bearer Token)
- ✅ Persistent Volumes für schnellere Builds
- ✅ GitHub API Rate Limit Handling
- ✅ SSL via Coolify
- ✅ Composer Cache
- ✅ Atomic Builds ohne Downtime
- ✅ Konfiguration via Environment Variables
- ✅ Automatische Whitespace-Bereinigung für JSON

## 📚 Dokumentation

Siehe [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) für:
- Detaillierte Setup-Anleitung
- Troubleshooting
- Monitoring
- Updates

## ⚠️ Wichtig

- **Erster Build dauert 3-5 Minuten** (Satis Installation)
- **Ohne GITHUB_TOKEN**: Rate Limits & langsamer
- **ACCESS_TOKEN**: Ohne ist Repository öffentlich!

## 🐛 Troubleshooting

**In Coolify Container Console (satis-builder):**

```bash
# Prüfe Environment Variables
env | grep SATIS

# Prüfe generierte satis.json
cat /config/satis.json

# Manueller Build mit Debug-Output
/satis/bin/satis build -vvv /config/satis.json /build/output

# Build-Output prüfen
ls -lah /build/output/
```

**Häufige Probleme:**

- **404 Fehler:** SATIS_REPOSITORIES enthält noch Platzhalter (YOUR-USERNAME)
- **Malformed URL:** Zeilenumbrüche in SATIS_REPOSITORIES (wird automatisch bereinigt seit v1.0.0)
- **Rate Limits:** GITHUB_TOKEN fehlt oder ist ungültig
- **Packages nicht sichtbar:** Warte 15 Minuten oder trigger manuellen Build

## 📞 Support

Bei Problemen siehe [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) Troubleshooting-Sektion.
