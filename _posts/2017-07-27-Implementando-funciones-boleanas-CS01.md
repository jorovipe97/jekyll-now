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

- [Conocimientos importantes:](#conocimientos-importantes)
  * [¿Porque es importante realizar en orden el proyecto?](#porque-es-importante-realizar-en-orden-el-proyecto)
  * [HardwareSimulator y tecnicas para hacer debugging](#hardwaresimulator-y-tecnicas-para-hacer-debugging)
  * [Represantacion cananonica y sus implicaciones teoricas](#represantacion-cananonica-y-sus-implicaciones-teoricas)
  * [La función Nand y sus superpoderes.](#user-content-la-función-nand-y-sus-superpoderes)
  * [¿Donde esta el bit menos significativo?](#donde-esta-el-bit-menos-significativo)
- [Implementando el chip Not con chips Nand](#implementando-el-chip-not-con-chips-nand)
- [Implementando el Or con chips Nand](#implementando-el-or-con-chips-nand)
- [Implementando el And con chips Nand](#implementando-el-and-con-chips-nand)
- [Implementando el Xor](#implementando-el-xor)
- [Implementando el Mux](#implementando-el-mux)
- [Implementando el DMux](#implementando-el-dmux)
- [¿Que es un chip multi-bit?](#que-es-un-chip-multi-bit)
  * [Implementando el Not16](#implementando-el-not16)
  * [Implementando el And16](#implementando-el-and16)
  * [Implementando el Or16](#implementando-el-or16)
  * [Implementando el Mux16](#implementando-el-mux16)
- [¿Que es un chip multi-way?](#que-es-un-chip-multi-way)
  * [Implementando el Or8Way](#implementando-el-or8way)
  * [Implementando el Mux4Way16](#implementando-el-mux4way16)
  * [Implementando el Mux8Way16](#implementando-el-mux8way16)
  * [Implementando el DMux4Way](#implementando-el-dmux4way)
  * [Implementando el DMux8Way](#implementando-el-dmux8way)
- [Lista de chequeo y auto evaluación.](#lista-de-chequeo-y-auto-evaluación)
    
    
# Conocimientos importantes:
Para la realizacion del problema planteado en esta ocación resulta muy util los siguientes conocimientos:

## ¿Porque es importante realizar en orden el proyecto? - concepto8a
Es importante ir implementando los chips de una modo progresivo, es decir en el orden en el que se dio en la lista-1, esto debido a que por ejemplo el **Xor** y el **Mux** dependen del **And** el **Or** y el **Not**, a su vez el **Mux4Way16** puede depender del **Xor** y otros chips implementados anteriormente, esto permitira disminuir drasticamente el uso de chips que usa cada chip determinado.

## HardwareSimulator y tecnicas para hacer debugging - concepto8b
HardwareSimulator busca los chips que usemos en un chip cuya implementacion se propia en un orden especifica si por ejemplo vamos a  implementar un **Xor** cuya implementacion depende del **And** el **Or** y el **Not** (chips basicos), y suponiendo que estamos implementando el **Xor** en el directorio */home/jose/chips/Xor.hdl* el buscara los chips basicos en el mismo directorio donde se encuentra el *Xor.hdl* si ahi no se encuentran los buscara en un directorio propio del HardwareSimulator donde se encuentran muchos chips ya implementados, pero entonces ¿como usar esto para hacer **debugging**?

Supongamos que tenemos nuestra implementacion del Xor y los chips basicos de los cuales depende este en el mismo directorio (HardwareSimulator) */home/jose/chips/*, ¿como sabemos si el error esta en la implementacion misma del Xor o en los chips basicos que este usa?

**Respuesta**: Podriamos crear un subdirectorio en el directorio *~/chips/xor_temp/* y pasar alli nuestra implementacion del Xor y probarla, como en dicho subdirectorio temporal los chips basicos no estan, HardwareSimulator usara los chips nativos, si resulta que al hacer el test no ocurre ningun error seria una clara señal de que nuestro Xor ha sido bien implementado y que posiblemente el problema lo tenga la implementación nuestra de uno de los chips basicos, para descubrir cual es el chip con el problema se podria ahora ir pasando uno a uno dichos chips al directorio temporal anteriormente creado y ejecutar el test para que HardwareSimulator use la version del chip basico que fue implementado por nosotros mismos hasta que aparezca de nuevo el error y asi podriamos deducir cual es el chip defectuoso.

## Represantacion cananonica y sus implicaciones teoricas - concepto2-concepto3
Consiste en representar una funcion boleana como suma de miniterminos a partir de su tabla de verdad, por ejemplo vamos a continuación a usar esta tecnica para hallar la funcion boleana del Xor dada su tabla de verdad:

![Miniterm technique example](https://cdn.rawgit.com/jorovipe97/computer_science_code/master/projects_resources/01/xor_miniterms_sum.png)

Como se puede ver se necesita primero que todo tener definida la tabla de verdad de la función en cuestion, luego miramos en dicha tabla únicamente los casos en los que la salida de la función es **true**, se configuran en un and las entradas de una de las salidas que fueron **true** pasando por un **Not** las entradas que valieron 0, se hace el mismo proceso con las entradas de las demas salidas que valieron **true** y luego a estos terminos se les une con un **Or**.

La representación canónica tiene una implicación teórica importante y es que cualquier función boleana sin importar su complejidad se puede expresar en la forma canónica, es decir cualquier función boleana se puede representar usando los operadores **And**, **Or** y **Not**.

Esto hara que en muchos chips no tengamos que pensar mucho para hallar su funcion boleana, sin embargo cabe decir que en chips mas complejos sera necesario el pensamiento.

**Ejemplo**
Vamos a representar como suma de miniterminos la siguiente funcion booleana:

![](https://imgur.com/6DFrPss.gif)

Lo primero que debemos hacer es observar los casos en los que la salida de la funcion es **true** para agruparlos con el operador Or.

![](https://imgur.com/W7Eadeo.gif)

Vemos que hay 3 entradas que hacen la salida de la funcion booleana **true**, asi que procedemos a sumar (agrupar con el operador or) dichos miniterminos:

![](https://latex.codecogs.com/gif.latex?m_1&plus;m_2&plus;m_3=\overline{a}&space;\cdot&space;\overline{m}&space;\cdot&space;w&plus;\overline{a}&space;\cdot&space;m&space;\cdot&space;\overline{w}&plus;\overline{a}\cdot&space;m&space;\cdot&space;w)

Y con esta sencilla tecnica tendriamos una función que puede ser representada con la anterior tabla de verdad.

## Simplificando función boleana - concepto4
![](https://imgur.com/SOGv8Lw.gif)

Para simplificar funciones booleanas nos ayudaremos de la tabla anterior de teoremas del algebra de boole, y supongamos que queremos simplificar la función obtenida en el punto anterior con la suma de miniterminos:

![](https://latex.codecogs.com/gif.latex?\overline{a}&space;\cdot&space;\overline{m}&space;\cdot&space;w&plus;\overline{a}&space;\cdot&space;m&space;\cdot&space;\overline{w}&plus;\overline{a}\cdot&space;m&space;\cdot&space;w)

Aplicando la ley distributiva en su forma **or**:

![](https://latex.codecogs.com/gif.latex?\overline{a}&space;\cdot&space;[\overline{m}&space;\cdot&space;w&space;&plus;&space;m&space;\cdot&space;\overline{w}&plus;&space;m&space;\cdot&space;w])

Volviendo a aplicar la ley distributiva en su forma **or**:

![](https://latex.codecogs.com/gif.latex?\overline{a}&space;\cdot&space;[\overline{m}&space;\cdot&space;w&space;&plus;&space;m&space;\cdot&space;(\overline{w}&plus;&space;w)])

Aplicando la ley inversa en su forma **or**:

![](https://latex.codecogs.com/gif.latex?\overline{a}&space;\cdot&space;[\overline{m}&space;\cdot&space;w&space;&plus;&space;m&space;\cdot&space;1])

Por ultimo, aplicando la ley distributiva en su forma **or** obtenemos:

![](https://latex.codecogs.com/gif.latex?\overline{a}&space;\cdot&space;\overline{m}&space;\cdot&space;w&space;&plus;&space;\overline{a}&space;\cdot&space;m)

La cual como podemos ver es una version simplificada de la funcion que obtuvimos originalmente usando la tecnica de los miniterminos.

## Demostrando teoremas utilizando tablas de verdad - concepto5

![](https://imgur.com/SOGv8Lw.gif)
Procederemos ahora a demostrar los dos ultimos teoremas de la tabla anterior utilizando tablas de verdad:

### Absorption law AND form
![](https://latex.codecogs.com/gif.latex?A(A&plus;B)=A)

Tabla de verdad de A(A+B)
<table style="width: 100%">
	<tr>
		<th>A</th>
		<th>B</th>
		<th>A(A+B)</th>
	<tr>
	<tr>
		<td>0</td>
		<td>0</td>
		<td>0</td>
	<tr>
	<tr>
		<td>0</td>
		<td>1</td>
		<td>0</td>
	<tr>
	<tr>
		<td>1</td>
		<td>0</td>
		<td>1</td>
	<tr>
	<tr>
		<td>1</td>
		<td>1</td>
		<td>1</td>
	<tr>
	
</table>

## La función Nand y sus superpoderes
La funcion **Nand** (al igual que la **Xor**) tiene una importancia teorica y practica, a partir de ella podemos construir la compuerta **And** la **Or** y la **Not**, ademas teniendo en cuenta que apartir de estas tres podemos implementar cualquier otra compuerta boleana sin importar su complejidad, encontramos por lo tanto que a partir de la **Nand** se puede construir cualquier otra compuerta boleana.

## Donde esta el bit menos significativo - concepto8d-concepto8e

![Significative bit](https://cdn.rawgit.com/jorovipe97/computer_science_code/master/projects_resources/01/significative-bits.png)
Es importante tener en cuenta que cuando trabajemos con entradas o salidas multi-bit el bit menos significativo estara ubicado en el indice 0, tal como se muestra en la imagen anterior, esto se menciona porque se puede por habíto del programador creer que el bit ubicado en el indice 0 es el bit mas significativo cuando en realidad no lo es, esta confusión puede llevar a problemas inesperados a la hora de implementar algun chip, por consiguiente el bit mas significativo estara en el indice n-1, donde n es el numero de bits del bus en cuestion.

## Pines desconectados - concepto8c
![](https://imgur.com/skMDoSv.gif)

Si por ejemplo tenemos un bus de entrada con 8 bits y solo conectamos 4 pines de dicho bus, los pines que queden desconectados tendran un valor indeterminado, es decir, pueden ser true o falso pero no se sabe en que estado estan.

## Valor de los pines de entrada y salida - concepto8f
![](https://imgur.com/skMDoSv.gif)

Para asegurarnos de haber entendido el concepto anterior vamos a intentar predecir los valores de los pines **in**, **out**, **x** e **y**. respectivamente, se debe tener en cuenta que **v** es un bus de 3 bits, *v=100*, es decir **v[0]=0**, **v[1]=0** y **v[2]=1**.

### in:
- in[7] = true
- in[6] = true
- in[5] = ? **// desconectado, valor indeterminado** 
- in[4] = v[2] = true
- in[3] = v[1] = false
- in[2] = v[0] = false
- in[1] = ? **// desconectado, valor indeterminado** 
- in[0] = ? **// desconectado, valor indeterminado**

### out
Ahora supongamos que la logica del Chip Foo nos da el siguiente output.
- out[7] = true
- out[6] = true
- out[5] = false **// Notese que el hecho de que in[5] fuera indeterminado, no significa que el valor que habia alli era false, pues esto asume que en la logica del chip Foo in[5]=out[5] y esto no es necesariamente cierto**
- out[4] = false
- out[3] = true
- out[2] = true
- out[1] = true
- out[0] = false

### x
- x[3] = out[3] = true
- x[2] = out[2] = true
- x[1] = out[1] = true
- x[0] = out[0] = false

### y
- y[4] = out[6] = true
- y[3] = out[5] = false
- y[2] = out[4] = false
- y[1] = out[3] = true
- y[0] = out[2] = true

## Orden de declaracion en HDL - concepto1
Se debe tener en cuenta que HDL es un lenguaje de descripción en el que lo que importa es la forma en que se conectan los componentes y no el orden en el que se conectan.

Para poner un ejemplo, imaginemos que estamos conectando el cargador de nuestro dispositivo movil, en principio da igual si primero conectamos el cable al celular y despues al toma de corriente o si primero conectamos el cable al toma y despues al celular, el resultado obtenido con ambos procedimientos es el mismo: **un dispositivo movil conectado al toma de corriente**.

# Implementando el chip Not con chips Nand
A continuación muestro el diagrama del CHIP

![Not](https://cdn.rawgit.com/jorovipe97/computer_science_code/1e0838ec/projects_resources/01/not_2.png)

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

![Or using nand´s](https://cdn.rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/or_using_nands.jpg)

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

![And](https://cdn.rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/and_using_nands.jpg)
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

![Xor](https://cdn.rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/xor_2.jpg)

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

![Mux](https://cdn.rawgit.com/jorovipe97/computer_science_code/f417c6049fa7343fad3b63fd36f3a76fafbb2600/projects_resources/01/Mux.jpg)

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

![DMux](https://cdn.rawgit.com/jorovipe97/computer_science_code/b0811151/projects_resources/01/dmux.jpg)

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

Dicho lo anterior procederé mostrando únicamente la implementación en HDL de la versión multi-bit de varios chips, sin entrar en detalles explicando la razon de dicha implementación puesto que lo dicho anteriormente aplica en los casos aqui mostrados.

## Implementando el Not16
```HDL
CHIP Not16 {
    IN in[16];
    OUT out[16];

    PARTS:
    // Put your code here:
    Not(in=in[0], out=out[0]);
    Not(in=in[1], out=out[1]);
    Not(in=in[2], out=out[2]);
    Not(in=in[3], out=out[3]);
    Not(in=in[4], out=out[4]);
    Not(in=in[5], out=out[5]);
    Not(in=in[6], out=out[6]);
    Not(in=in[7], out=out[7]);
    Not(in=in[8], out=out[8]);
    Not(in=in[9], out=out[9]);
    Not(in=in[10], out=out[10]);
    Not(in=in[11], out=out[11]);
    Not(in=in[12], out=out[12]);
    Not(in=in[13], out=out[13]);
    Not(in=in[14], out=out[14]);
    Not(in=in[15], out=out[15]);
}
```
**Nota para el lector**:  En este HDL no se cuenta con bucles por lo tanto es necesario usar las tecnicas de copy and paste la cantidad de veces que sea necesaria (en este caso 15 veces).

## Implementando el And16
```HDL
CHIP And16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    // Put your code here:
    And(a=a[0], b=b[0], out=out[0]);
    And(a=a[1], b=b[1], out=out[1]);
    And(a=a[2], b=b[2], out=out[2]);
    And(a=a[3], b=b[3], out=out[3]);
    And(a=a[4], b=b[4], out=out[4]);
    And(a=a[5], b=b[5], out=out[5]);
    And(a=a[6], b=b[6], out=out[6]);
    And(a=a[7], b=b[7], out=out[7]);
    And(a=a[8], b=b[8], out=out[8]);
    And(a=a[9], b=b[9], out=out[9]);
    And(a=a[10], b=b[10], out=out[10]);
    And(a=a[11], b=b[11], out=out[11]);
    And(a=a[12], b=b[12], out=out[12]);
    And(a=a[13], b=b[13], out=out[13]);
    And(a=a[14], b=b[14], out=out[14]);
    And(a=a[15], b=b[15], out=out[15]);
}
```

## Implementando el Or16
```HDL
CHIP Or16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    // Put your code here:
    Or(a=a[0], b=b[0], out=out[0]);
    Or(a=a[1], b=b[1], out=out[1]);
    Or(a=a[2], b=b[2], out=out[2]);
    Or(a=a[3], b=b[3], out=out[3]);
    Or(a=a[4], b=b[4], out=out[4]);
    Or(a=a[5], b=b[5], out=out[5]);
    Or(a=a[6], b=b[6], out=out[6]);
    Or(a=a[7], b=b[7], out=out[7]);
    Or(a=a[8], b=b[8], out=out[8]);
    Or(a=a[9], b=b[9], out=out[9]);
    Or(a=a[10], b=b[10], out=out[10]);
    Or(a=a[11], b=b[11], out=out[11]);
    Or(a=a[12], b=b[12], out=out[12]);
    Or(a=a[13], b=b[13], out=out[13]);
    Or(a=a[14], b=b[14], out=out[14]);
    Or(a=a[15], b=b[15], out=out[15]);    
}
```

## Implementando el Mux16
```HDL
CHIP Mux16 {
    IN a[16], b[16], sel;
    OUT out[16];

    PARTS:
    // Put your code here:
    Mux(a=a[0], b=b[0], sel=sel, out=out[0]);
    Mux(a=a[1], b=b[1], sel=sel, out=out[1]);
    Mux(a=a[2], b=b[2], sel=sel, out=out[2]);
    Mux(a=a[3], b=b[3], sel=sel, out=out[3]);
    Mux(a=a[4], b=b[4], sel=sel, out=out[4]);
    Mux(a=a[5], b=b[5], sel=sel, out=out[5]);
    Mux(a=a[6], b=b[6], sel=sel, out=out[6]);
    Mux(a=a[7], b=b[7], sel=sel, out=out[7]);
    Mux(a=a[8], b=b[8], sel=sel, out=out[8]);
    Mux(a=a[9], b=b[9], sel=sel, out=out[9]);
    Mux(a=a[10], b=b[10], sel=sel, out=out[10]);
    Mux(a=a[11], b=b[11], sel=sel, out=out[11]);
    Mux(a=a[12], b=b[12], sel=sel, out=out[12]);
    Mux(a=a[13], b=b[13], sel=sel, out=out[13]);
    Mux(a=a[14], b=b[14], sel=sel, out=out[14]);
    Mux(a=a[15], b=b[15], sel=sel, out=out[15]);
}
```


# ¿Que es un chip multi-way?
La interpretación que he dado a la palabra way es la de autopista, via o carril, basado en eso, un chip multi-way es un chip que es capaz de recibir mas entradas de las que su implementacion estandar lo permite, por ejemplo el **Or** estandar tiene 2 entradas y una salida, en cambio un **Or8Way** tendría 8 entradas y mantendría la única salida.

Cabe resaltar que es posible hacer un chip m-way y n-bit al mismo tiempo, por ejemplo un Or8Way16 sería un chip que tiene ocho entradas *(carriles)* donde cada entrada tiene capacidad para que *"transiten"* por ella 8 bits simultaneamente.

A diferencia de los chips multi-bits los chips multi-ways no son todos tan sencillos, por lo que dare explicaciones en los casos que se amerite.

## Implementando el Or8Way
![Or8Way](https://cdn.rawgit.com/jorovipe97/computer_science_code/master/projects_resources/01/or8way.jpg)

En este caso la implementación es sencilla puesto que resulta unicamente de encadenar Or's
```HDL
CHIP Or8Way {
    IN in[8];
    OUT out;

    PARTS:
    // Put your code here:
    Or(a=in[0], b=in[1], out=or1);
    Or(a=or1, b=in[2], out=or2);
    Or(a=or2, b=in[3], out=or3);
    Or(a=or3, b=in[4], out=or4);
    Or(a=or4, b=in[5], out=or5);
    Or(a=or5, b=in[6], out=or6);
    Or(a=or6, b=in[7], out=out);

}
```

# Implementando el Mux4Way16
![Mux4way16](https://cdn.rawgit.com/jorovipe97/computer_science_code/master/projects_resources/01/mux4way16_1.jpg)

Este chip fue implementado usando 3 Mux16, la función de los 2 primeros multiplexores es recibir las 4 entradas, cada uno se encarga de  2 de ellas, ademas se ve que se usa el bit ubicado en la posicion 0 del selector para controlar el selector del mux1 y el otro bit se usa para controlar el selector del mux2.

En este punto hay un problema por resolver en el chip:
> ¡Tenemos dos salidas y solo debe haber una!

Para solucionar este problema se pensó en una función de dos entradas cuya salida fuera **true** en 2 casos y **false** en 2 casos para que funcionara como un selector *"inteligente que supiera cuando escoger mux1 y cuando mux2"*, asi se llego a la función Xor que devuelve **true** solo cuando sus entradas tienen valores logicos diferentes, se pudo llegar al circuito que se muestra en la imagen anterior cuando se observó que las dos veces que el **multiplexor selector** (muxsel) vale **true** la salida del **mux2** es diferente y las dos veces que la salida del **muxsel** vale **false** la salida del **mux1** tambíen es diferente.

Asi ya solo era cuestion de escribir el chip en HDL:
```HDL
CHIP Mux4Way16 {
    IN a[16], b[16], c[16], d[16], sel[2];
    OUT out[16];

    PARTS:
    // Put your code here:
    Mux16(a=a, b=d, sel=sel[0], out=mux1);
    Mux16(a=b, b=c, sel=sel[1], out=mux2);

    Xor(a=sel[0], b=sel[1], out=muxsel);

    Mux16(a=mux1, b=mux2, sel=muxsel, out=out);
}
```

Cabe decir que esta implementación muestra ciertas ventajas pero tambien tiene algunas desventajas, entre las ventajas encontramos que requiere de muy pocos chips lo que la hace una solución elegante, ademas con esta implementación no es necesario crear otros *"chips auxiliares"*, entre sus desventajas se encuentra que hay que conectar las entradas del **Mux4Way16** de una forma *"desordenada"* en los dos primeros mux, esto debido a que si no lo hacemos asi cuando el selector sea 00 no siempre devolvera a, o cuando sea 01 tampoco devolvera b sino otro valor como por ejemplo d, esta desventaja resulta en un mayor tiempo de construcción para el chip puesto que se debe cambiar de una forma manual el orden en el que se hace el cableado en los multiplexores internos.

Dado que tanto el **Mux8Way16** como el **DMux4Way** y el **DMux8Way** fueron implementados siguiendo esta lógica al ver su implementación interna se observa el mismo *"desorden aparente"*.

## Implementando el Mux8Way16
Como se digo anteriormente para este chip se siguió la misma lógica que la que se uso para el **Mux4Way** sin embargo aqui la conexión lucía mas confusa, por lo que fue necesario abstraerse de una explicación del circuito y mas bien los esfuerzos fueron dedicados a encontrar un patron para lograr llegar a una solución mas general.

En este caso se usaron dos **Mux4Way16** que sirvieron de procesadores de las 8 entradas, donde cada uno se encargaba de 4 y un Mux16 que sirvió de multiplexor selector.
![mux8way16_2](https://cdn.rawgit.com/jorovipe97/computer_science_code/29fd2f65/projects_resources/01/mux8way16_2.jpg)

En esta ocasión el **muxsel** no podia ser un **Xor** de dos entradas porque era necesario procesar los 3 bits del selector, por lo tanto se diseñó un **Xor3Way** y su salida se uso para controlar la salida a escoger por el **Mux16**, a continuacipon una tabla de verdad donde se muestra el funcionamiento de este circuito y la salida del **muxsel**.
![Mux8way16_1](https://cdn.rawgit.com/jorovipe97/computer_science_code/29fd2f65/projects_resources/01/mux8way16_1.jpg)

Cabe decir que aunque toricamente las salidas estaban ordenadas, fue necesario conectar las entradas del **Mux8Way16** de una forma *"desordenada"* a las entradas de los 2 **Mux4Way16**, a continuación se muestra la implementación final en HDL.
```HDL
CHIP Mux8Way16 {
    IN a[16], b[16], c[16], d[16],
       e[16], f[16], g[16], h[16],
       sel[3];
    OUT out[16];

    PARTS:
    // Put your code here:
    // Final 4mux selector
    Xor(a=sel[0], b=sel[1], out=mux4waysel1-1);
    Xor(a=mux4waysel1-1, b=sel[2], out=mux4waysel1-2);
    /*
	En HDL el signo - no se interpreta como un signo menos sino como 
	un guion, por tanto una variable puede tener dicho
	signo ej: foo1-2, en cambio el raya piso es un operador
	que no es valido, ej: foo1_2, es un nombre de pin no valido

	otro asunto a tener en cuenta es que si tengo un pin de 3 bits
	y quiere seleccionar los dos primeros puedo usar la siguiente sintaxis:
	x[0..1]
	y para seleccionar los dos ultimos podria hacer
	x[1..2]
	Esto es sintazis valida en HDL
    */

    // sel[0..1] es sintaxis valida?
    Mux4Way16(a=a, b=d, c=f, d=g, sel=sel[0..1], out=mux4way16-1);
	Mux4Way16(a=e, b=b, c=c, d=h, sel=sel[1..2], out=mux4way16-2);

	Mux16(a=mux4way16-1, b=mux4way16-2, sel=mux4waysel1-2, out=out);
}
```
Como se puede ver este enfoque tiene la ventaja de que su implementación final requiere de muy pocos chips, sacrificando el entendimiento intuitivo del circuito.

## Implementando el DMux4Way
La implementación que se realizó basa su funcionamiento en una idea parecida a la utilizada en la implementación del **Mux4Way16** pero en este caso el problema es el inverso, en un principio se pensó que la técnica de la representación canónica podia derivar en una construcción mas sencilla del chip, y aunque en efecto resulto mas sencilla de entender, el numero de chips necesarios para su implementación crecia de forma cuadratica dependiendo del numero de *"ways"* que requiriese el chip, por lo tanto esta idea fue abandonada y se retornó a la idea originalmente planteada.
![DMux4Way circuit](https://cdn.rawgit.com/jorovipe97/computer_science_code/master/projects_resources/01/dmux4way_1.jpg)

En este caso hay un **DMux16** que recibe la señal de entrada y basado en **xor1** (que depende de los bits del selector) decide si enviar la señal por **demuxsela** o **demuxselb**, luego estos **DMux16** hacen su trabajo y ocurre la magia:
```HDL
CHIP DMux4Way {
    IN in, sel[2];
    OUT a, b, c, d;

    PARTS:
    // Put your code here:
    Xor(a=sel[1], b=sel[0], out=xor1);
    DMux(in=in, sel=xor1, a=dmuxsela, b=dmuxselb);

    DMux(in=dmuxsela, sel=sel[1], a=a, b=d);
    DMux(in=dmuxselb, sel=sel[0], a=c, b=b);
}
```
Aqui es de nuevo importante recordar la desventaja subyacente en esta idea y es el *"desorden aparente en el cableado"*.

## Implementando el DMux8Way
En este punto ya se ha explicado la misma idea repetidas veces asi que intentare no repetir lo mismo, en esta ocasión similar a como en la del **Mux8Way16** se decidió usar 2 **DMux4Way**, un **DMux16** y un **Xor** de 3 entradas para el manejar el selector.
![DMux8Way](https://cdn.rawgit.com/jorovipe97/computer_science_code/b0811151/projects_resources/01/dmux8way_1.jpg)
```HDL
CHIP DMux8Way {
    IN in, sel[3];
    OUT a, b, c, d, e, f, g, h;

    PARTS:
    // Put your code here:
    Xor(a=sel[2], b=sel[1], out=xor1);
    Xor(a=xor1, b=sel[0], out=xor2);
    DMux(in=in, sel=xor2, a=dmuxa, b=dmuxb);

    DMux4Way(in=dmuxa, sel=sel[1..2], a=a, b=d, c=f, d=g);
    DMux4Way(in=dmuxb, sel=sel[0..1], a=e, b=b, c=c, d=h);

}
```


# Lista de chequeo y auto evaluación
[Lista de chequeo](https://docs.google.com/spreadsheets/d/1X6-rO-6J3cyO9a6pEXFOBlY33oGKyIMm0gvI9SM1FYo/edit?usp=sharing)

[Auto evaluación](https://docs.google.com/spreadsheets/d/1iQwefnnzveo2K_xAvspReNPAY6KGPqGVXmIErIpdYyM/edit?usp=sharing)
