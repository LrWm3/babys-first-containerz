FROM node as builder

WORKDIR /home/node
COPY package.json /home/node/package.json
RUN npm install

COPY index.js /home/node/index.js
CMD node