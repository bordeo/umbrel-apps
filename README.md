# Bordeo Umbrel Community App Store

Community app store for [umbrelOS](https://umbrel.com) (`id: bordeo`).

## Add this store in Umbrel

1. Open **App Store**
2. Click **⋯** (three dots)
3. Choose **Community App Stores**
4. Paste the store URL:

   `https://github.com/bordeo/umbrel-apps`

5. Install apps from the **Bordeo** section

## Apps

### `bordeo-scrypted` — Scrypted

High-performance NVR / HomeKit video bridge ([Scrypted](https://www.scrypted.app/)).

| | |
|---|---|
| Image | `koush/scrypted:v0.147.0-noble-full` (pinned by tag **and** sha256 digest) |
| Flavour | `-noble-full` (current upstream `full` / `latest`; replaces older `-jammy-full`) |
| Host HTTP | `11080` (`SCRYPTED_INSECURE_PORT`) |
| Host HTTPS | `10443` (`SCRYPTED_SECURE_PORT`) |
| Umbrel Open / app_proxy | `21080` → `$GATEWAY_IP:11080` |
| Data volume | `${APP_DATA_DIR}/data/scrypted` → `/server/volume` |

**Host networking:** required for HomeKit mDNS and pairing. The HomeKit plugin keeps its default **Ciao** advertiser. Do **not** stop host avahi and do **not** set `SCRYPTED_DOCKER_AVAHI`.

**Updates:** Watchtower / Docker socket are not used. Bump image tags via PRs (see automation below) and update through Umbrel.

### Migrate from `denny-scrypted`

Documentation only — run these on your Umbrel host (SSH), not from this repo.

1. **Stop** the old app in the Umbrel UI (`denny-scrypted`).
2. **Copy** the data directory (same volume layout):

   ```bash
   # Adjust UMBREL_ROOT if your install path differs
   UMBREL_ROOT="${UMBREL_ROOT:-$HOME/umbrel}"
   mkdir -p "$UMBREL_ROOT/app-data/bordeo-scrypted/data"
   cp -a "$UMBREL_ROOT/app-data/denny-scrypted/data/scrypted" \
         "$UMBREL_ROOT/app-data/bordeo-scrypted/data/scrypted"
   ```

3. Add this community store (if needed), then **install** `bordeo-scrypted`.
4. Confirm cameras / HomeKit look correct, then **uninstall** `denny-scrypted`.

Do not run both apps at once — both bind host ports `11080` / `10443`.

## Automation

### Lint CI (`.github/workflows/lint.yml`)

On pull requests and pushes: YAML syntax check + `docker compose config` with dummy Umbrel env vars.

### Scrypted update checker (`.github/workflows/update-scrypted.yml`)

Runs **daily** and on **workflow_dispatch**. It looks for a newer stable `koush/scrypted` image of the same flavour (e.g. `-noble-full`) on Docker Hub (and cross-checks GitHub releases on `koush/scrypted`), skipping pre-releases / betas.

**4-day cooldown (configurable):** a release is only proposed once it has been published for at least **4 days** (Docker Hub tag `last_updated` and/or GitHub `published_at`). Newer builds younger than the cooldown are ignored until they age. If several qualify, the newest that is ≥ cooldown days old is chosen. Override with the `cooldown_days` workflow input / `COOLDOWN_DAYS` env (default `4`).

When an update qualifies, the workflow opens a PR (using `GITHUB_TOKEN` only: `contents: write`, `pull-requests: write`) that:

- bumps the image tag **and** sha256 digest in `bordeo-scrypted/docker-compose.yml`
- bumps `version` and refreshes `releaseNotes` in `bordeo-scrypted/umbrel-app.yml`

There is **no auto-merge** — the owner reviews and merges.

## Layout

```
umbrel-app-store.yml          # store id + name
bordeo-scrypted/
  umbrel-app.yml              # app manifest
  docker-compose.yml          # services
  data/.gitkeep
.github/workflows/
  lint.yml
  update-scrypted.yml
```

Follows the [Umbrel community app store template](https://github.com/getumbrel/umbrel-community-app-store).
