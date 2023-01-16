# Установка и контейнеризация strapi CMS

Последовательность действий:
- Создание проекта __npx create-strapi-app__, указать название и выбрать тип рекомендуемый тип сборки
- Внутри проекта создать файл __Dockerfile__
```
FROM node:16-alpine
RUN apk update && apk add --no-cache build-base gcc autoconf automake zlib-dev libpng-dev nasm bash vips-dev
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}
WORKDIR /opt/
COPY ./package.json ./yarn.lock ./
ENV PATH /opt/node_modules/.bin:$PATH
RUN yarn config set network-timeout 600000 -g && yarn install
WORKDIR /opt/app
COPY ./ .
RUN yarn build
EXPOSE 1337
CMD ["yarn", "start"]
```
- Далее создать __.dockerignore__
```
.tmp/
node_modules/
.cache/
.git/
build/
data/
```
- После чего необходимо собрать образ __docker build -t strapi-prod .__
- Далее создаем __docker-compose.yaml__
```
version: '3'

networks:
  strapi_net:
    driver: bridge

services:
  strapi:
    container_name: strapi
    # build: .
    image: strapi-dev
    restart: unless-stopped
    env_file: .env
    environment:
      DATABASE_CLIENT: ${DATABASE_CLIENT}
      DATABASE_HOST: ${DATABASE_HOST}
      DATABASE_PORT: ${DATABASE_PORT}
      DATABASE_NAME: ${DATABASE_NAME}
      DATABASE_USERNAME: ${DATABASE_USERNAME}
      DATABASE_PASSWORD: ${DATABASE_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      ADMIN_JWT_SECRET: ${ADMIN_JWT_SECRET}
      APP_KEYS: ${APP_KEYS}
      NODE_ENV: ${NODE_ENV}
    volumes:
      - ./config:/opt/app/config
      - ./src:/opt/app/src
      - ./package.json:/opt/package.json
      - ./yarn.lock:/opt/yarn.lock
      - ./.env:/opt/app/.env
      - ./public/uploads:/opt/app/public/uploads
    ports:
      - '1337:1337'
    networks:
      - strapi_net
    depends_on:
      - strapi_db

  strapi_db:
    container_name: strapi_db
    # platform: linux/amd64 #for platform error on Apple M1 chips
    restart: unless-stopped
    env_file: .env
    image: postgres:12.0-alpine
    environment:
      POSTGRES_USER: ${DATABASE_USERNAME}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
      POSTGRES_DB: ${DATABASE_NAME}
    volumes:
      # - strapi-data:/var/lib/postgresql/data/ #using a volume
      - ./db_data:/var/lib/postgresql/data/ # if you want to use a bind folder

    ports:
      - '1234:5432'
    networks:
      - strapi_net

volumes:
  db_data:
```
- Не забываем добавить данные в __.env__
```
NODE_ENV=production
DATABASE_CLIENT=postgres
DATABASE_HOST=strapiDB
DATABASE_PORT=5432
DATABASE_NAME=strapiDB
DATABASE_USERNAME=developer
DATABASE_PASSWORD=password
```
- После чего можем собрать и запустить приложение __docker-compose up --build -d__
