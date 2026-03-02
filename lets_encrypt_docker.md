# Let's Encrypt com Docker + Nginx Reverse Proxy (Pronto para Produção)

## 🎯 Objetivo

Arquitetura segura e profissional:

Internet → Nginx (Docker) → Backend (Spring Boot / Quarkus) → PostgreSQL

SSL gerenciado automaticamente via Let's Encrypt.

------------------------------------------------------------------------

# 1️⃣ Pré-requisitos

-   VPS Ubuntu 22.04
-   Docker instalado
-   Docker Compose instalado
-   Domínio apontando para IP da VPS
-   Portas 80 e 443 liberadas

------------------------------------------------------------------------

# 2️⃣ Instalar Docker

``` bash
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo systemctl start docker
```

Verificar:

``` bash
docker --version
docker-compose --version
```

------------------------------------------------------------------------

# 3️⃣ Estrutura de Diretórios

``` bash
mkdir -p /opt/app
cd /opt/app
mkdir nginx certbot
```

------------------------------------------------------------------------

# 4️⃣ docker-compose.yml

Crie o arquivo:

``` yaml
version: '3.8'

services:

  backend:
    image: sua-imagem-backend:latest
    container_name: backend
    restart: always
    expose:
      - "8080"

  nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot/www:/var/www/certbot
      - ./certbot/conf:/etc/letsencrypt
    depends_on:
      - backend

  certbot:
    image: certbot/certbot
    volumes:
      - ./certbot/www:/var/www/certbot
      - ./certbot/conf:/etc/letsencrypt
```

------------------------------------------------------------------------

# 5️⃣ Configuração Nginx

Crie:

/opt/app/nginx/conf.d/app.conf

``` nginx
server {
    listen 80;
    server_name seu-dominio.com www.seu-dominio.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

------------------------------------------------------------------------

# 6️⃣ Subir ambiente

``` bash
docker-compose up -d
```

------------------------------------------------------------------------

# 7️⃣ Gerar Certificado Inicial

``` bash
docker-compose run --rm certbot certonly   --webroot   --webroot-path=/var/www/certbot   seu-dominio.com -d www.seu-dominio.com
```

Após gerar, atualize o nginx para HTTPS:

``` nginx
server {
    listen 443 ssl;
    server_name seu-dominio.com www.seu-dominio.com;

    ssl_certificate /etc/letsencrypt/live/seu-dominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/seu-dominio.com/privkey.pem;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 80;
    server_name seu-dominio.com www.seu-dominio.com;
    return 301 https://$host$request_uri;
}
```

Reinicie:

``` bash
docker-compose restart nginx
```

------------------------------------------------------------------------

# 8️⃣ Renovação Automática

Crie um cron:

``` bash
crontab -e
```

Adicione:

``` bash
0 3 * * * docker-compose -f /opt/app/docker-compose.yml run --rm certbot renew && docker-compose -f /opt/app/docker-compose.yml restart nginx
```

------------------------------------------------------------------------

# 🔐 Boas Práticas Produção

-   Nunca expor backend diretamente
-   Usar variáveis de ambiente
-   Usar network interna Docker
-   Configurar firewall (UFW)
-   Backup da pasta certbot/conf
-   Monitoramento (Prometheus + Grafana)

------------------------------------------------------------------------

# 🚀 Arquitetura Final

Cliente HTTPS → Nginx Docker (443) → Backend Docker (8080 interno)

Seguro, escalável e pronto para produção.
