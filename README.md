# NoDock
Docker Compose for Node projects with Node, MySQL, NGINX and Certbot images

**WARNING: THIS PROJECT IS STILL IN EARLY DEVELOPMENT, DO NOT USE IN PRODUCTION**

## Requirements
* [Docker Engine 1.12+](https://docs.docker.com/engine/installation/)
* [Docker Compose 1.8+](https://docs.docker.com/compose/install/)

## Usage

#### Install in your project

As a submodule:
```
git submodule add https://github.com/Osedea/nodock.git
```

#### Build and Run the containers
```
cd nodock
# Simple app
docker-compose up -d node mysql nginx
# or
# All containers
docker-compose up -d
```

## Allow HTTPS

By default HTTPS is disabled. To enable it, you may use the following settings

```
# docker-compose.override.yml

version: '2'

services:
    nginx:
        build:
            args:
                web_ssl: "true"
```
Add your certificate to `nginx/certs/cacert.pem` and the private key to `nginx/certs/privkey.pem`.

#### Generate and use a self-signed cert

`self_signed: "true"` will generate the necessary files, do note that `self_signed: "true"` as no effect if `web_ssl: "false"`

```
# docker-compose.override.yml

version: '2'

services:
    nginx:
        build:
            args:
                web_ssl: "true"
                self_signed: "true"
```

#### Use certbot (Let's Encrypt) to generate the cert

`CN` must be a publicly accessible address and `EMAIL` should be the server admin contact email.

```
version: '2'

services:
    nginx:
        build:
            args:
                web_ssl: "true"
    certbot:
        environment:
            CN: "example.com"
            EMAIL: "fake@gmail.com"
```
Don't forget to bring up the container if you plan on using certbot (`docker-compose up -d certbot`).

## Running multiple node containers

To add more node containers, simply add the following to your `docker-compose.override.yml` or environment specific docker-compose file.

```
# docker-compose.override.yml

version: '2'

services:
    node2: # name of new container
        extends: node # extends the settings from the "node" container
        entrypoint: run-nodock "node alternate.js" # the entrypoint for the "node2" container
    nginx:
        ports:
            - "10000:10000" # the port(s) to forward for the "node2" container
        links:
            - node2 # link "nginx" to "node2"
```

You'll also need to add a server block for "node2".
```
# nginx/sites/node2.conf

server {
    listen 10000 default_server;

    location / {
        proxy_pass http://node2:8000;
    }
}
```

## Customization

To customize the NoDock installation, either add a `docker-compose.override.yml` in the NoDock directory or store environment specific configurations.

```
docker-compose -f nodock/docker-compose.yml -f docker-compose.dev.yml up -d
```

#### Change the node entrypoint

Use `main.js` instead of `index.js`
```
# docker-compose.override.yml

version: '2'

services:
    node:
        entrypoint: run-nodock "node main.js"
```

#### Change the MySQL database/user/password
```
# docker-compose.override.yml

version: '2'

services:
    mysql:
        build:
            args:
                mysql_database: default_database
                mysql_user: default_user
                mysql_password: secret
```

#### Change the NGINX reverse proxy port

Use port `8080` instead of `8000` to bind your Node server
```
# docker-compose.override.yml

version: '2'

services:
    nginx:
        build:
            args:
                web_reverse_proxy_port: "8080"
```

#### Change the NODE_ENV variable

The default `NODE_ENV` value is `production`, you can change it to development by doing the following
```
# docker-compose.override.yml

version: '2'

services:
    node:
        environment:
            NODE_ENV: development
```

#### Use a specific Node version

The default node version is `latest`, this is **NOT** advisable for production
```
# docker-compose.override.yml

version: '2'

services:
    node:
        build:
            args:
                node_version: 4.6.0
```
