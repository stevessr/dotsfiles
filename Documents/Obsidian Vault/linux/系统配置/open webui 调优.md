# NGINX 反代
```
# /etc/nginx/nginx.conf or /usr/local/openresty/nginx/conf/nginx.conf

  

#----------------------------------------------------------------------

# Global Settings

#----------------------------------------------------------------------

  

# User and group Nginx runs as (often nginx or www-data)

# user nginx nginx;

  

# Automatically set worker processes based on CPU cores

worker_processes auto;

  

# Automatically bind worker processes to specific CPU cores

worker_cpu_affinity auto;

  

# Path to the PID file

pid /www/server/nginx/nginx.pid; # Make sure this path is writable by the Nginx user

  

# Error log configuration

# Levels: debug, info, notice, warn, error, crit, alert, emerg

error_log logs/error.log notice;

  

#----------------------------------------------------------------------

# Events Module Settings

#----------------------------------------------------------------------

events {

    # Max connections per worker process

    worker_connections 1024;

  

    # Use epoll for high performance I/O multiplexing on Linux

    use epoll;

  

    # Accept multiple connections at once if supported by the OS

    # multi_accept on;

}

  

#----------------------------------------------------------------------

# HTTP Block Settings

#----------------------------------------------------------------------

http {

    #------------------------------------------------------------------

    # Basic HTTP Settings

    #------------------------------------------------------------------

    include       mime.types;          # Include MIME type mappings

    default_type  application/octet-stream; # Default MIME type for unknown extensions

  

    #------------------------------------------------------------------

    # Logging Settings

    #------------------------------------------------------------------

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '

                      '$status $body_bytes_sent "$http_referer" '

                      '"$http_user_agent" "$http_x_forwarded_for"';

  

    access_log logs/access.log main; # Path to the access log

  

    #------------------------------------------------------------------

    # Performance Tuning

    #------------------------------------------------------------------

    sendfile        on;   # Enable efficient file transfer using sendfile()

    # tcp_nopush      on;   # Send HTTP response headers in one packet (useful with sendfile on)

    # tcp_nodelay     on;   # Send data immediately without buffering (good for low latency)

  

    keepalive_timeout  65;  # Timeout for keep-alive connections

    # keepalive_requests 100; # Max requests per keep-alive connection

  

    # client_max_body_size 100m; # Max allowed size of client request body

  

    #------------------------------------------------------------------

    # Gzip Compression Settings

    #------------------------------------------------------------------

    gzip              on;             # Enable gzip compression

    gzip_disable      "msie6";        # Disable for IE6 (buggy)

    gzip_vary         on;             # Add "Vary: Accept-Encoding" header

    gzip_proxied      any;            # Compress responses from proxied servers

    gzip_comp_level   6;              # Compression level (1-9)

    gzip_buffers      16 8k;          # Number and size of compression buffers

    gzip_http_version 1.1;            # Minimum HTTP version to compress

    gzip_min_length   1000;           # Minimum response size to compress

    gzip_types        text/plain text/css application/json application/javascript application/xml text/xml image/svg+xml; # MIME types to compress

  

    #------------------------------------------------------------------

    # Proxy Cache Settings

    #------------------------------------------------------------------

    # Defines a cache storage area on disk

    # levels: Directory structure levels (e.g., 1:2 means /cache/c/c4/...)

    # keys_zone: Name and size of the shared memory zone for cache keys

    # max_size: Maximum size of the cache on disk

    # inactive: Files not accessed for this duration are removed

    proxy_cache_path  cache levels=1:2 keys_zone=cache_zone:64m max_size=1000m inactive=10m use_temp_path=off;

  

    # Defines the default cache key format

    proxy_cache_key   "$scheme$request_method$host$request_uri$is_args$args";

  

    #------------------------------------------------------------------

    # Server Block (Virtual Host)

    #------------------------------------------------------------------

    server {

        listen 80 default_server; # Listen on port 80 for IPv4

        # listen [::]:80 default_server; # Listen on port 80 for IPv6 (optional)

        server_name localhost your_domain.com; # Replace with your actual domain or IP

  

        # Default root directory for requests

        root /mnt/disk/development/frontend;

        # Default file to serve if directory is requested

        index index.html index.htm;

  

        #--------------------------------------------------------------

        # Location Blocks (Request Routing)

        #--------------------------------------------------------------

  

        # Serve static files from /op directory alias

        location /op {

            alias /mnt/disk/op/;      # Serve files directly from this path

            autoindex on;             # Optional: Enable directory listing if index file not found

            sendfile on;              # Use sendfile for these static assets

            tcp_nopush on;

            expires 30d;              # Set browser cache expiration

            add_header Cache-Control "public"; # Mark as publicly cacheable

        }

  

        # Serve static files from /admin directory alias (seems similar to root, maybe redundant?)

        # If /admin is part of your frontend SPA, the location block below might handle it better.

        # Consider if this specific alias is still needed. If not, remove it.

        location /admin {

            alias /mnt/disk/development/frontend/; # Alias to the same root? Check if this is correct.

            autoindex on;             # Optional: Enable directory listing

            sendfile on;

            tcp_nopush on;

            expires 30d;              # Set browser cache expiration

            add_header Cache-Control "public";

            # If /admin is part of the SPA, you might need try_files here too:

            # try_files $uri $uri/ /index.html;

        }

  

        # Proxy API requests under /api/v1 WITHOUT caching

        location /api/v1 {

            proxy_pass http://127.0.0.1:8088; # Proxy to backend service

  

            # Common Proxy Headers

            proxy_set_header Host $host;

            proxy_set_header X-Real-IP $remote_addr;

            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_set_header REMOTE-HOST $remote_addr; # Custom header? Ensure backend uses it.

  

            # Proxy Settings

            proxy_http_version 1.1;

            proxy_read_timeout 3600s; # Long timeout (1 hour), adjust if needed

            proxy_send_timeout 3600s; # Long timeout (1 hour), adjust if needed

  

            # Cache Control for this specific location (Bypass/No Cache)

            proxy_cache_bypass $http_upgrade; # Don't use cache for WebSocket upgrades

            proxy_no_cache $http_upgrade;     # Don't store cache for WebSocket upgrades

            # Explicitly disable caching for this path if needed beyond bypass/no_cache

            # proxy_cache off;

  

            # Add cache status header (will likely show BYPASS or MISS)

            add_header X-Cache $upstream_cache_status;

  

            # Security Headers (Consider placing globally if applicable to all proxied requests)

            add_header Strict-Transport-Security "max-age=31536000" always; # Use 'always' to ensure it's added

  

            # WebSocket Headers (Needed only if this endpoint handles WebSockets)

            # proxy_set_header Upgrade $http_upgrade;

            # proxy_set_header Connection "upgrade";

        }

  

        # Proxy API requests under /api WITH caching

        location /api {

            proxy_pass http://127.0.0.1:8088; # Proxy to backend service

  

            # Common Proxy Headers

            proxy_set_header Host $host;

            proxy_set_header X-Real-IP $remote_addr;

            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_set_header REMOTE-HOST $remote_addr;

  

            # Proxy Settings

            proxy_http_version 1.1;

            proxy_read_timeout 3600s; # Long timeout

            proxy_send_timeout 3600s; # Long timeout

  

            # Cache Settings for this location

            proxy_cache cache_zone;           # Use the defined cache zone

            proxy_cache_key $host$uri$is_args$args; # Use a specific cache key (or inherit global)

            proxy_cache_valid 200 304 1m;     # Cache 200/304 responses for 1 minute

            proxy_cache_valid 301 302 10m;    # Cache redirects for 10 minutes

            proxy_cache_valid any 1m;         # Cache other status codes (e.g., 404) for 1 minute (optional)

            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504; # Serve stale cache on errors

            proxy_cache_lock on;              # Prevent multiple requests for the same uncached item simultaneously

  

            # Cache Bypass

            proxy_cache_bypass $http_upgrade; # Bypass for WebSocket upgrades

            proxy_no_cache $http_upgrade;     # Don't store cache for WebSocket upgrades

  

            # Headers

            add_header X-Cache $upstream_cache_status; # Add cache status header

            add_header Strict-Transport-Security "max-age=31536000" always;

            proxy_ignore_headers Set-Cookie Cache-Control expires; # Ignore backend cache control headers

  

            # WebSocket Headers (Needed only if this endpoint handles WebSockets)

            # proxy_set_header Upgrade $http_upgrade;

            # proxy_set_header Connection "upgrade";

        }

  

        # Proxy WebSocket requests under /ws

        location /ws {

            proxy_pass http://127.0.0.1:8088; # Proxy to backend WebSocket service

  

            # Proxy Settings for WebSocket

            proxy_http_version 1.1;

            proxy_set_header Upgrade $http_upgrade; # Essential for WebSocket handshake

            proxy_set_header Connection "upgrade";   # Essential for WebSocket handshake

            proxy_set_header Host $host;

            proxy_set_header X-Real-IP $remote_addr;

            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_read_timeout 86400s; # Very long timeout for persistent connections (24 hours)

            proxy_send_timeout 86400s; # Very long timeout for persistent connections

  

            # Caching is generally NOT desired for WebSockets. Removed cache directives.

            # add_header X-Cache $upstream_cache_status; # This header is less relevant here

            add_header Strict-Transport-Security "max-age=31536000" always;

        }

  

        # Proxy requests under /ollama (Assuming potentially long-running requests, maybe SSE or similar)

        location /ollama {

            proxy_pass http://127.0.0.1:8088; # Proxy to Ollama backend service

  

            # Common Proxy Headers

            proxy_set_header Host $host;

            proxy_set_header X-Real-IP $remote_addr;

            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_set_header X-Forwarded-Proto $scheme;

  

            # Proxy Settings

            proxy_http_version 1.1;

            proxy_read_timeout 3600s; # Long timeout, adjust as needed for Ollama responses

            proxy_send_timeout 3600s; # Long timeout

            proxy_buffering off;      # Disable buffering for streaming responses (like SSE)

  

            # WebSocket Headers (Include if /ollama uses WebSockets)

            # proxy_set_header Upgrade $http_upgrade;

            # proxy_set_header Connection "upgrade";

  

            # Caching is likely NOT desired for dynamic AI responses. Removed cache directives.

            # add_header X-Cache $upstream_cache_status;

            add_header Strict-Transport-Security "max-age=31536000" always;

        }

  

        # Handle Single Page Application (SPA) routing for specific paths

        # This ensures that direct navigation to /c/some/path, /channels/etc, /workspace/view, /admin/panel

        # serves the main index.html, letting the frontend router take over.

        location ~ ^/(c|channels|workspace|admin)/ {

            # Serve static assets if they exist directly (e.g., /admin/logo.png)

            # Otherwise, fall back to serving /index.html for the SPA router

            try_files $uri $uri/ /index.html;

  

            # No proxying here unless specific API calls under these paths are intended

            # and not covered by /api or /api/v1. The original 'if' logic based on

            # Accept header was complex and potentially problematic.

            # If you need to proxy specific non-HTML requests under these paths,

            # add more specific location blocks *before* this one. Example:

            # location /admin/api/users { proxy_pass ...; }

        }

  

        # Catch-all for other requests - handles root SPA routing and static files

        # This block should usually come last or after more specific locations.

        location / {

            try_files $uri $uri/ /index.html; # Serve file, directory, or fallback to index.html for SPA

            expires -1; # Prevent caching of index.html itself (usually desired for SPAs)

            add_header Cache-Control "no-store"; # Stronger cache prevention

        }

  

        # Example Lua block (OpenResty specific)

        location = /hello { # Use exact match '=' for performance

            default_type text/html;

            content_by_lua_block {

                ngx.say("<h1>Hello, OpenResty!</h1>")

            }

        }

  

        # Optional: Deny access to hidden files (e.g., .htaccess, .git)

        location ~ /\. {

            deny all;

        }

  

        # Optional: Favicon and robots.txt handling

        location = /favicon.ico {

            log_not_found off;

            access_log off;

        }

        location = /robots.txt {

            log_not_found off;

            access_log off;

        }

    }

  

    #------------------------------------------------------------------

    # Include other configuration files (optional)

    #------------------------------------------------------------------

    # include /etc/nginx/conf.d/*.conf;

    # include /etc/nginx/sites-enabled/*;

}
```