# Radarr vs Sonarr Quick Comparison

## At a Glance

| Feature | Radarr | Sonarr |
|---------|--------|--------|
| **Purpose** | Movie Management | TV Show Management |
| **Default Port** | 7878 | 8989 |
| **URL Base** | `/radarr` | `/sonarr` |
| **Database Type** | SQLite | SQLite |
| **Config Folder** | `/config` | `/config` |
| **Download Client Support** | Yes | Yes |
| **API Available** | Yes (v3) | Yes (v3) |
| **Notifications** | Yes | Yes |
| **Custom Scripts** | Yes | Yes |

## What Each Does

### Radarr (Movies) 🎬
- Searches for movies across multiple indexers
- Downloads movies automatically when available
- Organizes movies in your library folder
- Monitors for quality upgrades
- Renames and sorts files automatically
- Tracks your movie collection

**Example Flow:**
1. Add "The Dark Knight" to Radarr
2. Radarr searches for it automatically
3. Downloads the movie when available
4. Renames it: `The Dark Knight (2008).mkv`
5. Moves to: `/media/movies/The Dark Knight (2008)/`

### Sonarr (TV Shows) 📺
- Searches for TV shows across multiple indexers
- Downloads episodes automatically when aired
- Organizes TV shows in your library folder
- Monitors for quality upgrades
- Renames and sorts files automatically
- Tracks your TV show collection
- Understands seasons and episodes

**Example Flow:**
1. Add "Breaking Bad" to Sonarr
2. Sonarr searches for all seasons
3. Downloads episodes as they're available
4. Renames: `Breaking Bad - s01e01 - Pilot.mkv`
5. Moves to: `/media/tv/Breaking Bad/Season 01/`

## Key Differences

### File Organization
**Radarr:**
```
/movies/
├── The Dark Knight (2008)/
│   └── The Dark Knight (2008).mkv
├── Inception (2010)/
│   └── Inception (2010).mkv
└── Interstellar (2014)/
    └── Interstellar (2014).mkv
```

**Sonarr:**
```
/tv/
├── Breaking Bad/
│   ├── Season 01/
│   │   ├── Breaking Bad - s01e01 - Pilot.mkv
│   │   ├── Breaking Bad - s01e02 - Cat's in the Bag.mkv
│   │   └── ...
│   ├── Season 02/
│   │   ├── Breaking Bad - s02e01 - Seven Thirty-Seven.mkv
│   │   └── ...
│   └── ...
└── Game of Thrones/
    ├── Season 01/
    ├── Season 02/
    └── ...
```

### Settings & Configuration
Both have similar settings sections:
- **General** - Basic settings (URL Base, Port, Authentication)
- **Media Management** - File naming, organization
- **Indexers** - Where to search (torrents, usenet, etc.)
- **Download Clients** - How to download (transmission, qBittorrent, etc.)
- **Notifications** - Where to alert you (Discord, Telegram, etc.)
- **Connections** - External integrations
- **Backup** - Backup your configuration

### Episode vs Movie Awareness

**Radarr:**
- Just knows "this is a movie"
- Doesn't need season/episode logic
- Simpler matching and naming

**Sonarr:**
- Understands seasons and episodes
- Needs to download Season 1 Episode 1, Season 1 Episode 2, etc.
- Waits for weekly/daily releases
- More complex naming patterns

## Common Setup

Most people run both together:

```
yourdomain.com/radarr  → Radarr (Movies)
yourdomain.com/sonarr  → Sonarr (TV Shows)
```

They:
- Share the same Nginx reverse proxy
- Can share the same download client
- Keep movies and TV separate in `/media/`
- Run independently (if one fails, the other works)

## Access Points

### Local Network (No Reverse Proxy)
- Radarr: `http://192.168.1.100:7878`
- Sonarr: `http://192.168.1.100:8989`

### Through Reverse Proxy (This Setup)
- Radarr: `https://yourdomain.com/radarr`
- Sonarr: `https://yourdomain.com/sonarr`

### Docker (If Using Docker Compose)
- Radarr: `http://localhost:7878`
- Sonarr: `http://localhost:8989`
- Nginx: `http://localhost` (redirects to https)

## Which One to Use When?

| Scenario | Use |
|----------|-----|
| Want to download a specific movie? | Radarr |
| Want to automatically get new TV episodes? | Sonarr |
| Adding a movie collection? | Radarr |
| Adding a TV series? | Sonarr |
| Movie stops being available? | Radarr can upgrade quality |
| New season of a show airs? | Sonarr automatically handles it |

## Integration with Other Services

Both work with:
- **Download Clients:** qBittorrent, Transmission, Deluge, SABnzbd
- **Media Players:** Plex, Jellyfin, Kodi
- **Notifications:** Discord, Telegram, Pushbullet, Email
- **Home Automation:** Home Assistant, Node-RED
- **Indexers:** Prowlarr (centralized indexer management)

## Performance Impact

Both are fairly lightweight:
- **CPU:** Minimal (searching/downloading is light)
- **RAM:** ~100-200MB each
- **Disk:** Minimal (they store metadata)
- **Network:** Only active when searching/downloading

Running both together usually uses less than 500MB RAM total.

## Choosing Download Quality

Both let you set quality profiles:

**Radarr Example:**
- 720p (if available)
- 1080p (upgrade if found)
- 4K (ultimate)

**Sonarr Example:**
- 480p (acceptable)
- 720p (preferred)
- 1080p (upgrade if found)

You can have multiple profiles and assign different content to different profiles.

## API Usage

Both provide APIs for external integrations:

**Radarr API:**
```bash
curl -H "X-Api-Key: YOUR_KEY" \
  https://yourdomain.com/radarr/api/v3/movie
```

**Sonarr API:**
```bash
curl -H "X-Api-Key: YOUR_KEY" \
  https://yourdomain.com/sonarr/api/v3/series
```

## Common Questions

**Q: Can they share a download folder?**
A: Yes, but they'll see each other's files. Better to use separate folders: `/downloads/radarr/` and `/downloads/sonarr/`

**Q: Do they need the same authentication?**
A: No, you can set different auth for each via Nginx.

**Q: Can I access them from outside my network?**
A: Yes, that's what the reverse proxy setup is for!

**Q: What if one service crashes?**
A: The other keeps working. The reverse proxy will show an error for the broken one.

**Q: Can they share a download client?**
A: Yes! Just configure both to use the same client (qBittorrent, Transmission, etc.)

**Q: Which uses more resources?**
A: About the same. Sonarr might be slightly more active if you have many ongoing series.

## Next Steps

1. **Install both apps** (Docker or bare metal)
2. **Configure Nginx** to proxy both
3. **Set up SSL certificates**
4. **Configure download clients** in each
5. **Add indexers** (via Prowlarr or manually)
6. **Add content** (movies to Radarr, shows to Sonarr)
7. **Enable notifications** so you know when content downloads
8. **Monitor and enjoy!** 🎬📺
