# Web app example for application

## Pre Requirements (Required)

### Create virtual machine space

```dotenv
# vm.env
REQUST_MEMORY=1024# 1 GB
REQUST_VCPUS=1 # 1 CPU

LIMIT_MEMORY=2048 # 2 GB
LIMIT_VCPUS=2 # 2 CPU
```

## Access container registry  (Required)

- Login to container registry artifactory

```bash
docker login <container-registry> -u <username> -p <password>
```
## Third party services

- PostgreSQL 11.2

## Setup application spec by docker-compose

#### Pre init container before start application (optional)

- Setup network app-network

```bash
docker create network app-network
```

- Setup init container for database (migrate, seed)

```bash
docker-compose -f docker-init-compose.yml up -d
```
```yaml
version: "3.8"
networks:
  default:
    external:
      name: app-network
services:
  migrate:
    container_name: app-migrate
    networks:
      - default
    image: registry.gitlab.com/omi4571821/app-example:0.1.3-alpha # image from container registry
    env_file:
      - app.env # setup configuration for database and application
    command: python manage.py migrate # migrate database schema
```

### Start application  (Required)

```bash
docker-compose up -d
```
- Spec for setup application by docker-compose

```yaml
version: "3.8"
networks:
  default:
    external:
      name: app-network
services:
  database: # database third party service
    container_name: database
    networks:
      - default
    image: postgres:13.1-alpine
    env_file:
      - database.env
    volumes:
      - ./database-data:/var/lib/postgresql/data/
  app-web:
    container_name: app-web
    networks:
      - default
    image: registry.gitlab.com/omi4571821/app-example:0.1.3-alpha # image from container registry
    command: # command for start application
      - python
      - manage.py
      - runserver
      - "0.0.0.0:8000"
    ports:
      - "8000:8000" # expose port for access application
    env_file:
      - app.env  # setup configuration for database and application
    depends_on:
      - database # wait for database ready
```
- setup configuration application

```dotenv
# app.env
DB_NAME=omi
DB_USER=postgres
DB_PASS=postgres
DB_HOST=database
DB_PORT=5432
APP_ENV=prod
```

- setup configuration database

```dotenv
# database.env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=omi
```

### Verify application (Required)

#### Check application

```bash
curl -m 3 http://localhost:8080 
```

- curl example and -m for timeout health check application listen on port 8080


## Register application to google sheet
- Fill in information in google sheet
  - App name (unique)
  - Username for container registry
  - Password for container registry
  - Container registry url
  - Git spect url
  - Release version
- Google Sheet url https://docs.google.com/spreadsheets/d/1NFQVRmeHG-q6gvH9dwVQJCPN4x6pdZfdRanV9bio1Ac/edit?usp=sharing









   

