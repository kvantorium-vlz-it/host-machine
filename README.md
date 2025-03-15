# Host infrastructure

## apps
- traefik (main reverse-proxy / handles docker images with labels)
- nginx (static sites)

## Adding new site

### Docker

Add labels for image

```docker-compose.yml
labels:
    # specify visibility for traefik
    - "traefik.enable=true"

    # specify domain for image
    - "traefik.http.routers.cms.rule=Host(`<domain>`)"

    # specify entrypoint:
    # web - for http connection
    # websecure - for https connection
    - "traefik.http.routers.cms.entrypoints=<entrypoint>"

    # specify certification resolver (lestencrypt)
    - "traefik.http.routers.cms.tls.certresolver=<certresolver>"
```

### nginx (static site)

1. Add configuration for site: `nginx/conf.d/<domain.conf>`

```conf
server {
    listen 80;
    server_name static1.lamparty.ru;

    location / {
        root /var/www/static1.lamparty.ru;
        try_files $uri $uri/ =404;
    }
}
```

2. Add static site folder: `nginx/www/<domain>` with `index.html` file

3. Add router for site in `traefik/config/dynamic/traefik.yml`:

```yml
http:
  routers:
    # ...
    <project-name>:
      rule: "Host(`<domain>`)"
      entryPoints: ["web", "websecure"] # same as docker entrypoints label
      service: nginx

  services:
    # ...
    nginx:
      loadBalancer:
        servers:
          - url: "http://nginx:80"
```

### node/python/etc as web service

1. Check ip address using `ifconfig` and find interface inet starts with 192.168.x.x

2. Setup traefik http router and service:

```yml
http:
  routers:
    # ...
    <project>:
      rule: "Host(`<domain>`)"
      entryPoints: ["web", "websecure"]
      service: <project-service>
      tls:
        certResolver: <certResolver>

  services:
    <project-service>:
      loadBalancer:
        servers:
          - url: "http://<address>:<port>"
```

# TODO
- nginx logs
- monitoring
