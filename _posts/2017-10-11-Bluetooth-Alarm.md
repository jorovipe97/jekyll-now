---
layout: post
title: Primeros pasos con Bluetooth Low Energy
published: false
---

En esta ocación vamos a realizar una alarma en una aplicación Android que mediante Bluetooth Low Energy encendera un led conectado a un dispositivo wearable, con esto queremos lograr una primera introducción practica a Bluetooth Low Energy.

Para este tutorial nececitaras los siguientes materiales:

# Materiales

- 1 Simblee (RFD77203 Simblee 29-pin GPIO Breakout)
- 1 Simblee interfaz USB (RFD22121 USB Programming Shield)
- 1 Placa RGB led/button (RFD22122 RGB LED/Button Shield)
- 1 Celular Android con version 4.1 o superior (Por compatibilidad con Bluetooth Low Energy)

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
Imaginemos que tenemos un **despertador de mesa** que podemos configurar inalambricamente desde una aplicación de smartphone, supongamos que dicho despertador usa un dispositivo Bluetooth Low Energy (BLE) para lograrlo, ¿como sabe el smartphone que cerca de el hay un reloj queriendo comunicarce mediante BLE? bien, resulta que para saberlo, el dispositivo BLE de nuestro despertador hipotetico esta gritandole al mundo cada cierta cantidad de tiempo que el esta ahi, listo a que cualquiera que sea capaz de entenderlo se conecte a el, cuando el dispositivo BLE del despertador se encuentra en este rol de publicista desesperado se dice que esta en el rol GAP (Generic Access Profile) y cuando alguien se conecta al despertador el BLE pasa al rol GATT (Generic Attribute Profile).

Ademas existen los Services, Characteristics y Descriptors, que juegan un papel muy importante en el BLE cuando estamos en el rol GATT, pero ¿que son estas tres palabras tan raras?

