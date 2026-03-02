# Instalação do Let's Encrypt em uma VPS (Ubuntu/Debian)

## 1. Pré-requisitos

-   VPS Ubuntu 22.04 ou Debian
-   Domínio apontando para o IP da VPS
-   Nginx ou Apache instalado

### Verificar DNS

``` bash
ping seu-dominio.com
```

------------------------------------------------------------------------

## 2. Atualizar servidor

``` bash
sudo apt update && sudo apt upgrade -y
```

------------------------------------------------------------------------

## 3. Instalar Certbot

### Para Nginx

``` bash
sudo apt install certbot python3-certbot-nginx -y
```

### Para Apache

``` bash
sudo apt install certbot python3-certbot-apache -y
```

------------------------------------------------------------------------

## 4. Gerar certificado

### Nginx

``` bash
sudo certbot --nginx
```

### Apache

``` bash
sudo certbot --apache
```

Durante o processo: 1. Informar e-mail 2. Aceitar os termos 3.
Selecionar domínio 4. Escolher redirecionamento HTTP → HTTPS

------------------------------------------------------------------------

## 5. Testar renovação automática

``` bash
sudo certbot renew --dry-run
```

------------------------------------------------------------------------

## 6. Verificar agendamento automático

``` bash
sudo systemctl list-timers | grep certbot
```

------------------------------------------------------------------------

## 7. Liberar portas (se usar UFW)

``` bash
sudo ufw allow 443
sudo ufw allow 80
sudo ufw enable
```

------------------------------------------------------------------------

## 8. Localização dos certificados

    /etc/letsencrypt/live/seu-dominio.com/

Arquivos principais: - fullchain.pem - privkey.pem

------------------------------------------------------------------------

## 9. Arquitetura recomendada (Backend Java)

Fluxo ideal:

Internet → Nginx (HTTPS 443) → Spring Boot / Quarkus (HTTP 8080)

Recomendado usar Nginx como reverse proxy para maior segurança e padrão
de produção.
