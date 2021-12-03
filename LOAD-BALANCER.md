# Load Balancer

To set up a load-balanced configuration, we encourage using [NGINX](https://www.nginx.com/) as  webserver or reverse proxy. NGINX allows you to easily build named groups of nodes and forward requests to them.

Examples mentioned in this documentation were tested with Debian Buster and nginx version: nginx/1.14.2


## Instalation

For most Debian setups, installation takes these executions:

```sh
sudo apt-get update
sudo apt-get install nginx
```
Check the official Nginx documentation for further reference.

## Reverse Proxy

To start using NGINX Plus or NGINX Open Source to load balance HTTP traffic to a group of servers, first, you need to define the group with the upstream directive. The directive is placed in the HTTP context.

Servers in the group are configured using the [server](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) directive. To pass requests to a server group, the name of the group is specified in the proxy_pass directive.

The following example shows how to proxy HTTP requests to the backend server group. The group consists of three servers, the three of them running instances of the application.  Because no load‑balancing algorithm is specified in the upstream block, NGINX uses the default algorithm, Round Robin:


```
# Server pool
upstream osskb_servers {
  server server1.eu:4443;  
  server server2.eu:4443;  
  server server3.eu:4443; 
}


server {
  listen 443;
  server_name osskb.org;

  ssl on;
  ssl_certificate /etc/letsencrypt/live/mysite.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/mysite.com/privkey.pem;

  access_log /var/log/nginx/mysite_com_access.log rt;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass http://osskb_servers;
    proxy_read_timeout 90;

  }
}

```

## HTTPS Certificates

To get our platform on HTTPS, we rely on multiple Certificate Providers, including [Let's Encrypt](https://letsencrypt.org/), a nonprofit Certificate Authority.

[Certbot](https://certbot.eff.org/) is a great tool to issue and automatically renew certificates. Since Certbot works with multiple web servers on multiple OSs, we recommend using Certbot's official [Installation instructions](https://certbot.eff.org/instructions) for better support.

First, make sure all traffic to port 80 gets redirected to HTTPS and ensure ACME Challenges are supported.

```
# /etc/nginx/sites-enabled/http.conf

server_tokens off;

server {
    listen 80 default_server;
    server_name _;

    location /.well-known/acme-challenge {
        root /var/www/letsencrypt;
        try_files $uri $uri/ =404;
    }

    location / {
        rewrite ^ https://$host$request_uri?;
    }
}
```

After reloading Nginx, certificates can be generated automatically with Certbot. 

After installing Certbot, use it as follows: 

``` certbot --nginx -d mysite.com```

This command will generate the corresponding certificates at this location and update you nginx configuration to use them.

 * /etc/letsencrypt/live/mysite.com/fullchain.pem
 * /etc/letsencrypt/live/mysite.com/privkey.pem

 ::: tip
Let’s Encrypt provides rate limits to ensure fair usage by as many people as possible. The main limit is Certificates per Registered Domain (50 per week). Other limits apply, please make sure to review them at [https://letsencrypt.org/docs/rate-limits/](https://letsencrypt.org/docs/rate-limits/)
:::