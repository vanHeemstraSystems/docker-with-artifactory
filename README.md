# Docker with Artifactory

See https://stackoverfow.com/questions/66658180/how-should-i-use-artifactory-in-dockerfile-for-npm-install

Use [Composerize](https://www.composerize.com) to convert a Docker command to a Dockerfile.

For Docker to be able to work with NPM packages that are hosted through Artifactory at a corporate level (i.e. within a company like "some-company") follow these instructions:

## 1. Write a sampel.env file and a copy as .env

Create a `sample.env` file within the same directory where `Dockerfile` and `compose.yaml` reside.

```
APP_PORT=8080
ARTIFACTORY_NPM_REGISTRY="https://some-department.some-company.com/artifactory/api/npm/mirror-npmjs/"
ARTIFACTORY_TOKEN="PLACE YOUR ARTIFACTORY TOKEN HERE, GENERATED IN ARTIFACTORY"
```
sample.env

See Artifactory documentation how to to generate a Token.

Create a copy of `sample.env` and name it `.env`.

Replace the "PLACE YOUR ARTIFACTORY TOKEN HERE, GENERATED IN ARTIFACTORY" with the real Artifactory token.

```
APP_PORT=8080
ARTIFACTORY_NPM_REGISTRY="https://some-department.some-company.com/artifactory/api/npm/mirror-npmjs/"
ARTIFACTORY_TOKEN="cmVmdGt#################################################################"
```
.env

## 2. Write a .npmrc file

Create a `.npmrc` file within the same directory where `Dockerfile` and `compose.yaml` reside.

```
@someregistry:registry=https://some-department.some-company.com/artifactory/api/npm/mirror-npmjs/
//some-department.some-company.com/artifactory/api/npm/mirror-npmjs/:always-auth=true
//some-department.some-company.com/artifactory/api/npm/mirror-npmjs/:_authToken=${ARTIFACTORY_TOKEN}
```
.npmrc

As you can see, `.npmrc` makes use of the environment variable `ARTIFACTORY_TOKEN` set in `.env`.

## 3. Write a compose.yaml file

Create a `compose.yaml` file within the same directory where `Dockerfile` resides.

```
name: some-name
services:
  lite:
    ports:
      - "${APP_PORT}:8080"
    user: $(id -u):$(id -g)
    volumes:
      - ./:/usr/local/structurizr
    restart: always
    logging:
      options:
        max-size: 1g
    args:
      ARTIFACTORY_NPM_REGISTRY: ${ARTIFACTORY_NPM_REGISTRY}
      ARTIFACTORY_TOKEN: ${ARTIFACTORY_TOKEN}
    image: https://some-department.some-company.com/artifactory/api/npm/mirror-nmpjs/structurizr/lite:2025.11.08
```
compose.yaml

Above for illustrative purpose only we applied the compose file to installing a package from NPM registry (mirrored in Artifactory) called: Structurizr.

You see how `compose.yaml` makes use of the environment variables `ARTIFACTORY_NPM_REGISTRY` and `ARTIFACTORY_TOKEN` set in `.env`.

## 4. Write a .dockerignore file

Create a `.dockerignore` file within the same directory where `Dockerfile` resides.

```
.env
```
.dockerignore

This prevents that the confidential information from the `.env` (like `ARTIFACTORY_TOKEN`) becomes part of the docker image.

## 5. Write a Dockerfile

Create a `Dockerfile` file within the same directory where all previous files reside.

```
ARG ARTIFACTORY_NPM_REGISTRY
ARG ARTIFACTORY_TOKEN
FROM node:12.21.0-alpine3.12
WORKDIR /usr/src/app
RUN apk update
RUN npm config set "//${ARTIFACTORY_NPM_REGISTRY}/:_authToken" "${ARTIFACTORY_TOKEN}"
COPY . .
EXPOSE 8080
CMD [ "npm", "run", "--debug" ]
```
Dockerfile

## 6. Run docker compose

With all above files in place, you can build and run the Docker image as follows:

```
docker compose up
```

Validate with:

```
docker ps
```



