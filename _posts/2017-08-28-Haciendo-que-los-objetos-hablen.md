---
layout: post
title: Lampara RGB controlada desde aplicación Andoid
published: false
---

# Consideraciones a tener en cuenta a la hora de seleccionar un LED
¿Qué consideraciones debe tener en cuenta para seleccionar un LED?
A primera vista puede parecer que un LED es el dispositivo mas facíl de comprar, solo es cuestion de ir a la tienda y pedir un LED, el vendedor escojera un LED entre los miles de LEDs "iguales" que tiene en la bodega y nos lo entregara.

Pero esto no es correcto, un LED puede brillar mas que otro, su package puede ser mas grande o pequeño en incluso el angulo de vision de la luz que emite puede variar, a continuación se explicaran con un poco mas de detalle algunos factores a tener en cuenta en un LED para seleccionarlo.

NOTA: Todas las propiedades de un LED especifico estan detalladas en su respectivo *datasheet*

## El brillo
¿Que tan brillante es un LED? esto generalmente esta en el *datasheet* y se expresa en candelas (cd), la cual es en terminos generales una unidad para medir la intensidad luminica de una fuente de luz.
![](http://imgur.com/0Ylb6mV.gif)

Por ejemplo, con el LED que estamos usando de ejemplo, se ve en su datasheet, que a 20mA tiene una intensidad luminica de 250 mcd (mili candelas), aunque debido a que nada es perfecto en el mundo real, el fabricante nos advierte que es posible que el LED que le compremos tenga una intensidad luminica de entre 180 cd a 250 cd.

## Tipo de brillo
Puede ser *diffuse* o *clear*, si tenemos un Diffused LED tendremos un LED que emite luz de una forma suave, uniforme y se puede ver desde casi cualquier angulo estos son generalmente buenos para labores de indicacion, en cambio con un Clear LED la luz es directa y muy fuerte, por lo tanto su luz solo se ve desde unos angulos muy especificos, en Colombia tambien son llamados *"LED de chorro"* porque se asemejan a una mangera a la que le tapamos casi todo el orificio de salida, haciendo que el agua salga con mucha presion y muy direccionada, estos LEDs son buenos para labores de iluminación.

Si hacemos una analogia con el agua los Diffused LEDs son como aspersores (de agua) mientras que los clear LED se asemejan mas a un limpiador (de agua) a alta presion.

## El color
Este factor es obvio y es quizas uno de los que intuitivamente tenemos en cuenta a la hora de seleccionar LEDs en la tienda de electronicos, lo unico complicado es que en el datasheet de un LED el color esta dado en una <a href="http://lsrtools.1apps.com/wavetorgb/index.asp?wavelength=640" target="_blank">longitud de onda</a> electromagnetica, por ejemplo la del LED que estamos analizando es de 640 la cual nuestro ojo percibe como un color rojo.

![](http://imgur.com/SWFC4nH.gif)


## Package dimensions
![](http://imgur.com/m3pfTqq.gif)

NOTA: Unidades estan en milimetros, en parentecis esta la misma unidad pero en pulgadas.

Este se refiere al tamaño de la capsula que protege el interior del LED, generalmente hay 3 tamaños muy usados, el de 3mm, el de 5mm y el de 10mm, generalmente el de 10mm es un diodo de 5mm con una capsula mas grande, por tanto se puede llegar a ser incluso menos brillante que un led de 5mm, esto deja en evidencia que no necesariamente es cierto que a mas grande la capsulta protectora del LED mas brillo obtendremos.


## Angulo de vision de la luz
![](http://imgur.com/om9jSid.gif)

No todos los LEDs se ven igual de brillantes desde todos los angulos, esto es especialmente notorio en los **clear LED** que solo se ven super brillantes si los miramos desde unos puntos, y se ven mas opacos si los vemos desde otros puntos, es decir el brillo percibido depende del angulo de vision desde el que se ve el LED.

## Forward voltage
Este se refiere a la caida de voltaje que produce el LED, en la datasheet que estamos viendo cuando la corriente es de 20 mA.

# Posibles usos de los LED en aplicaciones interactivas
Existen muchos (infinitos), pero mencionare solo algunos:
1. Como ojos en un mecatronico.
2. Como indicador de fuerza en un juego de fuerza (poniendo muchos leds en linea y prendiendo solo una cantidad relativa a la fuerza del golpe)
3. Para indicar que un boton fue precioniado.

# Circuito de acondicionamiento de un LED
¿Cómo es el circuito de acondicionamiento de un LED a un microcontrolador?. EXPLICAR y mostrar un ejemplo donde se calcule el circuito de acondicionamiento. Seleccione un LED (de algún fabricante) y busque la hoja de datos del mismo. Utilice los datos de la hoja de datos para realizar los cálculos. Señale exactamente qué parte de la hoja de datos consultó para extraer los datos del LED.

Es un circuito basico que consiste en conectar en serie una resistencia a un LED para garantizar una corriente (I) maxima en el LED.

Pero como podemos calcular una resistencia que permita que nuestro LED brille lo suficiente.

> poner calculos aqui.

## Riesgos de conectar un LED directamente a un microcontrolador sin un circuito de acondicionamiento
¿Por qué no se debería conectar un LED directamente a un microcontrolador sin un circuito de acondicionamiento?

# Variando el brillo de un LED utilizando una resistencia variable
¿Cómo es posible variar el brillo de un LED utilizando resistencias variables?

# Variando el brillo de un LED utilizando una señal PWM
¿Cómo es posible variar el brillo o la intensidad de un LED utilizando una señal de PWM?

# Generando distintos colores utilizando señales de PWM
¿Cómo es posible generar diferentes colores utilizando señales de PWM? 
Aqui se refieren a un diodo RGB? o se refieren a un diodo emisor de luz tipico.

# Implicacion de los diodos RGB de anodo comun y catodo comun en el programa del microcontrolador.
¿El programa del microcontrolador es igual sin importar que el LED sea de cátodo o ánodo común? Explique claramente.

# Referencias
<a href="https://learn.adafruit.com/all-about-leds/" target="_blank">All about LEDs</a>

<a href="https://electronics.stackexchange.com/questions/10962/what-is-forward-and-reverse-voltage-when-working-with-diodes" target="_blank">What is “forward” and “reverse” voltage when working with diodes?</a>

<a href="https://en.wikipedia.org/wiki/Light-emitting_diode#Colors_and_materials" target="_blank">Color de los LED</a>

<a href="https://cdn-shop.adafruit.com/datasheets/WP7113SRD-D.pdf" target="_blank">LED datasheet example</a>

<a href="https://es.wikipedia.org/wiki/Candela" target="_blank">Candela, wikipedia</a>
