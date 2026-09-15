# Multi-Service Reverse Proxy Setup Guide (Radarr + Sonarr)

## Step 1: Install Radarr and Sonarr

### Install Radarr (Movies)

```bash
wget https://github.com/Radarr/Radarr/releases/download/v5.0.0.9123/Radarr.develop.5.0.0.9123.linux-x64.tar.gz
tar -xzf Radarr.develop.*.linux-x64.tar.gz
sudo mv Radarr /opt/Radarr
sudo useradd -m radarr
sudo chown -R radarr:radarr /opt/Radarr
```

### Install Sonarr (TV Shows)

```bash
wget https://github.com/Sonarr/Sonarr/releases/download/v4.0.0.8000/Sonarr.develop.4.0.0.8000.linux-x64.tar.gz
tar -xzf Sonarr.develop.*.linux-x64.tar.gz
sudo mv Sonarr /opt/Sonarr
sudo useradd -m sonarr
sudo chown -R sonarr:sonarr /opt/Sonarr
```

### Using Docker (Recommended)

```bash
docker-compose -f examples/docker-compose.yml up -d
```

## Step 2: Configure Radarr (Movies)

1. **Start Radarr:**
   ```bash
   sudo systemctl start radarr
   ```

2. **Access Radarr:** http://localhost:7878

3. **Configure for Reverse Proxy:**
   - Go to Settings → General
   - Set **URL Base** to `/radarr`
   - Go to Security section
   - Set **Authentication** to your preferred method
   - Click Save

4. **Restart Radarr** to apply changes

## Step 3: Configure Sonarr (TV Shows)

1. **Start Sonarr:**
   ```bash
   sudo systemctl start sonarr
   ```

2. **Access Sonarr:** http://localhost:8989

3. **Configure for Reverse Proxy:**
   - Go to Settings → General
   - Set **URL Base** to `/sonarr`
   - Go to Security section
   - Set **Authentication** to your preferred method
   - Click Save

4. **Restart Sonarr** to apply changes

## Step 4: Install Nginx

### On Linux (Ubuntu/Debian):
```bash
sudo apt-get update
sudo apt-get install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

## Step 5: Configure Nginx for Both Services

1. **Copy both configurations:**
   ```bash
   sudo cp nginx/radarr.conf /etc/nginx/sites-available/radarr
   sudo cp nginx/sonarr.conf /etc/nginx/sites-available/sonarr
   sudo ln -s /etc/nginx/sites-available/radarr /etc/nginx/sites-enabled/radarr
   sudo ln -s /etc/nginx/sites-available/sonarr /etc/nginx/sites-enabled/sonarr
   ```

2. **Edit configurations:**
   ```bash
   sudo nano /etc/nginx/sites-available/radarr
   sudo nano /etc/nginx/sites-available/sonarr
   ```
   - Replace `yourdomain.com` with your actual domain

3. **Test Nginx configuration:**
   ```bash
   sudo nginx -t
   ```

4. **Reload Nginx:**
   ```bash
   sudo systemctl reload nginx
   ```

## Step 6: Set Up SSL/TLS (Production)

### Install Certbot:
```bash
sudo apt-get install certbot python3-certbot-nginx
```

### Generate Certificate:
```bash
sudo certbot certonly --nginx -d yourdomain.com -d www.yourdomain.com
```

### Update Nginx Configurations:
1. Copy SSL configs:
   ```bash
   sudo cp nginx/radarr-ssl.conf /etc/nginx/sites-available/radarr
   sudo cp nginx/sonarr-ssl.conf /etc/nginx/sites-available/sonarr
   ```

2. Edit both configs and update certificate paths:
   ```bash
   sudo nano /etc/nginx/sites-available/radarr
   sudo nano /etc/nginx/sites-available/sonarr
   ```

3. Test and reload:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

## Step 7: Configure Firewall

### UFW (Ubuntu):
```bash
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
```

## Step 8: Verify Setup

1. **Check Nginx status:**
   ```bash
   sudo systemctl status nginx
   ```

2. **Check both services:**
   ```bash
   sudo systemctl status radarr
   sudo systemctl status sonarr
   ```

3. **Test from browser:**
   - Radarr: http://yourdomain.com/radarr
   - Sonarr: http://yourdomain.com/sonarr

4. **Check logs:**
   ```bash
   sudo tail -f /var/log/nginx/radarr_error.log
   sudo tail -f /var/log/nginx/sonarr_error.log
   ```

## Media Library Structure

Recommended directory layout:

```
/media/
├── movies/               # Radarr library
│   ├── Action/
│   ├── Comedy/
│   ├── Drama/
│   └── Horror/
├── tv/                   # Sonarr library
│   ├── Breaking Bad/
│   ├── Game of Thrones/
│   ├── The Office/
│   └── Stranger Things/
└── downloads/            # Download staging
    ├── radarr/          # Radarr incomplete downloads
    └── sonarr/          # Sonarr incomplete downloads
```

### Create directories:
```bash
mkdir -p /media/movies
mkdir -p /media/tv
mkdir -p /media/downloads/radarr
mkdir -p /media/downloads/sonarr

# Set appropriate permissions
sudo chown -R radarr:radarr /media/movies
sudo chown -R radarr:radarr /media/downloads/radarr
sudo chown -R sonarr:sonarr /media/tv
sudo chown -R sonarr:sonarr /media/downloads/sonarr
```

## Service Management Commands

### Radarr
```bash
sudo systemctl start radarr
sudo systemctl stop radarr
sudo systemctl restart radarr
sudo systemctl status radarr
sudo journalctl -u radarr -f
```

### Sonarr
```bash
sudo systemctl start sonarr
sudo systemctl stop sonarr
sudo systemctl restart sonarr
sudo systemctl status sonarr
sudo journalctl -u sonarr -f
```

### Nginx
```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl status nginx
sudo nginx -t
```

## Updating Applications

### Update Radarr:
```bash
sudo systemctl stop radarr
cd /opt/Radarr && ./update.sh
sudo systemctl start radarr
```

### Update Sonarr:
```bash
sudo systemctl stop sonarr
cd /opt/Sonarr && ./update.sh
sudo systemctl start sonarr
```

### Update Nginx:
```bash
sudo apt-get update
sudo apt-get upgrade nginx
sudo systemctl reload nginx
```

## Troubleshooting

### Both services won't start
- Check disk space: `df -h`
- Check permissions: `ls -la /config`
- Check logs: `journalctl -u radarr -u sonarr -n 50`

### Can't access through domain
- Verify DNS: `nslookup yourdomain.com`
- Check firewall: `sudo ufw status`
- Check Nginx: `sudo nginx -t && sudo systemctl reload nginx`

### SSL certificate errors
- Renew certificate: `sudo certbot renew --force-renewal`
- Verify paths in Nginx configs
- Restart Nginx: `sudo systemctl restart nginx`

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for more detailed solutions.

## Next Steps

1. Configure download clients in both services
2. Set up indexers/providers
3. Configure media management settings
4. Add movies and TV shows to your library
5. Enable monitoring and notifications
6. Review security settings
7. Set up automated backups
