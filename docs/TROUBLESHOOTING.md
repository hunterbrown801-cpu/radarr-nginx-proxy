# Troubleshooting Guide

## Common Issues and Solutions

### Issue: "Not Authorized" or "Unauthorized Access"

**Symptoms:**
- Getting 401/403 errors after setting up reverse proxy
- Can access locally but not through domain
- Authentication prompt appears unexpectedly

**Solutions:**
1. **Check Radarr URL Base:**
   - Settings → General → URL Base should match your Nginx location path (e.g., `/radarr`)
   - Restart Radarr after changes

2. **Verify X-Forwarded headers in Nginx:**
   ```nginx
   proxy_set_header X-Forwarded-Proto $scheme;
   proxy_set_header X-Forwarded-Host $server_name;
   ```

3. **Review Radarr Authentication:**
   - Settings → General → Security
   - If using "None", ensure reverse proxy handles auth
   - If using "Forms", check username/password

4. **Check Radarr logs:**
   ```bash
   tail -f /config/logs/radarr.txt
   # Or Docker
   docker logs radarr
   ```

---

### Issue: CSS/JavaScript/Images Not Loading

**Symptoms:**
- Page loads but looks broken
- Browser console shows 404 errors for static assets
- CSS is not applied, images are missing

**Solutions:**
1. **Verify URL Base matches:**
   - Radarr setting: `/radarr`
   - Nginx location: `location /radarr`
   - They must match exactly

2. **Clear browser cache:**
   - Chrome: Ctrl+Shift+Delete
   - Firefox: Ctrl+Shift+Delete
   - Safari: Cmd+Shift+Delete

3. **Check browser console (F12):**
   - Look for 404 errors
   - Note the requested URLs
   - Verify they're being proxied correctly

4. **Test with curl:**
   ```bash
   curl -I https://yourdomain.com/radarr/content/images/logo.png
   ```

---

### Issue: Cannot Connect to Domain

**Symptoms:**
- Connection timeout
- "This site can't be reached"
- DNS errors

**Solutions:**
1. **Verify DNS resolution:**
   ```bash
   nslookup yourdomain.com
   dig yourdomain.com
   ```

2. **Check if Nginx is listening:**
   ```bash
   sudo netstat -tlnp | grep nginx
   # Should show 0.0.0.0:80 and 0.0.0.0:443
   ```

3. **Verify firewall rules:**
   ```bash
   # UFW
   sudo ufw status
   
   # Firewalld
   sudo firewall-cmd --list-all
   ```

4. **Check Nginx error log:**
   ```bash
   sudo tail -f /var/log/nginx/radarr_error.log
   ```

5. **Test locally first:**
   ```bash
   curl http://localhost/radarr
   ```

---

### Issue: SSL Certificate Errors

**Symptoms:**
- "ERR_SSL_PROTOCOL_ERROR"
- "SSL_ERROR_RX_RECORD_TOO_LONG"
- Browser shows certificate warnings

**Solutions:**
1. **Verify certificate exists:**
   ```bash
   sudo certbot certificates
   ```

2. **Check certificate validity:**
   ```bash
   openssl x509 -in /etc/letsencrypt/live/yourdomain.com/fullchain.pem -text -noout
   ```

3. **Ensure Nginx points to correct certificate:**
   ```bash
   grep ssl_certificate /etc/nginx/sites-available/radarr
   ```

4. **Renew certificate:**
   ```bash
   sudo certbot renew --force-renewal
   ```

5. **Test SSL configuration:**
   ```bash
   sudo nginx -t
   ```

6. **Restart Nginx:**
   ```bash
   sudo systemctl restart nginx
   ```

---

### Issue: "Bad Gateway" or "502" Errors

**Symptoms:**
- Nginx 502 Bad Gateway error
- Upstream connection errors in logs
- Radarr seems to be running but proxy fails

**Solutions:**
1. **Verify Radarr is running:**
   ```bash
   # Direct service
   sudo systemctl status radarr
   
   # Docker
   docker ps | grep radarr
   ```

2. **Check if Radarr is listening on port 7878:**
   ```bash
   netstat -tlnp | grep 7878
   telnet localhost 7878
   ```

3. **Verify Nginx upstream configuration:**
   ```bash
   grep -A2 "upstream radarr" /etc/nginx/sites-available/radarr
   # Should show: server localhost:7878;
   ```