![](https://imgur.com/AMs7xuN.gif)

## Services
Son un grupo de caracteristicas que cumplen una función especifica.

La especificación de Bluetooth Low Energy define algunos servicios como (<a href="https://www.bluetooth.com/specifications/gatt/services" target="_blank">GATT Services</a>) **Heart Rate**, **Blood Pressure**, **Environmental Sensing**, ademas tambien podrias crear un servicio totalmente nuevo como: **Detector de ronquidos**, cada servicio es identificado por un numero unico o UUID, por ejemplo el dispositivo BLE de nuestro despertador podria tener implementado el <a href="https://www.bluetooth.com/specifications/gatt/viewer?attributeXmlFile=org.bluetooth.service.current_time.xml" target="_blank">**Current Time**</a> service.

Un dispositivo bluetooth Low Energy generalmente puede tener mas de un servicio implementado.

## Characteristics
Es un valor de algun dato, como por ejemplo **Heart Rate Measurement** que se usa para enviar un ritmo cardiaco, **Body Sensor Location** usada para describir la ubicación prevista en la que se pone un dispositivo.

## Descriptors
Provee información adicional acerca de una caracteristica, un descriptor muy comun es el **Client Characteristic Configuration Descriptor** que se puede usar para activar las notificaciones/indicaciones en el cliente, asi cada vez que el valor de una characteristic cambie el server le envia el nuevo valor al cliente.

Cabe indicar que tanto los Services como las Characteristics y Descriptors son identificados con un ID unico (UUID) que es determinado por el **Bluetooth Special Interest Group**, sin embargo no estamos limitados a los UUID que ellos definieron ya que podemos crear los propios sin ningun problema.

# Los componentes del despertador
Te quiero mostrar ahora mediante un sencillo diagrama los componentes que conforman el Despertador prototipo que hemos usado:

![](https://imgur.com/dQ3WmtV.gif)

El simblee juega un papel principal, en nuestro prototipo ya que es el dispositivo que cuenta con un radio Bluetooth Low Energy y es el corazón del hardware de nuestro Despertador, la placa RGB se encargara de encender un LED cuando la alrma sea disparada, por cuestiones de disponibilidad no usaremos un dispositivo que haga ruido.

Por otra parte el conector USB simblee se usará solo para cargar el codigo al Simblee y como fuente de alimentación de voltaje.

# Comunicando el Hardware del despertador con la app android
Es necesario ahora definir un protocolo de comunicación que nos permitirá controlar el despertador desde la aplicación android, ademas tambien debemos definir las responsabilidades de cada elemento en todo el sistema

## Responsabilidades de la app android
La aplicación android se encargara de:
- Permitir al usuario configurar la hora y fecha de la alarma
- Guardar la fecha y hora de la alarma
- Encender el celular y enviar un mensaje al Simblee para que se encienda el LED del despertador cuando sea el momento de disparar la alarma.
- Escuchar si el usuario presiono el boton del RGB shield para apagar la alarma.

## Responsabilidades del Hardware del despertador (Simblee)
El simblee sera responsable de:
- Escuchar si la aplicación android envio el mensaje de encender la alarma.
- Decirle a la aplicación android si el usuario presiono el boton del RGB Led shield.
- Encender el LED del RGB led shield cuando la aplicación android envie el mensaje de encender.

## Protocolos de comunicación

- Cuando la aplicacion le envia un 0x01 al Simblee, este ultimo hara que el led RGB empiece a prender y apagar hasta que el usuario presione el boton del RGB shield.
- Cuando la aplicación le envia un 0x00 al Simblee este apagara el led RGB si este se encuentra encendido.
- Cuando el usuario presione el botton del RGB shield se enviara un 0x00 a la aplicación android.

Dicho lo anterior la aplicación que vamos a desarrollar consistira basicamente en programar cuando se debe enviar un 0x01 al Simblee mediante la conexion Bluetooth Low Energy, siguiendo este enfoque el protocolo de comunicación es muy sencillo, ten en cuenta que pudimos haber decidido que el simblee tuviera la logica necesaria para recibir un tiempo en el cual activarse pero no lo haremos de esa forma para mantener el protocolo lo mas sencillo posible.

# Programando el Simblee



# Programando la aplicacion Android
## Explicacion funcionamiento
Cundo terminemos la aplicación android tendremos algo como esto:

![](https://imgur.com/tvgxrrl.gif)

La aplicación tiene un boton de conectar y otro de desconectar que nos permitiran establecer una conexion BluetoothLE con el Simblee, cuando la aplicación se abre automaticamente se conecta pero en caso de que falle la conexion se puede intentar hacer una conexion manual usando el boton de conectar, luego pusimos 2 botones uno para configurar la fecha de la alarma y otro para configurar la hora de la alarma, la fecha de la alarma por defecto sera la fecha del dia de hoy asi que no es obligatorio seleccionar una fecha, en cambio, la hora de la alarma si debe ser seleccionada para poder setear la alarma, como habiamos dicho en la seccion de **protocolo** cuando esta fecha y hora lleguen la aplicación se encargara de enviar un 0x01 al Simblee, la anterior es la función fundamental de la aplicación.

DatePicker dialog

![](https://imgur.com/SkFQvkd.gif)

TimePicker dialog

![](https://imgur.com/27NgIGc.gif)

Una vez hemos seleccionado una hora a la cual queremos activar nuestra alarma el boton **set alarm** se activa para que pueda ser presionado por el usuario.

![](https://imgur.com/LaHvLhy.gif)

Y por último cuando el usuario presione el boton **set alarm** se usarán los servicios de alarmas del sistema operativo para encender la aplicación cuando la hora de sonar la alarma suene, cuando la aplicación es abierta desde los servicios de alarmas envia automaticamente un 0x01 al Simblee, logrando de este modo que nuestro preciado despertador (Luz LED) se pueda encender y empiece a parpadear.

## El codigo
Lo primero que debemos hacer es en el AndroidManifiest solicitar permisos para acceder al Blutooth y para encender el celular cuando este en modo dormido.

```xml

<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.WAKE_LOCK" />

```

Ademas tambien seria buena idea hacer que sea obligatorio para el dispositible tener compatibilidad con Bluetooth Low Energy
```xml

<uses-feature android:name="android.hardware.bluetooth_le" android:required="true" />

```

Una vez hemos terminado con los permisos, vamos a trabajar la interfaz de usuario, nuestra aplicación tendra solo una activity (MainActivity), el XML del Layout luce parecido a lo siguiente:

```xml
<?xml version="1.0" encoding="utf-8"?>

<!--
Haremos una interfaz usando LinearLayouts porque resulta intuitivo de entender

El gravity=center nos permite centrar los elementos hijos del ViewGroup vertical y horizontalmente.

-->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    tools:context="com.romualdo.ble.gattclient.MainActivity">

    <LinearLayout
        android:gravity="center"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingBottom="@dimen/h6"
        android:orientation="horizontal">

        <Button
            android:id="@+id/buttonConnect"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Connect"
            android:onClick="startClient"
            android:enabled="true"/>

        <Button
            android:id="@+id/buttonDisconnect"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Disconnect"
            android:onClick="disconnect"
            android:enabled="false"/>

    </LinearLayout>

    <View
        android:layout_width="80dp"
        android:layout_height="10dp"
        android:background="@color/clouds" />

    <LinearLayout
        android:gravity="center"
        android:paddingTop="@dimen/h6"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <!--
        showFatePickerDialog() es un metodo que hemos creado en el MainActivity y nos permite
        abrir un DatePickerDialog.
        -->
        <Button
            android:id="@+id/btnSetDate"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Fecha alarma"
            android:textSize="@dimen/text_default"
            android:onClick="showDatePickerDialog"/>

        <Button
            android:id="@+id/btnSetTime"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hora alarma"
            android:textSize="@dimen/h3"
            android:onClick="showTimePickerDialog"/>


    </LinearLayout>


    <LinearLayout
        android:gravity="center"
        android:paddingTop="22sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <TextView
            android:textStyle="bold"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="@dimen/h6"
            android:text="La alarma sonará el"/>

    </LinearLayout>


    <LinearLayout
        android:gravity="center"
        android:paddingTop="5sp"
        android:paddingBottom="15sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <TextView
            android:id="@+id/textDate"
            android:textAlignment="center"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="year/mo/da"
            android:textSize="@dimen/h5"/>

        <TextView
            android:textAlignment="center"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="a las"
            android:paddingLeft="@dimen/h6"
            android:paddingRight="@dimen/h6"
            android:textSize="@dimen/h6"/>

        <TextView
            android:id="@+id/textClock"
            android:textAlignment="center"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="hh:mm"
            android:textSize="@dimen/h5"/>

    </LinearLayout>

    <LinearLayout
        android:gravity="center"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">


        <TextView
            android:id="@+id/btnStatus"
            android:textStyle="italic"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Button up"
            android:textSize="@dimen/text_small" />

    </LinearLayout>

    <LinearLayout
        android:gravity="center"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <!--
        enabled="false" porque por defecto no queremos que el usuario pueda
        setear la alarma hasta que no halla seleccionado una hora, mediante codigo
        se cambia a enabled="true".
        -->
        <Button
            android:id="@+id/btnSetAlarm"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:text="Set Alarm"
            android:onClick="setAlarm"
            android:enabled="false"/>

        <Button
            android:id="@+id/btnOff"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Off Alarm"
            android:textSize="18sp"
            android:enabled="false"/>

    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">



    </LinearLayout>

</LinearLayout>

```

**Hechale un vistazo al codigo fuente completo** [](https://github.com/jorovipe97/AlarmBLE)


# Referencias
<a href="http://ticketmastermobilestudio.com/blog/android-bluetooth-low-energy-tutorial" target="_blank">Android BLE tutorial</a>
