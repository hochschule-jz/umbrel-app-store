# Julian's Apps — Umbrel Community App Store

A custom [Umbrel](https://umbrel.com) community app store.

## Apps

- **To Do** (`hochschule-jz-todo`) — a calm, self-hosted, single-user task manager backed by PostgreSQL.

## Add this store to your Umbrel

1. In umbrelOS, open the **App Store**, click the **⋯** menu (top-right) → **Community App Stores**.
2. Add this repository URL:
   ```
   https://github.com/hochschule-jz/umbrel-app-store
   ```
3. Open **Julian's Apps** and install **To Do**.

> umbrelOS pulls the prebuilt image from `ghcr.io/hochschule-jz/todo`, then starts the app + its PostgreSQL container. PostgreSQL data is stored under the app's data directory on the Pi's NVMe SSD, so it survives restarts and updates.

## Prerequisites (one-time, before first install)

The app image must exist and be publicly pullable:

1. Push the [app source repo](https://github.com/hochschule-jz/todo) and create a version tag to trigger the build:
   ```bash
   git tag v1.0.0 && git push origin v1.0.0
   ```
   The `Publish image` GitHub Action builds the **arm64** image and pushes
   `ghcr.io/hochschule-jz/todo:1.0.0` (+ `:latest`).
2. Make the package **public** so the Pi can pull it without credentials:
   GitHub → your profile → **Packages** → `todo` → **Package settings** → **Change visibility** → *Public*.

## Releasing a new version

1. Bump `version` in `hochschule-jz-todo/umbrel-app.yml` and the image tag in
   `hochschule-jz-todo/docker-compose.yml` (e.g. `:1.1.0`).
2. In the app repo: `git tag v1.1.0 && git push origin v1.1.0` (Action publishes the image).
3. Commit + push this store repo. umbrelOS will offer the update.
