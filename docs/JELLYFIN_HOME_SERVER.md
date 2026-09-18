# Jellyfin Home Media Server (Jellyfin + Prowlarr + Sonarr + Radarr + Jellyseerr + qBittorrent)

This guide turns this desktop into a self-hosted media server for your home
network: Jellyfin streams your library to TVs/phones/browsers, Prowlarr
manages your indexers, Sonarr/Radarr automatically fetch TV and movies,
qBittorrent does the downloading, and Jellyseerr lets your household request
new titles without needing access to the *arr apps directly.

Everything runs in Docker on this machine and is reachable from any device
on your home Wi-Fi/LAN. It is **not** exposed to the internet by default —
see [Remote access](#remote-access-optional) if you want that later.

## Architecture

```
                    Your home network (Wi-Fi/LAN)
                             |
        -------------------------------------------
        |            |            |         |      |
      phone         TV         laptop     tablet  ...
        \____________|____________|_________|_____/
                             |
                 this desktop (Docker host)
   +-------------------------------------------------------+
   | Jellyfin  <---- streams media to all the clients above |
   |    ^                                                   |
   |    | libraries read from /media/movies, /media/tv      |
   |    |                                                   |
   | Sonarr / Radarr --sync indexers--> Prowlarr            |
   |    |                                     |              |
   |    +--sends downloads to--> qBittorrent  |              |
   |                                                         |
   | Jellyseerr --requests--> Sonarr / Radarr, reads Jellyfin|
   +-------------------------------------------------------+
```

| Service     | Role                              | Default port |
|-------------|------------------------------------|---------------|
| Jellyfin    | Media server / streaming           | 8096          |
| Prowlarr    | Indexer manager                    | 9696          |
| Sonarr      | TV show automation                 | 8989          |
| Radarr      | Movie automation                   | 7878          |
| Jellyseerr  | Request management (like a "store")| 5055          |
| qBittorrent | Download client (torrents)         | 8080          |
| SABnzbd     | Download client (Usenet, optional) | 8081          |

## Prerequisites

- This desktop stays on and connected to your home network while you want
  the server available.
- [Docker Engine and the Docker Compose plugin](https://docs.docker.com/engine/install/)
  installed (`docker compose version` should work).
- Enough free disk space for your media library, separate from the OS disk
  if possible.
- Your desktop's local IP address is stable. Either set a static IP or
  create a DHCP reservation in your router so it doesn't change and break
  bookmarks/apps later.

## Step 1: Lay out your directories

Pick a location for everything, e.g. `~/media-server`, then:

```bash
mkdir -p ~/media-server && cd ~/media-server
mkdir -p jellyfin/config jellyfin/cache \
  prowlarr/config sonarr/config radarr/config \
  jellyseerr/config qbittorrent/config sabnzbd/config \
  media/movies media/tv downloads
```

Copy the compose file and (optionally) the Nginx config from this repo into
that directory:

```bash
cp /path/to/radarr-nginx-proxy/examples/docker-compose.jellyfin-stack.yml ~/media-server/
mkdir -p ~/media-server/nginx
cp /path/to/radarr-nginx-proxy/nginx/jellyfin-stack.conf ~/media-server/nginx/
```

Run `id` and edit the `PUID`/`PGID`/`TZ` values in
`docker-compose.jellyfin-stack.yml` to match your user, so containers write
files your account can read/delete.

## Step 2: Start the stack

```bash
cd ~/media-server
docker compose -f docker-compose.jellyfin-stack.yml up -d
```

Check everything is healthy:

```bash
docker compose -f docker-compose.jellyfin-stack.yml ps
```

Find this desktop's LAN IP (you'll use it from other devices):

```bash
ip addr show | grep 'inet ' | grep -v 127.0.0.1   # Linux
# or: hostname -I
```

You should now be able to open, from any device on the same network:

- Jellyfin: `http://<lan-ip>:8096`
- Prowlarr: `http://<lan-ip>:9696`
- Sonarr: `http://<lan-ip>:8989`
- Radarr: `http://<lan-ip>:7878`
- Jellyseerr: `http://<lan-ip>:5055`
- qBittorrent: `http://<lan-ip>:8080` (default login `admin` / a generated
  password — run `docker logs qbittorrent` once to find the temporary
  password, then change it immediately under WebUI settings)

## Step 3: Set up Prowlarr (indexers)

1. Open Prowlarr and finish the first-run setup, setting an authentication
   method (don't leave it as "None" once this is reachable on your LAN).
2. **Settings → Indexers → Add Indexer**: add whichever indexers you have
   access to (public trackers, private trackers you're a member of, or
   Usenet indexers). Prowlarr just manages the connection — it doesn't
   ship indexers itself.
3. **Settings → Download Clients → Add**: add qBittorrent
   (host `qbittorrent`, port `8080`), and SABnzbd too if you started that
   optional profile.
4. **Settings → Apps → Add Application**: add Sonarr and Radarr here.
   - Sonarr: Prowlarr Server = `http://prowlarr:9696`, Sonarr Server =
     `http://sonarr:8989`, plus the Sonarr API key (Sonarr → Settings →
     General).
   - Radarr: same idea with `http://radarr:7878` and its API key.
5. Click **Sync App Indexers** — Prowlarr pushes your indexer list into
   Sonarr and Radarr automatically, so you only manage indexers in one
   place going forward.

## Step 4: Set up qBittorrent

1. Log in, change the default password immediately
   (**Tools → Options → Web UI**).
2. **Options → Downloads**: set default save path to `/downloads`
   (matches the volume mount both Sonarr and Radarr also see).
3. Optional but recommended: create categories `tv-sonarr` and
   `radarr-movies` under the Torrents view, matching the categories you'll
   set in Sonarr/Radarr below — this keeps imports organized.

## Step 5: Set up Sonarr and Radarr

For each app:

1. **Settings → Media Management**: add a root folder —
   Sonarr → `/tv`, Radarr → `/movies`. Enable "Rename Episodes/Movies" if
   you want consistent file naming.
2. **Settings → Download Clients**: add qBittorrent
   (host `qbittorrent`, port `8080`, the category from Step 4).
3. **Settings → Indexers**: should already be populated from the Prowlarr
   sync in Step 3. If not, click sync again in Prowlarr.
4. **Settings → General → Security**: set an authentication method.
5. Add a show/movie from the top-level search to confirm a search and grab
   works end-to-end.

## Step 6: Set up Jellyfin

1. Complete the setup wizard, create your admin account.
2. **Dashboard → Libraries → Add Media Library**:
   - Content type "Shows" → folder `/data/tv`
   - Content type "Movies" → folder `/data/movies`
3. **Dashboard → Playback**: if this desktop has a compatible GPU, enable
   hardware acceleration here for smoother transcoding to weaker devices
   (phones, some smart TVs). CPU-only transcoding works but limits how many
   simultaneous streams you can transcode at once.
4. Create a user account per household member (**Dashboard → Users**) so
   everyone gets their own watch history and profile.

## Step 7: Set up Jellyseerr

1. On first run, point Jellyseerr at your Jellyfin server
   (`http://jellyfin:8096`) and sign in with a Jellyfin account.
2. **Settings → Services**: add Sonarr and Radarr
   (`http://sonarr:8989` / `http://radarr:7878`, plus their API keys and the
   root folders/quality profiles from Step 5).
3. Invite household members as Jellyseerr users (they can sign in with
   their own Jellyfin account) so they can request shows/movies without
   touching Sonarr, Radarr, or Prowlarr directly. Approved requests flow
   straight into Sonarr/Radarr and show up in Jellyfin once downloaded.

## Step 8: Connect your devices

- **Smart TVs / Roku / Fire TV / Android TV / Apple TV**: install the
  official Jellyfin app and point it at `http://<lan-ip>:8096`.
- **Phones/tablets**: Jellyfin app, same address (network discovery via UDP
  7359 should also auto-find it on the same Wi-Fi).
- **Browser**: `http://<lan-ip>:8096` for Jellyfin,
  `http://<lan-ip>:5055` for Jellyseerr requests.

## Optional: single-entry-point Nginx proxy

If you'd rather give people one URL instead of five ports, start the
bundled Nginx profile:

```bash
docker compose -f docker-compose.jellyfin-stack.yml --profile proxy up -d
```

This fronts Jellyfin, Prowlarr, Sonarr, Radarr, and qBittorrent behind
`http://<lan-ip>/jellyfin`, `/prowlarr`, `/sonarr`, `/radarr`,
`/qbittorrent` (see `nginx/jellyfin-stack.conf`). Each app must be told its
own base URL to match, as noted in the config comments — Sonarr/Radarr's
**Settings → General → URL Base**, Prowlarr's **Settings → General → URL
Base**, Jellyfin's **Dashboard → Networking → Base URL**.

Jellyseerr is deliberately left off the proxy — its web app does not
reliably support running under a subpath, so keep using
`http://<lan-ip>:5055` for it directly. This is a limitation of Jellyseerr
itself, not this config.

If you want a proper domain name and HTTPS instead of the raw LAN IP, the
existing Radarr-focused [SETUP.md](SETUP.md) walks through Certbot/Let's
Encrypt — the same approach applies here, but note Let's Encrypt needs a
publicly resolvable domain, which usually means also doing the
[remote access](#remote-access-optional) setup below.

## Updating

```bash
cd ~/media-server
docker compose -f docker-compose.jellyfin-stack.yml pull
docker compose -f docker-compose.jellyfin-stack.yml up -d
```

## Remote access (optional)

This setup is intentionally LAN-only by default — nothing here forwards
ports on your router. If you later want to watch/manage things away from
home, prefer a private overlay network (e.g. Tailscale or WireGuard)
installed on this desktop rather than forwarding Jellyfin/Sonarr/Radarr/
qBittorrent ports directly on your router: those apps are not hardened for
direct internet exposure, and an open qBittorrent or Sonarr/Radarr instance
is a common target for scanners. If you do choose to expose something
directly, put it behind Nginx with HTTPS and strong authentication, and
expose only Jellyfin — never the *arr apps or qBittorrent.

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for general reverse-proxy
issues. A few issues specific to this stack:

- **Choppy playback on phones/TVs**: usually means Jellyfin is transcoding
  in software. Check **Dashboard → Playback** for hardware acceleration
  options for your CPU/GPU, or pick a lower quality on the client.
- **"Permission denied" writing to `/media` or `/downloads`**: the
  `PUID`/`PGID` in the compose file don't match the owner of those host
  directories. Run `id` on the host and `chown -R <uid>:<gid>` the
  directories, or fix the compose file and `docker compose up -d` again.
- **Sonarr/Radarr can't reach qBittorrent or Prowlarr**: use the Docker
  service name (`qbittorrent`, `prowlarr`), not `localhost`, as the host —
  containers on the `media-network` reach each other by service name.
- **Jellyseerr shows no results / can't add a request**: double-check the
  Sonarr/Radarr API keys and root folder/quality profile mapping under
  Jellyseerr's **Settings → Services**.
