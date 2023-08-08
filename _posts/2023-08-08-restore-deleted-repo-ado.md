---
layout: post
title: Cómo recuperar repositorios git en Azure DevOps
published: true
---

> Aclaración, este método solo funciona durante 30 días después del borrado.

# Introducción
Bien, en este post vamos a ver cómo recuperar un repositorio git que hemos borrado en Azure DevOps. Estos repositorios se mantienen en un Recycle Bin durante 30 días, pasado ese tiempo, se borran definitivamente.

# Recuperar repositorio
Para hacerlo tendremos que utilizar la API de Azure DevOps. Para ello, vamos a utilizar Postman.

## Obtener token
Lo primero que tenemos que hacer es obtener un token para poder acceder a la API. Para ellos vamos a ir a nuestro perfil y crear un token (PAT).

Con permisos para lo que necesitamos, en este caso, repositorios.

![](../images/menu_pat.png)

![](../images/select_pat.png)

Copiamos el token y lo guardamos en un lugar seguro (si lo olvidamos, tendremos que crear otro).


## Consultar repositorios usando la API

Ahora vamos a consultar los repositorios que tenemos en nuestra organización. Para ello, vamos a utilizar la siguiente URL:

```url
{{devopsOrganization}}/{{devopsProject}}/_apis/git/repositories/?api-version=7.0
```

En mi caso es:

```url  
https://dev.azure.com/leomicheloni/test_git/_apis/git/repositories/?api-version=7.0
```
y usaremos basic auth con nuestro usuario y el token que acabamos de crear.
Para ellos tenemos que ir a la pestaña Authorization y seleccionar Basic Auth, o crear el header manualmente con el usuario y el token.
Por ejemplo: 

```
usuario@email.com:PAT

```
En base 64 y lo ponemos como header anteponiendo la palabra Basic.

Authorization: Basic dXNlcm5hbWVAZW1haWwuY29tOlBBVA==

Bien, con esto obtenemos los repositorios actuales.

![](../images/get_repo_response.png)


## Consultar repositorios borrados

En este caso la url es:
    
```url
{{devopsOrganization}}/{{devopsProject}}/_apis/git/recycleBin/repositories/?api-version=7.0
```

Y el resto es igual que antes. En este caso no hay nada.

``` json
{
    "value": [],
    "count": 0
}
```

Así que borramos un repositorio y volvemos a consultar.

![](../images/deleted_repo.png)

## Recuperar repositorio

Lo que tenemos que hacer es copiar el id del repositorio borrado y hacer estaba vez un PATH.

```url
{{devopsOrganization}}/{{devopsProject}}/_apis/git/recycleBin/repositories/[REPO_ID]?api-version=7.0
```

En mi caso:

```url
https://dev.azure.com/leomicheloni/test_git/_apis/git/recycleBin/repositories/a852e4c3-7ebc-4915-ae2a-e673d79b60eb?api-version=7.0
```

Adcionalmente tenemos que indicar content-type: application/json y colocar en el body:

```json
{
    "deleted": false
}
```


Ejecutamos el PATCH con la misma autenticación y voilá, ya tenemos el repositorio de vuelta.

Enjoy!
