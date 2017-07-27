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
    2. ¿Orden de lectura del HardwareSimulator?
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

## Represantacion cananonica y sus implicaciones teoricas
Consiste en representar una funcion boleana como suma de miniterminos a partir de su tabla de verdad, por ejemplo vamos a continuación a usar esta tecnica para hallar la funcion boleana del Xor dada su tabla de verdad:

> poner imagen xor_miniterms_sum.png

Como se puede ver se necesita primero que todo tener definida la tabla de verdad de la función en cuestion, luego miramos en dicha tabla unicamente los casos en los que la salida de la funcion es **true**, 

Esto hara que en muchos chips no tengamos que pensar mucho para hallar su funcion, sin embargo cabe decir que en chips mas complejos sera necesario el pensamiento.

