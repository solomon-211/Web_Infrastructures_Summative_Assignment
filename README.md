# HealthHub Medical Finder

A web application for finding medical facilities based on the search location in Rwanda in regard to the user's preferences.

## Repository

**GitHub Repository:** https://github.com/solomon-211/Web_Infrastructures_Summative_Assignment.git

## Features

- **Facility Search**: Find hospitals, clinics, pharmacies, and dental offices
- **Location-Based Results**: Search by city or district with distance filtering
- **Interactive Maps**: Google Maps integration for facility locations
- **User Authentication**: Secure login and registration system
- **Personal Dashboard**: User statistics and activity tracking
- **Favorites System**: Save and manage preferred facilities
- **Admin Panel**: User management and system analytics

## Project Structure
```
HealthHub/
├── Index.html          # Main application
└── README.md          # Documentation
```

## Code Organization (within index.html)

```
index.html
│
├── <head>
│   ├── Meta tags (SEO, viewport)
│   ├── Title
│   ├── Font Awesome CDN
│   └── <style>
│       ├── Reset & Base Styles
│       ├── Header & Search Card
│       ├── Filters & Buttons
│       ├── Facility Cards
│       ├── Pagination
│       ├── Alerts & Loading States
│       └── Responsive Media Queries
│
└── <body>
    ├── Container Structure
    │   ├── Header Section
    │   ├── Search & Filter Card
    │   ├── Results Container
    │   └── Pagination
    │
    ├── Footer (Attribution)
    │
    └── <script>
        ├── State Management
        ├── Configuration Constants
        ├── Event Listeners
        ├── API Integration Functions
        ├── Data Processing & Filtering
        ├── UI Rendering Functions
        ├── Cache Management
        ├── Error Handling
        └── Utility Functions
```

## Technology Stack

- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **APIs**: Nominatim (OpenStreetMap), Google Maps
- **Storage**: Browser localStorage
- **UI**: Font Awesome icons, responsive CSS Grid/Flexbox

## Security Features

- Client-side password hashing
- Input validation and sanitization
- XSS attack prevention
- Secure session management
- Data encryption for sensitive information

## APIs Used
1. Nominatim API (OpenStreetMap)
- Purpose: Geocoding services and medical facility search

### Endpoints Used:
/search: Geocode locations and search for facilities

Parameters: q (query), format (json), limit, countrycodes (rw)
Rate Limit: 1 request per second
Documentation: https://nominatim.org/release-docs/latest/api/Search/



### Features Utilized:

- Location coordinate retrieval
- Address details and formatting
- Facility metadata (phone, website, type)
- Distance calculations using Haversine formula

### Rate Limiting Strategy:

- Implemented 10-minute caching system
- Retry logic with exponential backoff
- Request timeout handling (10 seconds)

2. Google Maps API
- Interactive map visualization
- Direct linking to Google Maps with coordinates
- Opens facility location in new tab
- No API key required for basic links

## Security Considerations
API Key Management

- No API keys required: Using free, public APIs
- No sensitive data storage: Only user preferences stored locally
- HTTPS ready: Application works with secure connections



## Deployment instruction
To deploy the HealthHub Medical Finder application, follow these steps:

Step 1: Connect to Servers
- Use SSH to connect to your web server or hosting service.
Step 2: Prepare Web Directory
### On both Web01 and Web02
- cd /var/www/html

### Create application directory
- sudo mkdir -p healthhub
- sudo chown $USER:$USER healthhub
- cd healthhub

Step 3: Upload Application Files
## First option
### On your local machine

- Navigate to the project directory containing Index.html and README.md.

### Use SCP to transfer files to the server

- scp Index.html README.md user@web01:/var/www/html/healthhub/
- scp Index.html README.md user@web02:/var/www/html/healthhub/


Step 5: Configure Web Server
For Apache:
bash# Create virtual host configuration
sudo nano /etc/apache2/sites-available/healthhub.conf
### Add the following configuration:
apache<VirtualHost *:80>
    ServerName healthhub.solomon-leek.tech
    DocumentRoot /var/www/html/healthhub
    
    <Directory /var/www/html/healthhub>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/healthhub-error.log
    CustomLog ${APACHE_LOG_DIR}/healthhub-access.log combined
</VirtualHost>

### Enable the site:
- bashsudo a2ensite healthhub.conf
- sudo systemctl reload apache2

## For Nginx:
- bash# Create server block

sudo nano /etc/nginx/sites-available/healthhub
### Add the following configuration:
nginxserver {
    listen 80;
    server_name healthhub.solomon-leek.tech;
    root /var/www/html/healthhub;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    access_log /var/log/nginx/healthhub-access.log;
    error_log /var/log/nginx/healthhub-error.log;
}
### Enable the site:
- sudo ln -s /etc/nginx/sites-available/healthhub /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
Step 6: Verify Deployment
bash# Test Web01
curl http://web01.solomon-leek.tech/healthhub/

# Test Web02
curl http://web02.solomon-leek.tech/healthhub/

## Load Balancer Configuration
- The load balancer (Lb01) distributes incoming traffic between Web01 and Web02, ensuring high availability and optimal performance.

## Configuration Steps
- Step 1: Connect to Load Balancer

ssh username@lb01.solomon-leek.tech

- Step 2: Install HAProxy (if not already installed)
sudo apt update
sudo apt install haproxy -y

- Step 3: Configure HAProxy
sudo nano /etc/haproxy/haproxy.cfg

### Add the following configuration:
haproxyglobal
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

# Frontend configuration
frontend healthhub_frontend
    bind *:80
    default_backend healthhub_backend
    
    # Optional: Add custom headers
    http-request set-header X-Forwarded-For %[src]
    http-request set-header X-Forwarded-Proto http

# Backend configuration with health checks
backend healthhub_backend
    balance roundrobin
    option httpchk GET /healthhub/
    http-check expect status 200
    
    # Server definitions
    server web01 web01.solomon-leek.tech:80 check inter 2000 rise 2 fall 3
    server web02 web02.solomon-leek.tech:80 check inter 2000 rise 2 fall 3

# Statistics page (optional but recommended)
listen stats
    bind *:8080
    stats enable
    stats uri /stats
    stats refresh 30s
    stats auth admin:your_secure_password
Step 4: Validate and Restart HAProxy
bash# Validate configuration
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# Restart HAProxy
sudo systemctl restart haproxy

# Enable HAProxy to start on boot
sudo systemctl enable haproxy

# Check status
sudo systemctl status haproxy