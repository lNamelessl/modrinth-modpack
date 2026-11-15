# Modrinth Modpack Server on Railway

Paste a [Modrinth](https://modrinth.com/modpacks) modpack URL — get a full modded Minecraft server (mods, configs, loader) with RCON enabled, running on [Railway](https://railway.com).

**One-click deploy:** [![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/modrinth-modpack)

## How it works

The template runs the battle-tested [`itzg/minecraft-server`](https://github.com/itzg/docker-minecraft-server) image (digest-pinned) with `TYPE=MODRINTH`. At first boot the image downloads and installs the entire pack you point it at — mods, configs, mod loader — then boots the server. Everything lands on a persistent volume at `/data`.

## Deploy prompt

| Variable | Required | Meaning |
|---|---|---|
| `MODRINTH_MODPACK` | yes | Any Modrinth pack reference: project page URL, version page URL (pins that version), project slug, or project ID. Example: `cobblemon-fabric` |

Generated after deploy (no prompt): `RCON_PASSWORD` (random per deploy, Variables tab), `MEMORY` default 4 GB (change the value in the start command or set a `MEMORY` variable), MOTD.

## Post-deploy

1. **Watch first boot:** Deployments → View Logs shows the pack download/install, then `Done (X.XXXs)! For help, type "help"`. Light packs: 2 to 5 minutes.
2. **Join:** players connect at the TCP proxy address in the Variables tab (`RAILWAY_TCP_PROXY_DOMAIN`:`RAILWAY_TCP_PROXY_PORT`). Clients must run the same Modrinth pack.
3. **RCON:** port 25575, internal only — password in the Variables tab. `railway ssh` into the service and use the bundled `rcon-cli`.
4. **Change packs:** set `MODRINTH_MODPACK` to another pack/version and restart. The installer removes the previous pack's files and installs the new one. Your generated world survives unless the new pack ships a world of its own — wipe the volume for a guaranteed fresh start.

## Memory and cost (honesty table)

| Pack tier | MEMORY | Railway 24/7 cost (approx) | Notes |
|---|---|---|---|
| Light (SMP, performance) | 4 to 6 GB | ~35 to 45 USD/mo | Cobblemon measured ~3.7 GB RSS at boot |
| Mid | 6 to 8 GB | ~70 to 90 USD/mo | |
| Heavy (kitchen-sink) | 12 to 16 GB | ~150 to 180 USD/mo | specialist hosts cheaper for always-on; Railway wins for intermittent use (per-second billing) |

## Java pairing

The image is pinned `itzg/minecraft-server:2026.10.1-java21@sha256:07ab333a8542208c4c863a78fd3cf3fd1138d5a2e118306e1a316d202f69ab74` — Minecraft 1.20.5 to 1.21.x packs (the current Modrinth majority). Packs for Minecraft 26.x need Java 25: swap the service image to `itzg/minecraft-server:2026.10.1-java25@sha256:d12aed7e35981dfe259b60fbfee77d23a19b5898a7a31dca298d18b47b7b5776` after deploy. A Java mismatch crashes at boot with a clear class-version error.

Mojang EULA is accepted by the image so the server boots unattended; hosting a server means you accept the [Minecraft EULA](https://account.mojang.com/documents/minecraft_eula).

## Repo contents

- `TEMPLATE_OVERVIEW.md` — the marketplace listing overview
- `railway.json` — restart policy (ON_FAILURE, 10 retries); deliberately no HTTP healthcheck (the game port is raw TCP)
- The template config is image-sourced: `serializedConfig` carries the pinned image, the `sh -c` start command with baked `TYPE=MODRINTH`/`EULA=TRUE`/`MEMORY` defaults, the `/data` volume, and the 25565 TCP proxy
