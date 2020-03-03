FROM node:8
WORKDIR /node/src/app
COPY package*.json ./
RUN npm install --no-fund
COPY . .
EXPOSE 8888
CMD [ "node", "server.js" ]
