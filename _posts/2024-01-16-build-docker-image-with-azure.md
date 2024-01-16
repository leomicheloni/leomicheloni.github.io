---
layout: post
title: Cómo crear una imágen de Docker sin tener Docker instalado gracias a Azure
published: true
---

# Introducción

Todos nos hemos encontrado en la encrucijada de tener que crear una imagen de Docker y no tener Docker instalado en nuestro equipo. En este post vamos a ver cómo podemos crear una imagen de Docker sin tener Docker instalado en nuestro equipo utilizando una cuenta de Azure.

## El problema
Primero necesitamos una cuenta de Azure y una instancia de Azure Container Registry a la que tengamos acceso.
La idea es que teniendo el código o el binario, lo que queramos meter en la imagen (Sea lo que sea, multistae o no) y el Dockerfile podemos utilizar la capacidad de Azure Container Registry de crear una imagen a partir de un Dockerfile.

> Importante, nuestro código o binario subirán a la nube durante el proceso

Entonces, teniendo ya el código o binario y el Dockerfile, vamos a crear la imagen.

## Pasos para crear la imagen y subirla al ACR
- Hacer login en Azure
- Seleccionar la suscripción
- Crear la imagen


``` powershell

az login

az account set --subscription KKKKKK-KKKK-KKKK-KKKK-KKKKKKKKK

az account show

az acr build --registry miurl.azurecr.io -f Dockerfile --image test:latest .

```
**como vemos el comando acr build es idéntico al docker build, pero en vez de utilizar el contexto local, lo hace en la nube.**

Y como dije antes, tomará un tiempo porque subirá el código o binario a la nube y luego creará la imagen.


![](../images/acr.png)

Una vez que termine, podemos ver la imagen en el ACR.

Enjoy!