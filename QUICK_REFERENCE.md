# Quick Reference Card

## Essential Commands

### Service Management
```bash
# Radarr
sudo systemctl start radarr
sudo systemctl stop radarr
sudo systemctl restart radarr
sudo systemctl status radarr

# Nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl status nginx
```

### Configuration Validation
```bash
# Test Nginx config
sudo nginx -t

# Show full config
sudo nginx -T

# Check listening ports
sudo netstat -tlnp | grep -E 'nginx|7878'
```

### Logs
```bash
# Nginx access log
tail -f /var/log/nginx/radarr_access.log

# Nginx error log
tail -f /var/log/nginx/radarr_error.log

# Radarr logs
tail -f /config/logs/radarr.txt

# System logs
sudo journalctl -u radarr -f
sudo journalctl -u nginx -f
```

### SSL/TLS Certificates
```bash
# List certificates
sudo certbot certificates

# Renew certificates
sudo certbot renew

# Force renewal
sudo certbot renew --force-renewal

# Test renewal
sudo certbot renew --dry-run
```

---

## Nginx Configuration Quick Edit

```bash
# Edit Radarr config
sudo nano /etc/nginx/sites-available/radarr

# Test changes
sudo nginx -t

# Apply changes
sudo systemctl reload nginx
```

---

## Common Configuration Changes

### Change URL Base
```bash
# 1. Edit Radarr config
sudo nano /etc/nginx/sites-available/radarr

# 2. Replace location path
# OLD: location /radarr
# NEW: location /movies

# 3. Restart services
sudo systemctl reload nginx
# Then restart Radarr and update URL Base setting
```

### Enable SSL/TLS
```bash
# 1. Get certificate
sudo certbot certonly --nginx -d yourdomain.com

# 2. Update Nginx config
sudo nano /etc/nginx/sites-available/radarr

# 3. Add certificate paths
# ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
# ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

# 4. Test and reload
sudo nginx -t
sudo systemctl reload nginx
```

### Add Basic Authentication
```bash
# 1. Create password
sudo htpasswd -c /etc/nginx/.htpasswd radarr_user

# 2. Update Nginx config
sudo nano /etc/nginx/sites-available/radarr

# 3. Add auth directives
# auth_basic "Radarr Access";
# auth_basic_user_file /etc/nginx/.htpasswd;

# 4. Reload
sudo systemctl reload nginx
```

---

## Troubleshooting Quick Fixes

### Can't Access Through Domain
```bash
# Check DNS
nslookup yourdomain.com

# Check firewall
sudo ufw status
sudo firewall-cmd --list-all

# Check Nginx listening
sudo netstat -tlnp | grep nginx

# Check Nginx config
sudo nginx -t
```

### 502 Bad Gateway
```bash
# Check Radarr is running
sudo systemctl status radarr

# Check Radarr port
netstat -tlnp | grep 7878

# Check Nginx error log
sudo tail -50 /var/log/nginx/radarr_error.log
```

### SSL Certificate Errors
```bash
# Check certificate validity
openssl x509 -in /etc/letsencrypt/live/yourdomain.com/fullchain.pem -text -noout

# Renew certificate
sudo certbot renew --force-renewal

# Reload Nginx
sudo systemctl reload nginx
```

### Missing CSS/JavaScript
```bash
# Clear browser cache
# (Ctrl+Shift+Delete on most browsers)

# Check URL Base setting in Radarr
# Settings → General → URL Base

# Verify Nginx location matches
grep "location" /etc/nginx/sites-available/radarr
```

---

## Performance Monitoring

### Check System Resources
```bash
# CPU and Memory
top
free -h
df -h

# Nginx process count
ps aux | grep nginx | wc -l

# Radarr process
ps aux | grep Radarr
```

### Monitor Requests
```bash
# Real-time request count
tail -f /var/log/nginx/radarr_access.log | wc -l

# Requests per second
awk '{print $4}' /var/log/nginx/radarr_access.log | \
  cut -d':' -f4 | sort | uniq -c | sort -rn | head -5

# Slowest endpoints
awk '{print $7, $(NF-2)}' /var/log/nginx/radarr_access.log | \
  sort -k2 -rn | head -10
```

### Check Upstream Connection
```bash
# Test connection to Radarr
curl -v http://localhost:7878/radarr

# Check response headers
curl -I http://localhost:7878/radarr

# Test through Nginx
curl -v http://localhost/radarr
```

---

## File Locations Reference

