FROM node:alpine3.17
EXPOSE 8080
WORKDIR /opt/server
COPY package.json /opt/server/
RUN npm install
COPY server.js /opt/server/
CMD ["node", "server.js"]