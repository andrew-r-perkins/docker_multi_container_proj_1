FROM node:14

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 80

ENV MONGO_USER=myuser

CMD ["npm", "start"]
#CMD [ "node", "app.js" ]