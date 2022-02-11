# compose2garden sample application

## React application with a NodeJS backend and a MongoDB database

This sample app serves as an example of a migration from docker-compose to Garden.
It is based on the `react-express-mongodb` example from the [Awesome Compose](https://github.com/docker/awesome-compose) repository.

The docker-compose configuration and source code have been left untouched, with only the Garden configuration files added to the project.

Project structure:
```
.
├── backend
│   ├── backend.garden.yml
│   ├── Dockerfile
│   ...
├── docker-compose.yaml
├── frontend
    ...
│   ├── Dockerfile
│   ├── frontend.garden.yml
│   ...
│   └── project.garden.yml
├── mongo
│   └── mongo.garden.yml
└──README.md
```

[_docker-compose.yaml_](docker-compose.yaml)

```yaml
services:
  frontend:
    build:
      context: frontend
    ...
    ports:
      - 3000:3000
    ...
  server:
    container_name: server
    restart: always
    build:
      context: server
      args:
        NODE_PORT: 3000
    ports:
      - 3000:3000
    ...
    depends_on:
      - mongo
  mongo:
    container_name: mongo
    restart: always
    ...
```
The compose file defines an application with three services `frontend`, `backend` and `db`.
When deploying the application, docker-compose maps port 3000 of the frontend service container to port 3000 of the host as specified in the file.
Make sure port 3000 on the host is not already being in use.

Garden defines the same services but deploys the application on a local Kubernetes cluster, make sure it's running before deploying.
The project is configured to create two ingresses from which the services are available:

- `frontend` service: [http://compose2garden.local.app.garden](http://compose2garden.local.app.garden).
- `backend` service: [http://backend.compose2garden.local.app.garden](http://backend.compose2garden.local.app.garden).

## Deploy the project

### Deploy with garden

> Note: you need to have a running Kubernetes cluster on your local machine.

```sh
$ garden deploy
Deploy 🚀 

Using environment default.default

✔ providers                 → Getting status... → Cached
   ℹ Run with --force-refresh to force a refresh of provider statuses.
✔ mongo                     → Getting build status for v-4e4b9e4d48... → Already built
✔ backend                   → Getting build status for v-64f4cdb8df... → Already built
✔ mongo                     → Deploying version v-5e8e8da229... → Done (took 3.4 sec)
   ℹ mongo                     → Resources ready
✔ frontend                  → Getting build status for v-f6dbf8b6ea... → Already built
✔ backend                   → Deploying version v-f8ca19f295... → Done (took 3.1 sec)
   ℹ backend                   → Resources ready
   → Ingress: http://backend.compose2garden.local.app.garden
✔ frontend                  → Deploying version v-ad031cb91f... → Done (took 25.8 sec)
   ℹ frontend                  → Resources ready
   → Ingress: http://compose2garden.local.app.garden

Done! ✔️ 
```

#### Start Garden in dev-mode

```sh
$ garden dev 
Good morning! Let's get your environment wired up...

Using environment default.default

✔ providers                 → Getting status... → Cached
   ℹ Run with --force-refresh to force a refresh of provider statuses.
✔ mongo                     → Getting build status for v-4e4b9e4d48... → Already built
✔ backend                   → Getting build status for v-64f4cdb8df... → Already built
✔ frontend                  → Getting build status for v-f6dbf8b6ea... → Already built
ℹ backend                   → Syncing ./ to /usr/src/app in Deployment/backend (one-way-replica)
✔ backend                   → Connected to sync target /usr/src/app in Deployment/backend
✔ backend                   → Completed initial sync from ./ to /usr/src/app in Deployment/backend
✔ frontend                  → Connected to sync target /usr/src/app/src in Deployment/frontend
✔ frontend                  → Completed initial sync from ./src to /usr/src/app/src in Deployment/frontend
✔ mongo                     → Deploying version v-5e8e8da229... → Already deployed
   → Forward: localhost:27017 → mongo:27017 (db)
ℹ frontend                  → Syncing ./src to /usr/src/app/src in Deployment/frontend (one-way-replica)
✔ backend                   → Deploying version v-f8ca19f295... → Already deployed
   → Ingress: http://backend.compose2garden.local.app.garden
   → Forward: http://localhost:3000 → backend:3000 (http)
✔ frontend                  → Deploying version v-ad031cb91f... → Already deployed
   → Ingress: http://compose2garden.local.app.garden
   → Forward: http://localhost:60523 → frontend:3000 (http)

🌻  Garden dashboard running at http://localhost:9777

🕑  Waiting for code changes...
```

### Deploy with docker-compose

```sh
$ docker-compose up -d
Creating network "react-express-mongodb_default" with the default driver
Building frontend
Step 1/9 : FROM node:13.13.0-stretch-slim
 ---> aa6432763c11
...
Successfully tagged react-express-mongodb_app:latest
WARNING: Image for service app was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating frontend        ... done
Creating mongo           ... done
Creating app             ... done
```

