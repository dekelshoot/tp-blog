FROM node:alpine

ENV CI=true
ENV WDS_SOCKET_PORT=0

WORKDIR /client
COPY . /client/
RUN npm i 
EXPOSE 3000
CMD [ "npm", "start" ]