## How to Compose it

#### Setup Compose .env & Up
- Copy `.env.example` to `.env` and modify (or default)
- Run `docker-compose up -d`

#### Setup Akaunting .env
- Modify `akaunting/.env` (or default)
- Copy `akaunting/.env` to container by running `docker cp akaunting/.env akaunting-docker_akaunting_php_1:/var/www/akaunting/`
- Run `docker exec -it akaunting-docker_akaunting_php_1 chmod 755 /var/www/akaunting/.env`
- Access Akaunting: [http://127.0.0.1:8001](http://127.0.0.1:8001)

Above steps should produce these 3 containers (`docker ps`) :
- akaunting-docker_akaunting_php_1
- akaunting-docker_akaunting_nginx_1
- akaunting-docker_akaunting_db_1

TODO :
- [ ] ADD `akaunting/.env` directly when building the image


## Setup for HTTPS or Reverse Proxy

Run the following commands to use pre-configured trusted proxy configuration :

- Publish TrustedProxy vendor config

```
docker exec -it akaunting-docker_akaunting_php_1 \
    php /var/www/akaunting/artisan vendor:publish \
    --provider="Fideloper\Proxy\TrustedProxyServiceProvider"
```

- Copy `trustedproxy.php` from host to the `php` container

```
docker cp ./akaunting/config/trustedproxy.php akaunting-docker_akaunting_php_1:/var/www/akaunting/config/
```

References: [https://akaunting.com/docs/developer-manual/reverse-proxy](https://akaunting.com/docs/developer-manual/reverse-proxy)

### Nginx

Use the following configuration for Nginx:
```
server {
    server_name akaunting.localhost;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8001;
    }
}
```
