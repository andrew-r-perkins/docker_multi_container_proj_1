Project to explore dockerizing an app with multiple containers:

This was an multi container exec to take an existing Node.js backend & React frontend and dockerize it.

High level picture

############       ###########       ############
# Mongo DB # <---> # Backend # <---> # Frontend #
#          #       # node.js #       # React    #
############       ###########       ############

1) Create network...
>>> docker network create backend-net 
>>> docker network ls

NETWORK ID     NAME           DRIVER    SCOPE
ce39f03efcbb   backend-net    bridge    local
63e5489b5c38   bridge         bridge    local
ed32b9401045   host           host      local
fa06cd040c66   none           null      local

2) Start mongoDB...
NOTE - First time this is run, the DB is initialized (and username / pwd is set), if applicable.
Run mongo w/ username / pwd
>>> docker run --rm --name mongo -v "mongodb-data:/data/db" -d --network backend-net -e MONGO_INITDB_ROOT_USERNAME=myuser -e MONGO_INITDB_ROOT_PASSWORD=mypassword mongo
>>> docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS       NAMES
45bb9522cd8f   mongo     "docker-entrypoint.s…"   23 minutes ago   Up 23 minutes   27017/tcp   mongo

NOTE 1 - Used -v noamed mount. A bind mount could have been used if I wanted to give direct access from local machine.

3) Set up backend

Create Dockerfile:
FROM node:14
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
EXPOSE 80
ENV MONGO_USER=myuser
CMD [ "node", "app.js" ]

To enable live code updates, in package.json, add:

  "scripts": {
    "start": "nodemon -L app.js"
  }
  
  and
  
  "devDependencies": {
    "nodemon": "3.1.10"
  }
  
and in Dockerfile replace:
CMD [ "node", "app.js" ]
with:
CMD [ "npm", "start" ]

Create .dockeringore:
Dockerfile
.git
node_modules

Build image:
>>> docker build -t backend .
>>> docker images backend

REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
backend      latest    ab5cd9ada7aa   3 minutes ago   1.37GB

Run container:
>>> docker run --rm --name backend -d --network backend-net -p 80:80 -e MONGO_PASSWORD=mypassword -v app-logs:/app/logs -v "C:\Users\andyt\Documents\training\docker\multi-01-starting-setup\backend:/app" -v /app/node_modules backend
>>> docker network connect frontend-net backend
>>> docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS       NAMES
0f0937011d9f   backend   "docker-entrypoint.s…"   4 minutes ago    Up 4 minutes    80/tcp      backend
45bb9522cd8f   mongo     "docker-entrypoint.s…"   51 minutes ago   Up 51 minutes   27017/tcp   mongo

>>> docker volume ls

DRIVER    VOLUME NAME
local     81e81e5b8b11925ef377bce7f5b7ca1ba23ac088098df10b21b2867ac5dd7b69
local     app-logs

4) Set up frontend

Create Dockerfile:
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

NOTE - I was going to use frontend-net, but I discovered that as react runs in the browser and now in the container, this approch doesn't work. So in the code I use something like 'http://localhost/goals' 
which makes a call to the internet (outside of the container), localhost is resolved to the local machine where port 80 has already been expose and published by backend.

NOTE - I tried adding live code updated to the frontend, similar to the backend. For React, it was suggested to use the following switches in the 'docker run' when starting the frontend. THIS DID NOT WORK.
 -v "C:\Users\andyt\Documents\training\docker\multi-01-starting-setup\frontend\src:/app/src"
 -v /app/node_modules 
 -e HOST=0.0.0.0 frontend
The FULL SOLUTION is to Fully embrace WSL 2 - see https://www.docker.com/blog/docker-desktop-wsl-2-best-practices/

>>> docker build -t frontend .
>>> docker run -it --rm --name frontend -p 3000:3000 -d frontend
