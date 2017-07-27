---
layout: post
title: Implementando funciones boleanas con HDL - CS01
---

Este es el primer post de una serie de posts, en los cuales subiré mis aprendizajes acerca de un curso de computer science que estoy siguiendo...

En esta ocasión el objetivo es implementar las siguientes funciones lógicas (chips):

-Not
-And
-Or
-Xor
-Mux
-DMux
-Not16
-And16
-Or16
-Mux16
-Or8Way
-Mux4Way16
-Mux8Way16
-DMux4Way
-DMux8Way

El único problema es que se pone una restricción inicial: Sólo se puede usar la función Nand para construir las funciones básicas (Or, And, Not) y a partir de las anteriores y las que se vayan construyendo posteriormente se deben construir las demás.

Para facilitar la compresión dividiré el post en las siguientes partes:

1. Conocimientos necesarios.
1.a Representación canónica y sus implicaciones teóricas.
1.b La función Nand y sus superpoderes.
2. Implementando el Or con chips Nand.
3. Implementando el And con chips Nand.
4. Implementando el Xor.
5. Implementando el Mux.
6. Implementando el DMux.
7. ¿Que es un chip multi-bit?
7.b Implementando la versión. multi-bit de los anteriores chips.
8. ¿Que es un chip multi-way?
8.a Implementando el Or8Way.
8.b Implementando el Mux4Way16.
8.c Implementando el Mux8Way16.
8.d Implementando el DMux4Way.
8.e Implementando el DMux8Way.
9. Bibliografía.