4. **Check Nginx error logs:**
   ```bash
   sudo tail -f /var/log/nginx/radarr_error.log
   ```

5. **Check Radarr logs:**
   ```bash
   tail -f /config/logs/radarr.txt
   ```

6. **Test proxy connection:**
   ```bash
   curl -v http://localhost:7878/radarr
   ```

---

### Issue: WebSocket Connection Fails

**Symptoms:**
- Real-time updates don't work
- Live search doesn't function
- SignalR connection errors in console

**Solutions:**
1. **Verify WebSocket configuration in Nginx:**
   ```nginx
   location /radarr/signalr {
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
   }
   ```

2. **Check Radarr logs for WebSocket errors:**
   ```bash
   grep -i "websocket\|signalr" /config/logs/radarr.txt
   ```

3. **Test WebSocket connection:**
   ```bash
   curl -i -N -H "Connection: Upgrade" \
        -H "Upgrade: websocket" \
        https://yourdomain.com/radarr/signalr
   ```

---

### Issue: API Returns 404 Errors

**Symptoms:**
- External integrations fail
- API endpoints return 404
- `curl` requests to `/api/v3/` fail

**Solutions:**
1. **Include URL Base in API calls:**
   ```bash
   # Correct (with URL Base)
   curl https://yourdomain.com/radarr/api/v3/system/status
   
   # Incorrect (without URL Base)
   curl https://yourdomain.com/api/v3/system/status
   ```

2. **Verify API is enabled:**
   - Settings → General → Security
   - Check "Allow API access"
   - Note your API key

3. **Test API locally:**
   ```bash
   curl http://localhost:7878/radarr/api/v3/system/status
   ```

4. **Check API headers:**
   ```bash
   curl -v -H "X-Api-Key: YOUR_API_KEY" \
        https://yourdomain.com/radarr/api/v3/movie
   ```

---

### Issue: Slow Performance or Timeouts

**Symptoms:**
- Page loads slowly
- Timeout errors
- Long request times

**Solutions:**
1. **Increase proxy timeouts in Nginx:**
   ```nginx
   proxy_connect_timeout 120s;
   proxy_send_timeout 120s;
   proxy_read_timeout 120s;
   ```

2. **Enable Nginx buffering:**
   ```nginx
   proxy_buffering on;
   proxy_buffer_size 128k;
   proxy_buffers 4 256k;
   ```

3. **Check server resources:**
   ```bash
   top
   free -h
   df -h
   ```

4. **Monitor Nginx logs:**
   ```bash
   tail -f /var/log/nginx/radarr_access.log | grep -E 'HTTP/[0-9]+ [4-5][0-9]{2}'
   ```

---

### Issue: Radarr Won't Start After Configuration

**Symptoms:**
- Radarr crashes on startup
- Service fails to start
- No clear error messages

**Solutions:**
1. **Check Radarr logs:**
   ```bash
   tail -50 /config/logs/radarr.txt
   ```

2. **Verify configuration file syntax:**
   ```bash
   cat /config/config.xml
   ```

3. **Try starting manually:**
   ```bash
   cd /opt/Radarr && ./Radarr -nobrowser
   ```

4. **Check permissions:**
   ```bash
   ls -la /config
   sudo chown -R radarr:radarr /config
   ```

5. **Review recent changes:**
   - Did you modify config.xml?
   - Did Radarr crash before?
   - Check systemd journal:
     ```bash
     sudo journalctl -u radarr -n 50
     ```

---

## Getting Help

### Useful Log Locations:
- Radarr: `/config/logs/radarr.txt`
- Nginx Access: `/var/log/nginx/radarr_access.log`
- Nginx Error: `/var/log/nginx/radarr_error.log`
- Systemd: `sudo journalctl -u radarr`

### Useful Commands:
```bash
# Test configurations
sudo nginx -t

# Check listening ports
sudo netstat -tlnp
sudo ss -tlnp

# Monitor processes
ps aux | grep radarr
ps aux | grep nginx

# Restart services
sudo systemctl restart radarr
sudo systemctl restart nginx
```

### Resources:
- [Radarr GitHub Issues](https://github.com/Radarr/Radarr/issues)
- [Radarr Wiki](https://wiki.servarr.com/radarr/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Let's Encrypt Troubleshooting](https://certbot.eff.org/docs/troubleshooting/)
