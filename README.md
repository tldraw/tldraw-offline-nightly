# tldraw desktop (staging)

The staging channel for [tldraw desktop](https://github.com/tldraw/tldraw-desktop) — built continuously from the latest source so the team can dogfood unreleased changes in a packaged build before they ship to stable users.

Staging is intended for **internal/team use**. If you're an end user, install the stable build from [tldraw-desktop releases](https://github.com/tldraw/tldraw-desktop/releases/latest) instead.

## Install

Get the latest staging release from the [Releases page](https://github.com/tldraw/tldraw-desktop-staging/releases).

| Platform | Download |
| --- | --- |
| macOS (Apple Silicon + Intel) | `tldraw-staging-{version}-universal.dmg` |
| Windows x64 | `tldraw-staging-{version}-win-x64.exe` |
| Windows ARM64 | `tldraw-staging-{version}-win-arm64.exe` |
| Linux x64 | `tldraw-staging-{version}-linux-x64.AppImage` or `.deb` |
| Linux ARM64 | `tldraw-staging-{version}-linux-arm64.AppImage` |

Staging installs **side-by-side** with stable (different bundle id, different name, different icon) — installing staging does not touch your stable install or its data.

## Auto-updates

Staging uses a **silent update flow**: the app checks for updates in the background, downloads them automatically, and installs the new version on next quit. No dialogs. This is intended for low-friction dogfooding — if you'd rather be prompted before each update, run stable.

## Local Canvas API

The staging app runs the same local HTTP server as stable, exposing a Canvas API for programmatic access to your tldraw documents.

The server starts automatically when the app launches (default port 7236, falls back to a random port if taken). Connection details are written to:

- **macOS**: `~/Library/Application Support/tldraw-staging/server.json`
- **Windows**: `%APPDATA%/tldraw-staging/server.json`
- **Linux**: `~/.config/tldraw-staging/server.json`

The `server.json` file contains:

```json
{
  "port": 7236,
  "pid": 12345
}
```

### API endpoints

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/` | API documentation (plain text) |
| `GET` | `/api/llms` | tldraw SDK documentation (llms-full.txt) |
| `GET` | `/api/doc` | List all open documents (supports `?name=` filter) |
| `GET` | `/api/doc/:id/shapes` | Get all shapes on the current page |
| `GET` | `/api/doc/:id/screenshot` | Screenshot of the canvas as JPEG |
| `POST` | `/api/doc/:id/exec` | Execute arbitrary editor code |
| `POST` | `/api/doc/:id/actions` | Execute structured canvas actions |

### Screenshots

`GET /api/doc/:id/screenshot` supports query parameters:

- `size` - `small` (768px), `medium` (1536px), `large` (3072px), `full` (5000px)
- `bounds` - Crop to specific area: `bounds=x,y,w,h`

### Actions

`POST /api/doc/:id/actions` accepts a JSON body with an `actions` array. Each action has a `_type` field:

`create`, `update`, `delete`, `clear`, `move`, `place`, `label`, `align`, `distribute`, `stack`, `bringToFront`, `sendToBack`, `resize`, `rotate`, `pen`, `setMyView`

### Example usage

```bash
# Read server connection info
cat ~/Library/Application\ Support/tldraw-staging/server.json

# List open documents
curl http://localhost:7236/api/doc

# Get shapes from a document
curl http://localhost:7236/api/doc/{id}/shapes

# Take a screenshot
curl http://localhost:7236/api/doc/{id}/screenshot?size=medium -o screenshot.jpg

# Execute editor code
curl -X POST http://localhost:7236/api/doc/{id}/exec \
  -H 'Content-Type: application/json' \
  -d '{"code": "return editor.getCurrentPageShapeIds().size"}'
```

## Development

Source code lives in the [`tldraw-internal`](https://github.com/tldraw/tldraw-internal) monorepo at `apps/public/desktop/`. Staging builds are produced by the `release-desktop-staging.yml` workflow with `STAGING=true` at build time, which swaps the bundle id, product name, icon, and update channel.

## License

This project is not open source. All rights reserved.
