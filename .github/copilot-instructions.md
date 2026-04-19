# Luis815 Umbrel Community Store

This repository contains a personal Umbrel community app store.

## Structure

- `umbrel-app-store.yml`: Defines the app store metadata (ID, name).
- App directories: Named as `{store-id}-{app-id}` (e.g., `luis815-umbrel-ollama-amd`).
- Each app directory contains:
  - `umbrel-app.yml`: App metadata (name, description, version, etc.).
  - `docker-compose.yml`: Docker configuration for the app.

## Adding Apps

To add a new app:
1. Create a directory named `{store-id}-{app-id}`.
2. Add `umbrel-app.yml` with app details.
3. Add `docker-compose.yml` with the Docker setup.