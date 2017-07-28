---
layout: post
title: Implementando funciones boleanas con HDL - CS01
---

Este es el primer post de una serie de posts, en los cuales subiré mis aprendizajes acerca de un curso de computer science que estoy siguiendo: [nand2tetris](www.nand2tetris.org)

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
    5. ¿Donde esta el bit mas significativo?
2. Implementando el Or con chips Nand.
3. Implementando el And con chips Nand.
4. Implementando el Xor.
5. Implementando el Mux.
6. Implementando el DMux.
7. ¿Que es un chip multi-bit?
    1. Implementando el Not16.
    2. Implementando el And16.
    3. Implementando el Or16.
    4. Implementando el Mux16.        
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

**Respuesta**: Podriamos crear un subdirectorio en el directorio *~/chips/xor_temp/* y pasar alli nuestra implementacion del Xor y probarla, como en dicho subdirectorio temporal los chips basicos no estan, HardwareSimulator usara los chips nativos, si resulta que al hacer el test no ocurre ningun error seria una clara señal de que nuestro Xor ha sido bien implementado y que posiblemente el problema lo tenga la implementación nuestra de uno de los chips basicos, para descubrir cual es el chip con el problema se podria ahora ir pasando uno a uno dichos chips al directorio temporal anteriormente creado y ejecutar el test para que HardwareSimulator use la version del chip basico que fue implementado por nosotros mismos hasta que aparezca de nuevo el error y asi podriamos deducir cual es el chip defectuoso.

## Represantacion cananonica y sus implicaciones teoricas
Consiste en representar una funcion boleana como suma de miniterminos a partir de su tabla de verdad, por ejemplo vamos a continuación a usar esta tecnica para hallar la funcion boleana del Xor dada su tabla de verdad:

> poner imagen xor_miniterms_sum.png

Como se puede ver se necesita primero que todo tener definida la tabla de verdad de la función en cuestion, luego miramos en dicha tabla unicamente los casos en los que la salida de la funcion es **true**, se configuran en un and las entradas de una de las salidas que fueron **true** pasando por un **Not** las entradas que valieron 0, se hace el mismo proceso con las entradas de las demas salidas que valieron **true** y luego a estos terminos se les une con un **Or**.

La representación canonica tiene una implicación teorica importante y es que cualquier funcion boleana sin importar su complejida se puede expresar en la forma canonica, es decir cualquier funcion boleana se puede representar usando los operadores **And**, **Or** y **Not**.

Esto hara que en muchos chips no tengamos que pensar mucho para hallar su funcion boleana, sin embargo cabe decir que en chips mas complejos sera necesario el pensamiento.

## La función Nand y sus superpoderes.
La funcion **Nand** (al igual que la **Xor**) tiene una importancia teorica y practica, a partir de ella podemos construir la compuerta **And** la **Or** y la **Not**, ademas teniendo en cuenta que apartir de estas tres podemos implementar cualquier otra compuerta boleana sin importar su complejidad, encontramos por lo tanto que a partir de la **Nand** se puede construir cualquier otra compuerta boleana.

## ¿Donde esta el bit mas significativo?

> poner imagen significative-bits.png


# Implementando el chip Or con chips Nand
A continuación muestro el diagrama del CHIP

![Not](https://raw.githubusercontent.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/not.jpg)

```HDL
CHIP Not {
    IN in;
    OUT out;

    PARTS:
    // Put your code here:
    Nand(a=in, b=in, out=out);
}
```

Vemos que es una función sencilla, como tiene una unica entrada tiene solo dos posibles salidas y siempre sera el valor logico contrario al que entro.


# Implementando el Or con chips Nand

![Or using nand´s](https://rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/or_using_nands.jpg)

```HDL
CHIP Or {
    IN a, b;
    OUT out;

    PARTS:
    // Put your code here:
    Nand(a=a, b=a, out=nanda);
    Nand(a=b, b=b, out=nandb);
    Nand(a=nanda, b=nandb, out=out);
}
```
**Advertencia**: En adelante explicaré el circuito solo si considero que es importante hacer observaciones o aclarar puntos interesantes.


# Implementando el And con chips Nand

![And](https://rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/and_using_nands.jpg)
```HDL
CHIP And {
    IN a, b;
    OUT out;

    PARTS:
    // Put your code here:
    Nand(a=a, b=b, out=nand1);
    Nand(a=nand1, b=nand1, out=out);
}
```


# Implementando el Xor

![Xor](https://rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/xor_2.jpg)

Para hallar la funcion del boleana del Xor se uso la tecnica de los miniterminos (explicada en la seccion 1.iii.) esto simplificó el proceso significativamente.

Una vez hallada la funcion boleana fue relativamente sencillo implementar el chip en HDL.
```HDL
CHIP Xor {
    IN a, b;
    OUT out;

    PARTS:
    // Put your code here:
    Not(in=a, out=nota);
    Not(in=b, out=notb);

    And(a=a, b=notb, out=and1);
    And(a=nota, b=b, out=and2);

    Or(a=and1, b=and2, out=out);
}
```


# Implementando el Mux

![Mux](https://rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/Mux.jpg)

En esta ocación se volvio a usar la tecnica de los miniterminos abstrayendo el significado humanamente dado a las entradas y escribiendo una cruda tabla de verdad, luego a la funcion resultante se simplificó usando la ley identidad y la ley distributiva, disminuyendo asi el numero de chips necesarios para la implementacin del Mux.
```HDL
CHIP Mux {
    IN a, b, sel;
    OUT out;

    PARTS:
    // Put your code here:
    Not(in=sel, out=notsel);

    And(a=b, b=sel, out=and1);
    And(a=a, b=notsel, out=and2);

    Or(a=and1, b=and2, out=out);  
}
```

# Implementando el DMux

!(DMux)[https://raw.githubusercontent.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/dmux.jpg]

Para hallar la funcion boleana de este chip fue necesario hacer dos observaciones.
1. La función tiene dos salidas y por lo tanto tiene *"una funcion independiente para cada salida"*.
2. El **DMux** solo devuelve un **true** cuando en el **in** entra un **true**, cuando en el **in** hay un **false** no importa si el selector pin determina que salga por **a** o por **b** el **DMux** devolvera **false**.
```HDL
CHIP DMux {
    IN in, sel;
    OUT a, b;

    PARTS:
    // Put your code here:
    // a output
    Not(in=sel, out=notsel);
    And(a=in, b=notsel, out=a);

    // b output
    And(a=in, b=sel, out=b);
}
```

# ¿Que es un chip multi-bit?
Un chip multi-bit es un chip que realiza una operacion logica bit-wise es decir bit a bit, estos chips podrian interpretarse como la implementacion a bajo nivel de los operadores bit-wise de los lenguajes de programación de alto nivel como c++ ya que su funcionamiento externo es similar.

Para implementar la versión multi-bit de un chip determinado solo es cuestion de tomar la version one-bit de dicho chip y a cada input que tenga lo cambiamos por uno de n-bits y luego hacer lo mismo con la salida si corresponde, asi el Or multi-bit 16 seria tomar un or-one-bit y ponerle dos entradas de 16 bits a[16], b[16], y una salida de 16 bits (out[16]).

Es importante aclarar que la versión multi-bit de un chip realiza la operación bit a bit, por ejemplo en el ejemplo del or16, el bit a[0] se opera con el bit b[0] y el resultado sale por out[0], el a[1] con b[1] y el resultado sale por out[1] ... el a[15] con el b[15] y el resultado sale por out[15].

Dicho lo anterior procederé mostrando unícamente la implementación en HDL de la versión multi-bit de varios chips, sin entrar en detalles explicando la razon de dicha implementación puesto que lo dicho anteriormente aplica en los casos aqui mostrados.

# Implementando el Not16

