---
layout: post
title: Implementando funciones boleanas con HDL - CS01
---

Este es el primer post de una serie de posts, en los cuales subiré mis aprendizajes acerca de un curso de computer science que estoy siguiendo...

En esta ocasión el objetivo es implementar las siguientes funciones lógicas (chips) ver lista-1:

- Not
- And
- Or
- Xor
- Mux
- DMux
- Not16
- And16
- Or16
- Mux16
- Or8Way
- Mux4Way16
- Mux8Way16
- DMux4Way
- DMux8Way

El único problema es que se pone una restricción inicial: Sólo se puede usar la función Nand para construir las funciones básicas (Or, And, Not) y a partir de las anteriores y las que se vayan construyendo posteriormente se deben construir las demás.

Para facilitar la compresión dividiré el post en las siguientes partes:

1. Conceptos importantes.
    1. ¿Porque es importante realizar en orden el proyecto?
    2. HardwareSimulator y tecnicas para hacer debugging.
    3. Representación canónica y sus implicaciones teóricas.
    4. La función Nand y sus superpoderes.
2. Implementando el Or con chips Nand.
3. Implementando el And con chips Nand.
4. Implementando el Xor.
5. Implementando el Mux.
6. Implementando el DMux.
7. ¿Que es un chip multi-bit?
    1. Implementando la versión. multi-bit de los anteriores chips.
8. ¿Que es un chip multi-way?
    1. Implementando el Or8Way.
    2. Implementando el Mux4Way16.
    3. Implementando el Mux8Way16.
    4. Implementando el DMux4Way.
    5. Implementando el DMux8Way.
    6. Bibliografía.
    
# Conocimientos importantes:
Para la realizacion del problema planteado en esta ocación resulta muy util los siguientes conocimientos:

## ¿Porque es importante realizar en orden el proyecto?
Es importante ir implementando los chips de una modo progresivo, es decir en el orden en el que se dio en la lista-1, esto debido a que por ejemplo el **Xor** y el **Mux** dependen del **And** el **Or** y el **Not**, a su vez el **Mux4Way16** puede depender del **Xor** y otros chips implementados anteriormente, esto permitira disminuir drasticamente el uso de chips que usa cada chip determinado.

## HardwareSimulator y tecnicas para hacer debugging
HardwareSimulator busca los chips que usemos en un chip cuya implementacion se propia en un orden especifica si por ejemplo vamos a  implementar un **Xor** cuya implementacion depende del **And** el **Or** y el **Not** (chips basicos), y suponiendo que estamos implementando el **Xor** en el directorio */home/jose/chips/Xor.hdl* el buscara los chips basicos en el mismo directorio donde se encuentra el *Xor.hdl* si ahi no se encuentran los buscara en un directorio propio del HardwareSimulator donde se encuentran muchos chips ya implementados, pero entonces ¿como usar esto para hacer **debugging**?

Supongamos que tenemos nuestra implementacion del Xor y los chips basicos de los cuales depende este en el mismo directorio (HardwareSimulator) */home/jose/chips/*, ¿como sabemos si el error esta en la implementacion misma del Xor o en los chips basicos que este usa?

Respuesta: Podriamos crear un subdirectorio en el directorio *~/chips/xor_temp/* y pasar alli nuestra implementacion del Xor y probarla, como en dicho subdirectorio temporal los chips basicos no estan, HardwareSimulator usara los chips nativos, si resulta que al hacer el test no ocurre ningun error seria una clara señal de que nuestro Xor ha sido bien implementado y que posiblemente el problema lo tenga la implementación nuestra de uno de los chips basicos, para descubrir cual es el chip con el problema se podria ahora ir pasando uno a uno dichos chips al directorio temporal anteriormente creado y ejecutar el test para que HardwareSimulator use la version del chip basico que fue implementado por nosotros mismos hasta que aparezca de nuevo el error y asi podriamos deducir cual es el chip defectuoso.

## Represantacion cananonica y sus implicaciones teoricas
Consiste en representar una funcion boleana como suma de miniterminos a partir de su tabla de verdad, por ejemplo vamos a continuación a usar esta tecnica para hallar la funcion boleana del Xor dada su tabla de verdad:

> poner imagen xor_miniterms_sum.png

Como se puede ver se necesita primero que todo tener definida la tabla de verdad de la función en cuestion, luego miramos en dicha tabla unicamente los casos en los que la salida de la funcion es **true**, se configuran en un and las entradas de una de las salidas que fueron **true** pasando por un **Not** las entradas que valieron 0, se hace el mismo proceso con las entradas de las demas salidas que valieron **true** y luego a estos terminos se les une con un **Or**.

La representación canonica tiene una implicación teorica importante y es que cualquier funcion boleana sin importar su complejida se puede expresar en la forma canonica, es decir cualquier funcion boleana se puede representar usando los operadores **And**, **Or** y **Not**.

Esto hara que en muchos chips no tengamos que pensar mucho para hallar su funcion boleana, sin embargo cabe decir que en chips mas complejos sera necesario el pensamiento.

## La función Nand y sus superpoderes.
