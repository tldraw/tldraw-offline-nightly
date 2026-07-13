# tldraw offline nightly

The nightly channel for [tldraw offline](https://github.com/tldraw/tldraw-offline) — a local whiteboard for you and your agents. Nightly builds are packaged from the latest source before changes ship to stable users.

Nightly is intended for **internal/team use**. If you're an end user, get the stable build from [offline.tldraw.com](https://offline.tldraw.com) or the [tldraw-offline releases](https://github.com/tldraw/tldraw-offline/releases/latest) instead.

## Download

Artifact names are fixed across releases, so these links always point at the latest nightly build:

| Platform | Download |
| --- | --- |
| macOS (Apple silicon + Intel) | [tldraw-nightly-mac-universal.dmg](https://github.com/tldraw/tldraw-offline-nightly/releases/latest/download/tldraw-nightly-mac-universal.dmg) |
| Windows x64 | [tldraw-nightly-win-x64.exe](https://github.com/tldraw/tldraw-offline-nightly/releases/latest/download/tldraw-nightly-win-x64.exe) |
| Windows Arm64 | [tldraw-nightly-win-arm64.exe](https://github.com/tldraw/tldraw-offline-nightly/releases/latest/download/tldraw-nightly-win-arm64.exe) |
| Linux x64 | [tldraw-nightly-linux-x86_64.AppImage](https://github.com/tldraw/tldraw-offline-nightly/releases/latest/download/tldraw-nightly-linux-x86_64.AppImage) or [tldraw-nightly-linux-amd64.deb](https://github.com/tldraw/tldraw-offline-nightly/releases/latest/download/tldraw-nightly-linux-amd64.deb) |
| Linux Arm64 | [tldraw-nightly-linux-arm64.AppImage](https://github.com/tldraw/tldraw-offline-nightly/releases/latest/download/tldraw-nightly-linux-arm64.AppImage) |

Older builds are on the [Releases page](https://github.com/tldraw/tldraw-offline-nightly/releases). Versions are semver prereleases of the next release train, like `1.9.0-202607131439.9834194` (UTC timestamp + commit).

Nightly installs **side-by-side** with stable — different bundle id (`com.tldraw.desktop-nightly`), different name, different icon, and its own app data directory. Installing nightly does not touch your stable install or its files.

## Releases and auto-updates

A new nightly is published every morning (07:00 UTC) whenever something has changed since the last one, plus on-demand runs. Installed apps check for updates on launch, download them in the background, and then prompt to install now or on next startup. You can also check manually from the application menu.

## Local Canvas API

Like stable, the nightly app runs a local HTTP server exposing a Canvas API for programmatic access to your open tldraw documents — this is how coding agents inspect and edit a live canvas.

The server starts with the app (port 7236, falling back to a random port if taken) and writes connection details, including a per-launch auth token, to `server.json` in the nightly app data directory:

- **macOS**: `~/Library/Application Support/tldraw (Nightly)/server.json`
- **Windows**: `%APPDATA%/tldraw (Nightly)/server.json`
- **Linux**: `~/.config/tldraw (Nightly)/server.json`

### Endpoints

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/` or `/readme` | API documentation (plain text, no auth) |
| `POST` | `/api/search` | Search the Editor API reference and read live state (list documents, read shapes, screenshots) |
| `POST` | `/api/doc/:id/exec` | Run JavaScript against a document's live `Editor` |
| `POST` | `/api/doc/:id/script-workspace` | Expose a document's live script and asset paths for editing |
| `GET` | `/api/doc/:id/script-status` | Inspect the script watcher's apply state |

Every `/api` route requires the token from `server.json` as an `Authorization: Bearer <token>` header. The readme at `/` is public and self-documenting — start there.

### Example usage

```bash
# Read connection info (macOS)
SERVER_JSON="$HOME/Library/Application Support/tldraw (Nightly)/server.json"
PORT=$(jq -r .port "$SERVER_JSON")
TOKEN=$(jq -r .token "$SERVER_JSON")

# API documentation (no auth required)
curl "http://localhost:$PORT/readme"

# List open documents
curl -X POST "http://localhost:$PORT/api/search" \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"code": "return await api.getDocs()"}'

# Run code against a document's live editor
curl -X POST "http://localhost:$PORT/api/doc/{id}/exec" \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"code": "return editor.getCurrentPageShapeIds().size"}'
```

## License

tldraw offline nightly is not open source. All rights reserved.