| Component | Config Location |
|-----------|-----------------|
| Nginx Config | `/etc/nginx/sites-available/radarr` |
| Nginx Cache | `/var/cache/nginx/` |
| Nginx Logs | `/var/log/nginx/` |
| Radarr Config | `/config/config.xml` |
| Radarr Logs | `/config/logs/` |
| SSL Certs | `/etc/letsencrypt/live/yourdomain.com/` |
| Password File | `/etc/nginx/.htpasswd` |
| Systemd Service | `/etc/systemd/system/radarr.service` |

---

## Port Reference

| Service | Port | Purpose |
|---------|------|---------|
| HTTP | 80 | Web traffic (redirects to HTTPS) |
| HTTPS | 443 | Encrypted web traffic |
| Radarr | 7878 | Local Radarr instance |

---

## Important Settings in Radarr

### General Settings
- **URL Base**: `/radarr` (must match Nginx location)
- **Bind Address**: `0.0.0.0`
- **Port**: `7878`
- **Enable SSL**: Usually disabled (Nginx handles SSL)

### Security Settings
- **Authentication**: Forms / Windows / External / None
- **API Key**: Used for external integrations
- **Require API Auth**: Recommended to enable

### Media Settings
- **Movie Paths**: Configure download and library paths
- **Download Clients**: Setup your download software
- **Connection Limits**: Default is usually fine

---

## Backup and Restore

### Quick Backup
```bash
# Backup Radarr
tar -czf radarr-backup-$(date +%Y%m%d).tar.gz /config

# Backup Nginx
tar -czf nginx-backup-$(date +%Y%m%d).tar.gz /etc/nginx

# Backup SSL Certs
tar -czf ssl-backup-$(date +%Y%m%d).tar.gz /etc/letsencrypt
```

### Quick Restore
```bash
# Stop services
sudo systemctl stop radarr nginx

# Extract backup
tar -xzf radarr-backup-20240101.tar.gz -C /

# Start services
sudo systemctl start radarr nginx
```

---

## Useful Links

- **Radarr Wiki**: https://wiki.servarr.com/radarr/
- **Nginx Docs**: https://nginx.org/en/docs/
- **Let's Encrypt**: https://letsencrypt.org/
- **Certbot**: https://certbot.eff.org/
- **GitHub Issues**: https://github.com/Radarr/Radarr/issues
- **Servarr Discord**: https://discord.gg/Qc5B39w

---

## Emergency Commands

### Kill and Restart
```bash
# Force kill Nginx
sudo pkill -9 nginx
sudo systemctl start nginx

# Force kill Radarr
sudo pkill -9 Radarr
sudo systemctl start radarr
```

### Reset Permissions
```bash
# Fix Radarr directory permissions
sudo chown -R radarr:radarr /config
sudo chmod -R 755 /config

# Fix Nginx permissions
sudo chown -R www-data:www-data /var/log/nginx
```

### Emergency Access
```bash
# Access Radarr locally if reverse proxy is broken
# Navigate to http://localhost:7878

# Access Nginx config if locked out
sudo nano /etc/nginx/sites-available/radarr
```

---

## Health Check URLs

```bash
# Radarr health
curl http://localhost:7878/radarr/health

# Radarr API (requires API key)
curl -H "X-Api-Key: YOUR_KEY" http://localhost:7878/radarr/api/v3/system/status

# Nginx status (if enabled)
curl http://localhost/nginx_status

# Through reverse proxy
curl https://yourdomain.com/radarr/health
```

---

## Update Procedures

### Update Radarr
```bash
# Check for updates in UI
# Settings → System → Updates

# Or manual update
sudo systemctl stop radarr
cd /opt/Radarr && ./update.sh
sudo systemctl start radarr
```

### Update Nginx
```bash
sudo apt-get update
sudo apt-get upgrade nginx
sudo systemctl reload nginx
```

### Update Certificates
```bash
sudo certbot renew
sudo systemctl reload nginx
```

---

## Getting Help

1. **Check logs first**
   - Nginx error log
   - Radarr logs
   - System journal

2. **Test locally**
   - `http://localhost:7878/radarr`

3. **Check configuration**
   - Verify URL Base
   - Check authentication
   - Validate Nginx syntax

4. **Search for solution**
   - Radarr GitHub Issues
   - Servarr Wiki
   - Stack Overflow

5. **Ask for help**
   - Radarr Discord
   - GitHub Issues
   - This repository's Issues
