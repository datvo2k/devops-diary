# DOCKER USER AND LAYER CACHING

## 1.Add the Docker user

Rootless mode allows running the Docker daemon and containers as a non-root user to mitigate potential vulnerabilities in the daemon and the container runtime.
Any outsider can access this and hack the entire Container with all the other file running inside container.   
In the example container is going run root user:
```

FROM node:14.3.0-alpine3.10 as builder
ENV NODE_ENV="production"

COPY ./src /app
COPY ./package.json /app/package.json

WORKDIR /app

# Install Node.js dependencies defined in '/app/packages.json'
RUN npm install --production

FROM node:14.3.0-alpine3.10

# Author
LABEL maintainer="datvo2k@gmail.com"

ENV NODE_ENV="production"
COPY --from=builder /app /app

RUN npm install pino-pretty -g

WORKDIR /app
ENV PORT 5000
EXPOSE 5000
RUN echo "Start..."

# Start the application
CMD node application.js | pino-pretty -t SYS:standard -i hostname,pid,res,req,responseTime
```
Creating a non-root user and user authorization on folder:
```
FROM node:14.3.0-alpine3.10 as builder
ENV NODE_ENV="production"

# Copy app's source code to the /app directory
COPY ./src /app
COPY ./package.json /app/package.json

# The application's directory will be the working directory
WORKDIR /app

# Install Node.js dependencies defined in '/app/packages.json'
RUN npm install --production

FROM node:14.3.0-alpine3.10

# Author
LABEL maintainer="datvo2k@gmail.com"

ENV NODE_ENV="production"
COPY --from=builder /app /app

# Add group and user
RUN addgroup -S admin && adduser -S admin -G admin
RUN chown -R admin:admin /app
RUN chown -R admin:admin /usr/local/lib/node_modules

RUN npm install pino-pretty -g

WORKDIR /app
ENV PORT 5000
EXPOSE 5000
RUN echo "Start..."

USER devops

# Start the application
CMD node application.js | pino-pretty -t SYS:standard -i hostname,pid,res,req,responseTime
```

## 2. Layer Caching:
Docker creates layers while constructing an image to speed up subsequent deployments and builds. Every layer is a difference or delta from the one that came before it.
Every layer will be constructed for the first time, and the procedure will take some time. Your Dockerfile's RUN, COPY, or ADD instructions each produce a read-only layer (essentially, just a bunch of files). Docker will make advantage of mirror caching because nothing has changed.   
The CI process might benefit greatly from layer caching in terms of time savings.
The concept behind this is to enable image download from Hub and Docker repositories during CI processes.  
As an illustration, I'll use AzureDevops:
```
 - task: Docker@2
        displayName: "Registry Login"
        inputs:
          containerRegistry: "$(dockerhub)" # account dockerhub
          command: 'login'

      - script: "docker pull $(imageRepo):latest"   # name your repo in DockerHub
        displayName: "Pull latest for layer caching"
        condition: ne(variables['USE_BUILD_CACHE'], 'false')
        continueOnError: true # for the first build, tags will not exist yet

      - task: Docker@2
        displayName: "Build image"
        inputs:
          containerRegistry: "$(dockerhub)"
          repository: "$(imageRepo)"
          command: "build"
          arguments: "--cache-from=$(imageRepo):latest"
          Dockerfile: "$(Build.SourcesDirectory)/Dockerfile"
          tags: |
            $(tag)
            latest
      - task: Docker@2
        displayName: "Push image"
        inputs:
          containerRegistry: "$(dockerhub)"
          repository: "$(imageRepo)"
          command: "push"
          tags: |
            $(tag)
            latest
```










