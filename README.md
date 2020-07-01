# docker-nginx-certbot
Create and automatically renew website SSL certificates using the free [letsencrypt](https://letsencrypt.org/) certificate authority, and its client [*certbot*](https://certbot.eff.org/), built on top of the [nginx](https://www.nginx.com/) webserver.

This repository was originally forked from `@henridwyer`, many thanks to him for the good idea.  It has since been completely rewritten, and bears almost no resemblance to the original.  This repository is _much_ more opinionated about the structure of your webservers/containers, however it is easier to use as long as all of your webservers follow the given pattern.

# Usage

Create a config directory for your custom configs:

```bash
$ mkdir conf.d
```

And a `.conf` in that directory:
```nginx
server {
    listen              443 ssl;
    server_name         server.company.com;
    ssl_certificate     /etc/letsencrypt/live/server.company.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/server.company.com/privkey.pem;

    location / {
        ...
    }
}
```

Wrap this all up with a `docker-compose.yml` file:
```yml
version: '3'
services:
    frontend:
        restart: unless-stopped
        image: staticfloat/nginx-certbot
        ports:
            - 80:80/tcp
            - 443:443/tcp
        environment:
            CERTBOT_EMAIL: owner@company.com
        volumes:
          - ./conf.d:/etc/nginx/user.conf.d:ro
          - letsencrypt:/etc/letsencrypt
volumes:
    letsencrypt:
```

Launch that docker-compose file, and you're good to go; `certbot` will automatically request an SSL certificate for any `nginx` sites that look for SSL certificates in `/etc/letsencrypt/live`, and will automatically renew them over time.

Note: using a `server` block that listens on port 80 may cause issues with renewal. This container will already handle forwarding to port 443, so they are unnecessary.

## Templating

You may wish to template your configurations, e.g. passing in a hostname so as to be able to run multiple identical copies of this container; one per website.  The docker container will use [`envsubst`](https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html) to template all mounted user configs with a user-provided list of environment variables.  Example:

```nginx
# In conf.d/nginx_template.conf
server {
    listen              443 ssl;
    server_name         ${FQDN};
    ssl_certificate     /etc/letsencrypt/live/${FQDN}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/${FQDN}/privkey.pem;

    ...
}
```

```yml
version: '3'
services:
    frontend:
        restart: unless-stopped
        image: staticfloat/nginx-certbot
        ports:
            - 80:80/tcp
            - 443:443/tcp
        environment:
            CERTBOT_EMAIL: owner@company.com
            # variable names are space-separated
            ENVSUBST_VARS: FQDN
            FQDN: server.company.com
        volumes:
          - ./conf.d:/etc/nginx/user.conf.d:ro
          - letsencrypt:/etc/letsencrypt
volumes:
    letsencrypt:
```

# Changelog

### 1.1
- Upgraded to Python 3 installed within the environment, various quality of life improvements around initial setup and renewal.

### 1.0
- Many improvements thanks to contributors from across the globe.  Together, we have drastically reduced the amount of customization needed; configs can be mounted directly into a prebuilt image, and the configurations can even be templated.

### 0.8
- Ditch cron, it never liked me anway.  Just use `sleep` and a `while` loop instead.

### 0.7
- Complete rewrite, build this image on top of the `nginx` image, and run `cron`/`certbot` alongside `nginx` so that we can have nginx configs dynamically enabled as we get SSL certificates.

### 0.6
- Add `nginx_auto_enable.sh` script to `/etc/letsencrypt/` so that users can bring nginx up before SSL certs are actually available.

### 0.5
- Change the name to `docker-certbot-cron`, update documentation, strip out even more stuff I don't care about.

### 0.4
- Rip out a bunch of stuff because `@staticfloat` is a monster, and likes to do things his way

### 0.3
- Add support for webroot mode.
- Run certbot once with all domains.

### 0.2
- Upgraded to use certbot client
- Changed image to use alpine linux

### 0.1
- Initial release
