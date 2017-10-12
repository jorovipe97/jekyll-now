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

Para comenzar, hablemos primero de conceptos claves cuando se trabaja con Bluetooth Low Energy:

WARNING NOTICE: No doy garantias de que lo que diga a continuación es 100% preciso ya que no soy una autoridad en el tema y estoy lejos de serlo, te recomiendo visitar la pagina de <a href="https://www.bluetooth.com/" targe="_blank">bluetooth</a> para que investigues por tu propia cuenta, sin embargo intentare cometer la menor cantidad de errores en la explicación y ser claro posible.

# Conceptos claves
Ahora si, despues de tanto rodeo empecemos

## GATT and GAP Profile
Imaginemos que tenemos un **despertador de mesa** que podemos configurar inalambricamente desde una aplicación de smartphone, supongamos que dicho reloj usa un dispositivo Bluetooth Low Energy (BLE) para lograrlo, ¿como sabe el smartphone que cerca de el hay un reloj queriendo comunicarce mediante BLE? bien, resulta que para saberlo, el dispositivo BLE de nuestro despertador hipotetico esta gritandole al mundo cada cierta cantidad de tiempo que el esta ahi, listo a que cualquiera que sea capaz de entenderlo se conecte a el, cuando el dispositivo BLE del despertador se encuentra en este rol de publicista desesperado se dice que esta en el rol GAP (Generic Access Profile) y cuando alguien se conecta al despertador el BLE pasa al rol GATT (Generic Attribute Profile).

Ademas existen los Services, Characteristics y Descriptors, que juegan un papel muy importante en el BLE cuando estamos en el rol GATT, pero ¿que son estas tres palabras tan raras?

## Services
Son un grupo de caracteristicas que cumplen una función especifica.

La especificación de Bluetooth Low Energy define algunos servicios como (<a href="https://www.bluetooth.com/specifications/gatt/services" target="_blank">GATT Services</a>) **Heart Rate**, **Blood Pressure**, **Environmental Sensing**, ademas tambien podrias crear un servicio totalmente nuevo como: **Detector de ronquidos**, cada servicio es identificado por un numero unico o UUID, por ejemplo el dispositivo BLE de nuestro despertador podria tener implementado el <a href="https://www.bluetooth.com/specifications/gatt/viewer?attributeXmlFile=org.bluetooth.service.current_time.xml" target="_blank">**Current Time**</a> service.

Un dispositivo bluetooth Low Energy generalmente puede tener mas de un servicio implementado.

## Characteristics
Los servicios tienen literalmente ***characteristics***, es decir para acceder a una caracteristica debemos primero acceder al servicio que la contiene, por ejemplo entre las caracteristicas que ofrece el **Hearth Rate** service estan: **Heart Rate Measurement** que se usa para enviar un ritmo cardiaco, **Body Sensor Location** usada para describir la ubicación prevista en la que se pone el dispositivo.

A su vez las ***charasteristics*** tienen ***Descriptors*** que se encargan 

## Descriptors

# Referencias
<a href="http://ticketmastermobilestudio.com/blog/android-bluetooth-low-energy-tutorial" target="_blank">Android BLE tutorial</a>
