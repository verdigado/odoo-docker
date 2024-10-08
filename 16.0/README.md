# Build Odoo docker image from Debian package

The Dockerfile has been modified to use a local Odoo Debian package to install to the docker container.

## Steps to build docker image

1. Fetch the odoo debian package from https://odoo.com/download
   * for the enterprise version, you need to login to your Odoo account!
2. Rename the downloaded file to `odoo16.deb` and place it in this directory.
3. Create the docker image `my-odoo-16` with the following command:

```
docker build -t my-odoo-16 .
```

## Use the docker image

The following docker-compose file will create three containers:
* odoo16-web-live-1 --> the Odoo container itself
* odoo16-db-live-1 --> the database (postgresql 15)
* docker-odoo16-mailhog-1 --> mailhog instance
* docker-odoo16-adminer-1 --> adminer instance

```
version: '3.1'
services:
  web:
    container_name: odoo16-web-live-1
    image: my-odoo-16:latest
    depends_on:
      - db
    ports:
      - 8369:8069
    command: -- --dev=all
    environment:
      - OPENUPGRADE_TARGET_VERSION=15.0
    volumes:
      - odoo16-data-live:/var/lib/odoo
      - ./config:/etc/odoo
      - ./addons:/mnt/extra-addons
  db:
    container_name: odoo16-db-live-1
    image: postgres:15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_USER=odoo
    volumes:
      - odoo16-db-data-live:/var/lib/postgresql/data
      - ./import-db:/tmp/import-db
  mailpit:
    image: axllent/mailpit
    ports:
       - 1225:1025 # smtp server
       - 8325:8025 # web ui
  adminer:
    image: adminer
      #restart: always
    ports:
      - 8382:8080
volumes:
  odoo16-db-data-live:
  odoo16-data-live:
```

## Access the web interfaces

Ports for the web access can be configured in the docker-compose file.

### Odoo

#### Login

http://localhost:8369/web/login

#### Database-Manager

http://localhost:8369/web/database/manager

### Adminer (PostgreSQL)

http://localhost:8382/

* select database system "PostgreSQL"
* Server: db
* Username: odoo
* Password: odoo
* Database: empty

### Mailpit

http://localhost:8325/
