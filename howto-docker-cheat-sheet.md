# setup keycloak with postgres DB on docker

- ## the plain DOCKER way

### create a keycloak network

  `$ docker network create keycloak-network`

  if already exists, this statement prunes the existing network: `$ docker network rm keycloak-network`

### create a postgres container

  `$ docker run -d --name postgressql --net keycloak-network -e POSTGRES_DB=keycloak -e POSTGRES_USER=keycloak -e POSTGRES_PASSWORD=password postgres`

### create a keycloak container

  `$ docker run -d --name keycloak --net keycloak-network -e KEYCLOAK_DATABASE_HOST=postgressql -e KEYCLOAK_DATABASE_PORT=5432 -e KEYCLOAK_DATABASE_NAME=keycloak -e KEYCLOAK_DATABASE_USER=keycloak -e KEYCLOAK_DATABASE_PASSWORD=password -e KEYCLOAK_ADMIN_USER=admin -e KEYCLOAK_ADMIN_PASSWORD=Entwicklung123 --publish target=8080,published=127.0.0.1:8081,protocol=tcp bitnami/keycloak:19.0.3`

- ## the DOCKER-COMPOSE way

  with the following **docker-compose.yaml** file, all above steps can be automated and executed with one single command.

    ``` YAML
    version: "3.9"
    services:
    postgressql:
        image: postgres
        restart: always
        networks:
          - keycloak-network
        environment:
          - POSTGRES_DB=keycloak 
          - POSTGRES_USER=keycloak 
          - POSTGRES_PASSWORD=password
        ports:
          - "5432:5432"
        volumes: 
          - postgressql:/var/lib/postgresql/data
    keycloak:
        image: bitnami/keycloak:19.0.3
        restart: always
        depends_on:
          - postgressql
        networks:
          - keycloak-network
        environment:
          - KEYCLOAK_DATABASE_HOST=postgressql 
          - KEYCLOAK_DATABASE_PORT=5432 
          - KEYCLOAK_DATABASE_NAME=keycloak 
          - KEYCLOAK_DATABASE_USER=keycloak 
          - KEYCLOAK_DATABASE_PASSWORD=password 
          - KEYCLOAK_ADMIN_USER=admin 
          - KEYCLOAK_ADMIN_PASSWORD=Entwicklung123  
        ports: 
          - "8090:8080"
    networks:
      keycloak-network:
    volumes:
      postgressql:
        driver: local
    ```

### the commands to run and stop keycloak and a postgres instance *(and db adminer)*
  
- switch to directory with your docker-compose.yaml file `$ cd <your-dir>`
- to install and start in daemon mode `$ docker compose up -d`
- to stop the running containers `$ docker compose stop` or faster but not safe `$ docker compose kill`
- to start the containers `$ docker compose start`
- to remove from docker `$ docker compose down --volumes`
- to remove from docker but leave the volume with the database data `$ docker compose down`
- instead of the above **docker compose** CLI command you can also use the **docker-compose** CLI with. but maybe in the future the docker-compose will be removed.

### to log in into db adminer do the following

- open the following in a browser: `http://localhost:8092`
- enter the following credentials into the upcoming login mask
  - Datenbank System: **PostgreSQL**
  - Server: **postgressql**
  - Benutzer: **keycloak**
  - Passwort: **password**
  - Datenbank: **keycloak**
