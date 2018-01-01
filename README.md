# Introduction
---
Docker container wrapping a Nginx with LDAP authentication and Letsencrypt SSL based on https://github.com/g17/nginx-ldap. 

# Usage
---
## Setup LDAP
For details on how to setup LDAP authentication, see https://github.com/g17/nginx-ldap

## Setup volumes
The Docker file assumes the use of volumes mapped to the following container paths in order to support persistence of certificates.
* volume-name:/etc/nginx/
* volume-name:/etc/letsencrypt/
* volume-name:/etc/ssl/

## Setup of Let's Encrypt
This setup uses the certbot webroot authentication method which needs to be performed manually before the container can use SSL certificates from Let's Encrypt. For more indepth instructions, see the tutorial at [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-debian-8)
1. Start a shell session inside the container (the container must be running)
```bash
$ docker exec -it <container id> /bin/bash
```
2. Next add the .well-known path to your nginx configuration
```bash
$ nano /etc/nginx/nginx.conf
```
```text
location ~ /.well-known {
                allow all;
        }
```
3. Check nginx for configuration errors
```bash
$ nginx -t
```

4. Register your domain and remember this container must be publically available on port 443 (SSL)
```bash
$ certbot certonly -a webroot --webroot-path=/var/www/html -d example.com
```
Your certificate and configuration will be stored in /etc/letsencrypt and therefore you will want to make sure this volume is persisted someplace safe external to the container. 

5. The container will attempt auto-renew your certificates every 60 days. After attempting to renew the container, the container will stop, and therefore you will want to include a parameter such as *restart: "unless-stopped"* in your container orchestrator. 