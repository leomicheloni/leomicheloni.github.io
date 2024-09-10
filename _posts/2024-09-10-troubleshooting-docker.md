---
layout: post
title: Docker tricks, crear una imagen para poder depurar un error.
published: true
---



## "Introducción"

En este caso queremos crear una imagen pero nos da algún tipo de error, y es complicado de resolver.
Bueno, lo que podemos hacer es apuntar los comandos que queremos ejecutar, crear una imagen con su base y hasta el punto que funciona y hacer que inicie con un comando que nos permita crear al contenedor e ingresar.

## Crear imagen a partir de una con problemas

```dockerfile
FROM node:20.12.0 AS builder
ARG environment=dev

WORKDIR /app
COPY ./package.json /app/
COPY ./yarn.lock /app/

RUN yarn
COPY . /app/
RUN yarn build:$environment

FROM nginx:1.21.5-alpine

EXPOSE 80/tcp
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=builder /app/dist /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
````

y vemos que nos da un error al hacer ``` RUN yarn ```

## ¿Qué podemos hacer?

Facil, creamos una imagen con la base que tenemos y hasta el punto que funciona, y luego la ejecutamos con un comando que nos permita ingresar al contenedor.
Pero como comando de inicio, usamos ```tail -f /dev/null``` para que se quede esperando y no se cierre.

```dockerfile
FROM node:20.12.0 AS builder
ARG environment=dev

WORKDIR /app
COPY ./package.json /app/
COPY ./yarn.lock /app/

CMD ["tail", "-f", "/dev/null"]
```

una vez hecho esto, podemos hacer un ```docker build -t myimage .``` y luego un ```docker run -it myimage /bin/bash``` para ingresar al contenedor y ver que es lo que pasa.

Desde dentro del container ejecutamos el comando que da problemas y vemos el error que nos da.

``` bash
yarn

.....

Request failed \"401 Unauthorized\""

```

Y vemos que nos da un error al intentar restaurar los paquetes.

Nada más, una forma sencilla de ir depurando error por error dentro de un contenedor.

Agregamos una línea para copiar el archivo de configuración de npm y listo.

``` dockerfile
FROM node:20.12.0 AS builder
ARG environment=dev

WORKDIR /app
COPY ./package.json /app/
COPY ./yarn.lock /app/
COPY .npmrc /app/ # <-- Agregamos esta línea

RUN yarn
COPY . /app/
RUN yarn build:$environment

FROM nginx:1.21.5-alpine

EXPOSE 80/tcp
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=builder /app/dist /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
```

Nos leemos.



