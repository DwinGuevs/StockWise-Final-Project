FROM node:22-alpine AS test

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY src ./src
COPY tests ./tests

RUN npm test


FROM node:22-alpine AS production

ENV NODE_ENV=production

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev && npm cache clean --force

COPY src ./src

USER node

EXPOSE 3000

CMD ["npm","start"]