# Commands I Used

All of these ran on my Kali Linux VM against my own local lab (127.0.0.1).

## 1. Start the ColdBox / BoxLang app (port 8080)
```bash
sudo docker run -d --name coldbox-target -p 8080:8080 ortussolutions/commandbox bash -c \
"mkdir -p /mycoldbox && cd /mycoldbox && box install coldbox-cli && \
box coldbox create app name=ColdBoxTarget && box server start host=0.0.0.0 port=8080 openbrowser=false --console"
```

## 2. Recon and scanning
```bash
nmap -sV -p 8080 127.0.0.1
whatweb http://127.0.0.1:8080
nikto -h http://127.0.0.1:8080
```

## 3. Manual XSS / SQLi test (in Firefox)
```
http://127.0.0.1:8080/?search=<script>alert('XSS')</script>&id=1' OR '1'='1
```

## 4. Login test (sent in Burp Suite Repeater)
```http
POST /main/login HTTP/1.1
Host: 127.0.0.1:8080
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

username=Noor
```

## 5. Start the OWASP ModSecurity CRS WAF (port 8082)
```bash
sudo docker run -d --name coldbox-waf --add-host=host.docker.internal:host-gateway \
-p 8082:8080 -e PORT=8080 -e BACKEND=http://host.docker.internal:8080 \
-e PARANOIA=1 -e ANOMALY_INBOUND=5 owasp/modsecurity-crs:apache
```

## 6. Retests after hardening
```bash
# Reinit without the password
curl -i "http://127.0.0.1:8080/?fwreinit=1"

# Normal request through the WAF (should be 200)
curl -I "http://127.0.0.1:8082/?search=hello"

# XSS through the WAF (should be 403)
curl -i "http://127.0.0.1:8082/?search=%3Cscript%3Ealert(1)%3C/script%3E"

# See which WAF rules fired
sudo docker logs coldbox-waf 2>&1 | grep -E "941|403|id \"" | tail -n 8
```
