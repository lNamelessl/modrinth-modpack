# Deploy and Host

Deploy a full modded Minecraft server from any [Modrinth](https://modrinth.com/modpacks) modpack. Paste a pack URL, project slug, or version URL into the one deploy prompt — the server downloads and installs the entire pack (mods, configs, loader) on first start, then boots into a joinable server with RCON enabled.

**[Deploy on Railway](https://railway.com/deploy/modrinth-modpack)**

## About Hosting

- **One prompt:** `MODRINTH_MODPACK` — any Modrinth modpack reference: a project page URL (`https://modrinth.com/modpack/...`), a version page URL (pins that exact version), a project slug (`cobblemon-fabric`), or a project ID. The pack install runs on first boot: a light pack boots in about 2 to 5 minutes, larger packs take longer.
- **Java base:** the image pins `itzg/minecraft-server:2026.10.1-java21`. This covers the pack ecosystem for Minecraft 1.20.5 through 1.21.x (Java 21), which is the vast majority of current Modrinth packs. Packs targeting Minecraft 26.x need Java 25: after deploy, swap the service image to `itzg/minecraft-server:2026.10.1-java25` (same digest-pinning pattern) in Settings. Packs are Java-pinned by their mods, so a mismatch crashes at boot with a clear class-version or `depends java` error in the deploy logs.
- **Memory:** `MEMORY` defaults to 4 GB (set `MEMORY` before first deploy for heavy packs). Measured reference: the Cobblemon 1.21.1 pack idles at about 3.7 GB RSS with a 4 GB heap. Guidance: light SMP or performance packs 4 to 6 GB, mid-size packs 6 to 8 GB, big kitchen-sink packs 12 to 16 GB.
- **Cost honesty:** Railway bills 24/7 server uptime by the second. At typical rates, 4 GB 24/7 costs roughly 35 to 45 USD per month, 8 GB roughly 70 to 90 USD per month, 16 GB roughly 150 to 180 USD per month — specialist Minecraft hosts are cheaper for always-on heavy packs. Railway wins for intermittent servers (friend groups playing evenings and weekends) where per-second billing and instant deploy/destroy beat a monthly subscription, and for packs that need to live next to your other services.
- **Persistence:** a volume is mounted at `/data` — world, mods, configs, and loader all survive redeploys and restarts.
- **EULA:** the Minecraft EULA is accepted by the image (`EULA=TRUE`) so the server boots unattended. By hosting a server you accept the [Minecraft EULA](https://account.mojang.com/documents/minecraft_eula).
- **RCON:** enabled on port 25575, internal only. The generated password is in the Variables tab (`RCON_PASSWORD`, random per deploy). Use it with any RCON client or an MCP RCON control template to send commands (`list`, `say`, `whitelist`, backups) without opening the game.

## Why Deploy

- **Paste-a-pack simplicity:** every other Minecraft template here starts from a bare server or a panel. This one installs the modpack you point it at — the pack choice is the only decision.
- **Verified install path:** the deploy battery proves the whole chain: pack download and install in the logs, the `Done (X.XXXs)!` boot line, a live RCON round-trip, a Minecraft status ping through the public join address, and world-plus-mods survival across redeploys.
- **Pack updates are a variable edit:** change `MODRINTH_MODPACK` to a newer version URL and restart — the installer cleans up the previous pack's files and installs the new one.
- **Ops built in:** crash-only restart policy (no fake healthcheck for a non-HTTP game port), per-second billing, volumes, and RCON for scripted control.

## Common Use Cases

- Friend-group modded servers: paste the pack your group plays, share the join address from the Variables tab (`RAILWAY_TCP_PROXY_DOMAIN:RAILWAY_TCP_PROXY_PORT`), stop paying when the group moves on.
- Testing a modpack server-side before inviting anyone: deploy the pack, check the logs, connect once, destroy the project — you pay only for the minutes used.
- Version-pinned community servers: use a version page URL so the pack never drifts until you choose to bump it.
- Automated servers: RCON (password in Variables) drives commands, announcements, and backup hooks from scripts or an MCP control template.

## Dependencies for

### Deployment Dependencies

- **A Modrinth modpack reference** for `MODRINTH_MODPACK` — browse [modrinth.com/modpacks](https://modrinth.com/modpacks). Pick packs marked playable server-side; most popular packs (Cobblemon, Better MC, Prominence, and similar) are. The install fails fast with a clear log line if a pack is client-only or references unavailable files.
- **No other services needed** — single service, no database. The world lives on the attached volume.
- **Clients need the same pack:** players install the matching pack from Modrinth (the Modrinth app handles this) and join via the TCP proxy address shown in the Variables tab.
- **Java pairing:** the pinned image is Java 21 for Minecraft 1.20.5 to 1.21.x packs. For Minecraft 26.x packs, swap the image to the `java25` variant after deploy (see About Hosting).
