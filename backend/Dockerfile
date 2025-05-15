FROM node:14

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 80

ENV MONGO_USER=myuser
# NOTE - MONGO_PASSWORD was not set as an ENV for security.

CMD ["npm", "start"]
#CMD [ "node", "app.js" ]