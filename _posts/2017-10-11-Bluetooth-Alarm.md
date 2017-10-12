---
layout: post
title: Primeros pasos con Bluetooth Low Energy
published: false
---

En esta ocación vamos a realizar una alarma en una aplicación Android que mediante Bluetooth Low Energy encendera un led conectado a un dispositivo wearable, con esto queremos lograr una primera introducción practica a Bluetooth Low Energy.

Antes de entrar en detalles vamos a definir el problema que queremos solucionar con todo lo que aqui se va a explicar:

- Se requiere desarrollar un dispositivo wearable que puedan utilizar las personas al dormir con su pareja. La función del dispositivo será implementar una alarma que sólo despierte a la persona que lleva puesto el dispositivo.

El sistema constara de dos partes, la primera es una aplicación móvil que nos permitirá programar la hora de la alarma y la segunda parte importante es el hardware que hará el papel de Wearable que sera lo que el usuario se pondrá en el cuerpo (Brazalete) para que lo despierte solo a el.

![](https://imgur.com/IxMHB4R.gif)

NOTA: En el anterior dibujo el brazalete se ve flamantemente incomodo, no te dejes llevar, esto solo es un diagrama ilustrativo, lo mismo tendre que decir del prototipo que desarrollaremos asi que en este punto sera necesario que tengas la imaginación bien encendida.

