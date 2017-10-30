---
layout: post
title: Interrupciones en Arduino
published: false
---

En esta ocacion hablaremos de las interrupciones en Arduino las cuales podemos interpretar como un multithreading de bajo nivel.

Para practicar los conceptos aprendidos desarrollaremos una calculadora basica en Arduino que se controlará mediante mensajes seriales, dicha calculadora esperará por 10 segundos los operandos de la operación que halla sido seleccionada en caso de no haber sido suministrados por el usuario, se lanzará una interrupción que reiniciara la calculadora para que se vuelva a seleccionar otra operación.

# Flujo del programa:

![](https://imgur.com/QFRk0Wi.gif)

Es bueno hacer notar que nuestra implementación solo aceptará operandos >= 0.

Pero antes empezar aclaremos algunos conceptos que serán necesarios para implementar nuestra calculadora usando interrupciones.

# Interrupciones
¿Que son las interrupciones? aunque no es 100% preciso podemos entenderlas de la siguiente forma:

> Las interrupciones son a Arduino lo que el Multithreading es a un computador convencional

Vamos a ver en mas detalle esta interpretación: Supongamos que queremos encender y apagar un LED cada segundo, podriamos simplemente usar un delay de 1 segundo en el loop() despues de encender y apagar el led.

```c++
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT); // Ocasionally LED_BUILTIN is connected to Digital Pin 13
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
}
```

Este enfoque no estaria mal, pero tendria alguos inconvenientes si quisieramos hacer algo mas complejo, ¿que pasaria por ejemplo si quisieramos realizar unas operaciones muy importantes mientras el led prende y apaga?

```c++
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
  importantOperations();             // Execute important operations after two seconds        
}
```

La respuesta es que dichas operaciones importantes se estarian ejecutando solo cada dos segundos, es decir, para que el Arduino pueda ejectuar estas operaciones debe esperar a que el LED halla prendido y apagado, en muchas circunstancias (incluida la calculadora que implementaremos) este enfoque simplemente no sirve.

¿Como podriamos entonces ejecutar las operaciones importantes sin esperar a que el LED halla prendido y apagado? respuesta corta: usando interrupciones, respuesta larga:

```c++
#include <TimerOne.h>

const int led = 13;  // the pin with a LED

void setup(void)
{
  pinMode(led, OUTPUT);
  Timer1.initialize(1000000);
  Timer1.attachInterrupt(blinkLED); // blinkLED to run every 1 seconds
}

// The interrupt will blink the LED, and keep
// track of how many times it has blinked.
int ledState = LOW;

void blinkLED(void)
{
  if (ledState == LOW) {
    ledState = HIGH;
  } else {
    ledState = LOW;
  }
  digitalWrite(led, ledState);
}

// The main program will print the blink count
// to the Arduino Serial Monitor
void loop(void)
{
  importantOperations();
}
```

Si miras con detenimiento veras que en el ejemplo anterior no hay absolutamente nada de codigo relacionado con el blink led en el loop(), en cambio ahora se ejecutan las operaciones importantes durante la mayoria del tiempo y se ejecuta el metodo encargado de encender y apagar el led cada millon de microsegundos (1 segundo).



# Referencias
[TimerOne Arduino library](https://www.pjrc.com/teensy/td_libs_TimerOne.html)
[Gammon interrupt discussion](https://www.gammon.com.au/forum/?id=11488)
