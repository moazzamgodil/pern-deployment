## Sample Nginx Configuration

```
server {
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/[filename|domain]/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/[filename|domain]/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    root /home/ubuntu/[appDir]/build;
    index index.html index.htm index.nginx-debian.html; # Add index list
    server_name [domain] [www.domain];

    ## Only requests to our Host are allowed
    if ($host !~ ^(domain|www.domain)$ ) {
        return 301 https://domain$request_uri;
    }

    ## redirect www to nowww
    if ($host = 'www.domain' ) {
        return 301 https://domain$request_uri;
    }

    location / {
        try_files $uri /index.html =503;
    }

    # error 503 redirect to errror503.html
    error_page 503 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location /api {
        proxy_pass http://localhost:PORT;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    if ($host = www.domain) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host = domain) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host !~ ^(domain|www.domain)$ ) {
        return 301 https://domain$request_uri;
    }

    listen 80;
    listen [::]:80;

    server_name [domain] [www.domain];
    return 503; # managed by Certbot

    # error 503 redirect to errror503.html
    error_page 503 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}

server {
    listen 80;
    server_name [IP::1];
    return 301 https://domain$request_uri;
}

```
