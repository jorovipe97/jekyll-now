---
layout: post
title: Una interpretación del complemento a 2
published: true
---

En este breve post hablare de una interpretación del complemento a 2 que encontré en el libro (Computer Systems A programmer’s perspective)[https://www.amazon.com/Computer-Systems-Programmers-Perspective-MasteringEngineering/dp/0134123832] que me recomendó (@romualdo97)[https://romualdo97.github.io/]

El truco consiste en ver el complemento a 2 como una balanza.
!()[https://imgur.com/oj3RACa.gif]

Donde el bit más significativo representa el número más negativo posible, en este caso el **1000** significaría **-8**.

Si por ejemplo, se quisiera lograr un -5 deberíamos jugar con los otros bits para contrarrestar el peso del bit negativo:
!()[https://imgur.com/G3W0T1N]
