---
layout: post
title: Primeros pasos con Bluetooth Low Energy
---

En esta ocación vamos a realizar una alarma en una aplicación Android que mediante Bluetooth Low Energy encendera un led conectado a un dispositivo wearable, con esto queremos lograr una primera introducción practica a Bluetooth Low Energy.

# Aplicación resultado final
[![](https://imgur.com/Og8VHzZ.gif)](https://play.google.com/store/apps/details?id=com.romualdo.ble.gattclient&hl=es)

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
## Explicacion funcionamiento
El simblee solo debe encargarse de 3 asuntos.

1. Enviar un 1 para encender la alarma
2. Enviar un 0 para apagar la alarma
3. La alarma se puede apagar presionando el bton A del semblee o enviando un cero desde la aplicacion

## El codigo

El setup() y el loop()

```c++
void setup()
{
  // led turned on/off from the iPhone app
  pinMode(led, OUTPUT);

  Serial.begin(9600);
  
  // button press will be shown on the iPhone app)
  pinMode(button, INPUT);

  // this is the data we want to appear in the advertisement
  // (if the deviceName and advertisementData are too long to fix into the 31 byte
  // ble advertisement packet, then the advertisementData is truncated first down to
  // a single byte, then it will truncate the deviceName)
  SimbleeBLE.advertisementData = "ledbtn";
  SimbleeBLE.deviceName = "RoJo";

  // start the BLE stack
  SimbleeBLE.begin();
}

void loop() {
  // Que pasa si el boton esta en LOW
  // El Simblee se mantendra en estado dormido
  delay_until_button(HIGH);
  buttonStatus = true;
  SimbleeBLE.send(1); // Por tanto este codigo no se ejecutara
  // Cuando el digitalRead(ButtonA) == true por primera vez, se enviara el valor anterior.


  /*
     Ahora el Simblee quedara dormido mientras se mantiene precionado el botonA
     Ya que digitalRead(ButtonA) es diferente de true
  */
  delay_until_button(LOW); // Cuando se deje de presionar el boton, se procedera a ejecutar la siguiente linea
  //buttonStatus = false;
  SimbleeBLE.send(0); // y por tanto se envia un 0 por BLE
  // And we come back to the first line of the loop() function
  // And consencuently Simblee keeps in ultra low power mode until buttonA will be pressed again.
}
```

Cuando el simblee recibe datos se llama este metodo, procedemos a preguntar si el dato que llegó fue un 01x01, en caso afirmativo pone en true unas variables flag que se permitiran encender el led.

En caso negativo se apaga el led.
```c++
void SimbleeBLE_onReceive(char *data, int len)
{
  // if the first byte is 0x01 or great than zero / on / true
  if (data[0])
  {
    canTurnOnAlarm = (byte) 1;
    buttonStatus = false;
    //digitalWrite(led, HIGH);
    //blinkLed(500);
  }
  else 
  {
    digitalWrite(led, LOW);
    canTurnOnAlarm = (byte) 0;
  }
}
```

En este metodo hemos escrito el blink, que recibe como argumento el tiempo en milisegundos que durara el led prendido y apagado en el blink.
```c++
void blinkLed(int ms)
{
  digitalWrite(led, HIGH);
  Simblee_ULPDelay(ms);
  digitalWrite(led, LOW);
  Simblee_ULPDelay(ms);
}
```

```c++
int delay_until_button(int state)
{
  // set button edge to wake up on
  if (state)
    Simblee_pinWake(button, HIGH); // Congifures pin button to wake up the device on a high signal
  else
    Simblee_pinWake(button, LOW); // Configures pin button to wake up the device on a LOW signal

  do 
  {
    // switch to lower power mode until a button edge wakes us up
    // The alarm keeps ringin as long as canTurnOnAlarm is equal to true and buttonA is not pressed.
    if (!buttonStatus && canTurnOnAlarm)
    {
        blinkLed(500);
    }
    else
    {
      Simblee_ULPDelay(1000);
    }
  }
  while (! debounce(state)); // Mantener dormido mientras debounce sea igual a false
  // Si el simblee se duerme porque las condiciones del do..while siguen ejecutandose?
  // que es exactamente lo que se duerme cuando el simblee se pone en modo ultra low power delay

  // debounce(state) va a dar false mientras digitalRead() != state
  // Cuando el programa inicia, queda bloqueado aca, hasta que halla una señal true en el buttonA
  /* Cuando ocurre la señal true, la condicion del do..while se hace false
     y se procede con la siguiente linea
  */

  // SIMBLEE framework stuffs
  // if multiple buttons were configured, this is how you would determine what woke you up
  if (Simblee_pinWoke(button))
  {
    // execute code here
    Simblee_resetPinWake(button);
  }
}
```

**Mira el codigo fuente completo en ** [el repositorio](https://github.com/jorovipe97/GattServerSimblee).

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

Una vez hemos terminado con los permisos, vamos a trabajar la interfaz de usuario, nuestra aplicación tendra solo una activity (MainActivity), el XML del Layout se ve parecido a lo siguiente:

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
        
        <!-- This TextView will be used to show to user the selected date of the alarm -->
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
        
        <!-- This TextView will be used to show to user the selected time of the alarm -->
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

Ahora vamos con la lógica de la aplicación, lo primero de lo que nos tenemos que asegurar es que cuando el usuario abre la aplicación tenga encendido el bluetooth, y en caso de no tenerlo solicitarle que lo encienda.

![](https://imgur.com/43qPJma.gif)

Para lograr este comportamiento, en el onCreate() del activity preguntamos si el Bluetooth esta encendido, y si no, abrimos una activity por defecto que le solicitara al usuario encender el Bluetooth.

```java
// Ensures Bluetooth is available on the device and it is enabled. If not,
// displays a dialog requesting user permission to enable Bluetooth.
if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
} else {
    // Si el dispositivo tiene bluetooth o el bluetooth esta encendido, hacer visible el boton de conectar y de desconectar de la interfaz de usuario.
    connectBtn.setVisibility(View.VISIBLE);
    disconectBtn.setVisibility(View.VISIBLE);
}
```

Cuando el usuario haya decidido permitir o no permitir encender el bluetooth se llamara el siguiente metodo de la MainActivity

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request we're responding to
    // just UI topics
    if (requestCode == REQUEST_ENABLE_BT) {
        if (resultCode == RESULT_OK) {
            Log.w(TAG, "Bluetooth enabled");
            Toast.makeText(this, "Bluetooth enabled", Toast.LENGTH_SHORT).show();
            connectBtn.setVisibility(View.VISIBLE);
            disconectBtn.setVisibility(View.VISIBLE);
        }
        else {
            // Si no se pudo enceder el bluetooth o el usuario no lo permitio, desactivar los botones de conectar y de desconectar.
            connectBtn.setVisibility(View.VISIBLE);
            disconectBtn.setVisibility(View.VISIBLE);
            connectBtn.setEnabled(false);
            disconectBtn.setEnabled(false);
            Toast.makeText(this, "Bluetooth not enabled, closing app...", Toast.LENGTH_SHORT).show();
            // TODO: Catch exceptions if bluetooth is not available on device.
        }
    }
}
```
Ademas debemos conectarnos mediante bluetooth al Simblee tan pronto la aplicación es abierta, para ello llamamos el metodo startClient() que hemos creado en el onCreate de la activity.

```java
public void startClient() {
    try {
        BluetoothDevice bluetoothDevice = mBluetoothAdapter.getRemoteDevice(MAC_ADDRESS);
        mBluetoothGatt = bluetoothDevice.connectGatt(this, false, mGattCallback);

        if (mBluetoothGatt == null) {
            Log.w(TAG, "Unable to create GATT client");
            Toast.makeText(this, "Cant connect to " + MAC_ADDRESS, Toast.LENGTH_SHORT).show();
            return;
        }
    }
    catch (Exception e) {
        Log.w(TAG, e.toString());
    }
}
```

¿Recuerdas que habiamos dicho que cada service, characteristic o descriptor esta identificad con un ID unico (UUID)? que bien, resulta que en nuestra aplicación hemos sacado manualmente estos datos usando una aplicación llamada **nRF Connect**

![](https://imgur.com/EKrhzpc.gif)

```java

public static final String MAC_ADDRESS = "CA:A5:4F:3A:A9:5C";
public static final UUID UUID_SERVICE = UUID.fromString("0000fe84-0000-1000-8000-00805f9b34fb");
public static final UUID UUID_CHARACTERISTIC_BUTTONSTATUS = UUID.fromString("2d30c082-f39f-4ce6-923f-3484ea480596");

public static final UUID UUID_CHARACTERISTIC_LED = UUID.fromString("2d30c083-f39f-4ce6-923f-3484ea480596");

// This is one of the most used descriptor: Client Characteristic Configuration Descriptor. 0x2902
public static final UUID UUID_DESCRIPTOR = UUID.fromString("00002902-0000-1000-8000-00805f9b34fb");
```

En este codigo es muy importante el mGattCallback que basicamente nos permitira ejecutar codigo cuando ocurran eventos como: 1. Conexion establecida, 2. Servicios descubiertos, etc.

```java
private BluetoothGattCallback mGattCallback = new BluetoothGattCallback() {

    private final String TAG = "mGattCallback";

    @Override
    public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {

        super.onConnectionStateChange(gatt, status, newState);
        Log.i(TAG, status + " " + newState);
        if (newState == BluetoothProfile.STATE_CONNECTED)
        {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    connectBtn.setEnabled(false);
                    disconectBtn.setEnabled(true);
                }
            });
            mBluetoothGatt.discoverServices();
        } else if (newState == BluetoothProfile.STATE_DISCONNECTED)
        {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    connectBtn.setEnabled(true);
                    disconectBtn.setEnabled(false);
                }
            });
        }
    }


    @Override
    public void onServicesDiscovered(BluetoothGatt gatt, int status) {
        //Log.i(TAG, "Service discovered");

        if (status == gatt.GATT_SUCCESS) {
            BluetoothGattService service = gatt.getService(UUID_SERVICE);
            if (service != null) {
                Log.i(TAG, "Service connected");
                BluetoothGattCharacteristic characteristic = service.getCharacteristic(UUID_CHARACTERISTIC_BUTTONSTATUS);
                if (characteristic != null) {
                    Log.i(TAG, "Characteristic connected");
                    gatt.setCharacteristicNotification(characteristic, true);
                    BluetoothGattDescriptor descriptor = characteristic.getDescriptor(UUID_DESCRIPTOR);
                    if (descriptor != null) {
                        // Los descriptors son muy importntes
                        // TODO: Continue studying about descriptors in BLE
                        descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
                        gatt.writeDescriptor(descriptor);
                        Log.i(TAG, "Descriptor sended");
                    }
                }

                BluetoothGattCharacteristic characteristicLed = service.getCharacteristic(UUID_CHARACTERISTIC_LED);
                if (characteristicLed != null) {

                    Runnable myRunnable = new Runnable() {
                        @Override
                        public void run() {
                            if (isAlarmFired) {
                                writeLedCharacteristic(true);
                            }
                        }
                    };
                    Handler mainHandler = new Handler(Looper.getMainLooper());
                    mainHandler.postDelayed(myRunnable, 1000); // This causes myRunnable executed after 1000 ms, this is necesary for work, in this case we dont use runOnUiThread() because we are not modifying ui components.

                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            if (isAlarmFired) {
                                btnOff.setEnabled(true);
                            }
                        }
                    });
                }
            }
        }
    }
/*
    @Override
    public void onCharacteristicRead(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {
        super.onCharacteristicRead(gatt, characteristic, status);
    }*/

    @Override
    public void onCharacteristicChanged(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic) {
        readBtnStateCharacteristic(characteristic);
    }

    private void readBtnStateCharacteristic(BluetoothGattCharacteristic characteristic) {
        if (UUID_CHARACTERISTIC_BUTTONSTATUS.equals(characteristic.getUuid())) {
            byte[] data = characteristic.getValue();
            //int state = Ints.fromByteArray(data);
            Log.i(TAG, data[0] + "");
            if (data[0] == 1) {

                Runnable r = new Runnable() {
                    @Override
                    public void run() {
                        WakeLocker.release();
                    }
                };

                Handler mainHandler = new Handler(Looper.getMainLooper());
                mainHandler.postDelayed(r, 2000);
                
                // In android callbacks are executed in another thread diferent to main thread, because of that, we should execute ui operations whitin a runnable in runOnUiThread method.
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        btnOff.setEnabled(false);
                        statusBtn.setText("Button Down");
                    }
                });
            } else if (data[0] == 0) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        statusBtn.setText("Button Up");
                    }
                });
            }
        }
    }
};
```


Para configurar el momento en el que se disparara la alarma usamos un DatePicker y un TimePicker para seleccionar la fecha y la hora respectivamente.

```xml
<Button
    android:id="@+id/btnSetTime"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hora alarma"
    android:textSize="@dimen/h3"
    android:onClick="showTimePickerDialog"/>
```

Lo que hace el boton **set time** es abrir un TimePickerDialog:

```java
public void showTimePickerDialog(View view) {
    DialogFragment fragment = new SelectTimeFragment();
    fragment.show(getSupportFragmentManager(), "TimePicker");
}
```

La clase SelectTimeFragment es una custom class que es hija de DialogFragment e implementa TimePickerDialog.OnTimeSetListener para poder saber cuando el usuario a presionado el boton OK del time picker.

```java
public class SelectTimeFragment extends DialogFragment implements TimePickerDialog.OnTimeSetListener {
    private OnFragmentInteractionListener mListener;

    public SelectTimeFragment() {
        // Required empty public constructor
    }
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }


    @Override
    public Dialog onCreateDialog(Bundle savedInstance) {
        Calendar c = Calendar.getInstance();
        int hour = c.get(Calendar.HOUR_OF_DAY);
        int minute = c.get(Calendar.MINUTE);
        
        // Por defecto al abrirse esta en la hora actual.
        return new TimePickerDialog(getActivity(), this, hour, minute, true);
    }

    // Called when user set a time
    @Override
    public void onTimeSet(TimePicker view, int hourOfDay, int minuteOfDay) {
        if (mListener != null) {
            // Toast.makeText(getActivity(), "Time seted", Toast.LENGTH_LONG).show();
            mListener.onPickerTimeSet(hourOfDay, minuteOfDay);
        }
    }


    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        if (context instanceof OnFragmentInteractionListener) {
            mListener = (OnFragmentInteractionListener) context;
        } else {
            throw new RuntimeException(context.toString()
                    + " must implement OnFragmentInteractionListener");
        }
    }

    @Override
    public void onDetach() {
        super.onDetach();
        mListener = null;
    }

    /**
     * This interface must be implemented by activities that contain this
     * fragment to allow an interaction in this fragment to be communicated
     * to the activity and potentially other fragments contained in that
     * activity.
     * <p>
     * See the Android Training lesson <a href=
     * "http://developer.android.com/training/basics/fragments/communicating.html"
     * >Communicating with Other Fragments</a> for more information.
     */
    public interface OnFragmentInteractionListener {
        void onPickerTimeSet(int hour, int minuts);
    }
}
```

Luego en el MainActivity implementamos la interfaz SelectTimeFragment.OnFragmentInteractionListener para poder ejecutar codigo justo en el momento en el que el usuario a seleccionado una hora.

```java
public class MainActivity extends AppCompatActivity  implements
        SelectTimeFragment.OnFragmentInteractionListener {
    
    // class code here
    // ...
    
    // Code to execute when user select a time from the TimePicker dialog box.
    public void onPickerTimeSet(int hour, int minuts) {
        isTimeSeted = true;
        if (isTimeSeted) {
            canSetAlarm = true;
        }

        // Se activa el boton set alarm.
        btnSetAlarm.setEnabled(canSetAlarm);

        // Se hace un formato manual de la fecha para que las horas/minutos menores a 10 aparezcan con un 0 antes.
        String strhour = hour+"";
        String strminute = minuts+"";

        if (hour < 9) {
            strhour = "0" + hour;
        }
        if (minuts < 9) {
            strminute = "0" + minuts;
        }

        // Guarda la hora y el minuto de la alarma en un field tipo Calendar que sera usado para pasarlo como argumento al AlarmManager para disparar la alarma posteriormente.
        alarmOnDate.set(Calendar.HOUR_OF_DAY, hour);
        alarmOnDate.set(Calendar.MINUTE, minuts);
        textClock.setText("");
        
        textClock.setText(strhour + ":" + strminute);
    }
    
}
```

El mismo enfoque se sigue para ejecutar codigo en el MainActivity cuando el usuario ha seleccionado una fecha: 

```java
public class MainActivity extends AppCompatActivity  implements
        SelectDateFragment.OnFragmentInteractionListener {
    
    // class code here
    // ...
    
    // Code to execute when user select a time from the TimePicker dialog box.
    public void onPickerDateSet(Calendar c) {
        // The date is optional, in case when user doesnt specify one, today date is used.
        isDateSeted = true;
        if (isTimeSeted) {
            canSetAlarm = true;
        }
        btnSetAlarm.setEnabled(canSetAlarm);
        
        // Saves in the field alarmOnDate the yar, month and day to shoot the alarm.
        // this field will be used later on the AlarmManager for shoot the alarm.
        alarmOnDate.set(c.get(Calendar.YEAR), c.get(Calendar.MONTH), c.get(Calendar.DATE));
        
        // Show in the user interface the date picked.
        textDate.setText(c.get(Calendar.DATE) + "/" + (c.get(Calendar.MONTH)+1) + "/" + c.get(Calendar.YEAR));
    }
    
}
```

Ten en cuenta que el boton **set alarm** no se activa hasta que el usuario haya seleccionado una hora con el TimePicker, una vez esto ocurra se llamara el siguiente metodo cuando se presione dicho boton.

```java

public void setAlarm(View view) {
    // When user clicks setAlarmButton, button get disabled inmediatelly.
    btnSetAlarm.setEnabled(false);

    // AlarmReceiver is an Android Broadcast Receiver that will have called if alarm get fired. 
    Intent intent = new Intent(this, AlarmReceiver.class);

    // If users has not specified the date of the alarm, app will use today date, so alarm date is optional
    if (!isDateSeted) {
        Calendar c = Calendar.getInstance(Locale.getDefault());
        alarmOnDate.set(c.get(Calendar.YEAR), c.get(Calendar.MONTH), c.get(Calendar.DATE));
    }

    // Creating a pending intent to be called when alarm get fired.
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, REQUEST_SET_ALARM, intent, PendingIntent.FLAG_CANCEL_CURRENT);
    
    // Passing the pending intent to AlarmManager for get fired when actualTimeInMillis - alarmOnDate.getTimeInMilles() <= 0
    alarmManager.set(AlarmManager.RTC_WAKEUP, alarmOnDate.getTimeInMillis(), pendingIntent);

}

```

Es importante tener en cuenta que la presición de la alarma es un parametro configurable, en nuestro caso no es tan precisa lo que hara que la alarma no se dispara justo a las 14:22 si no, muy probablemente unos cuantos segundos despues.

Cuando la alarma se llame desde el servicios de alarma del sistema operativo se llamara el siguiente codigo, el cual es el Broadcast Receiver que pasamos mediante la pending intent en el codigo anterior.

```java
public class AlarmReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.

        // This line ensures CPU is running as long as WakeLocker.release() dont get called.
        WakeLocker.acquire(context.getApplicationContext());
        
        // An intent for open the MainActivity from the BroadcastReceiver
        Intent i = new Intent(context.getApplicationContext(), MainActivity.class);
        i.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        // This extra later will have used for know if the activity was opened from Alarm manager or not.
        i.putExtra(MainActivity.EXTRA_IS_FROM_ALARM, true);
        
        // Start the activity specified in the intent i
        context.startActivity(i);

    }
}
```


**Hechale un vistazo al codigo fuente completo** [Link repositorio](https://github.com/jorovipe97/AlarmBLE)

# Notas finales
El enfoque que hemos usado para solucionar el problema tiene falencias porque hace que los dos dispositivos sean muy dependientes uno del otro tanto el simblee de la aplicación como viceversa, ¿que pasaria si al momento de disparar la alarma el celular no puede conectarse al simblee? nunca sonaria la alarma, ¿que pasaria si por accidente el usuario dejo el celular en el baño y por tanto fuera del radio de alcance del radio bluetooth? nunca sonaria la alarma, ¿que pasaria si al usuario se le apaga el celular porque olvido dejarlo cargando? repitan conmigo: nunca sonaria la alarma, dicho esto te propongo una version 2.0 en la que el sistema de tiempo y alarma no sea manejado por el celular sino por el simblee, de tal modo que la aplicación solo sea utilizada para decirle al simblee cuando debe prender la alarma, esto haria el protocolo de comunicación un poco mas complejo pero el sistema en general quedaria mucho mas robusto y seguro.

# Referencias
<a href="http://ticketmastermobilestudio.com/blog/android-bluetooth-low-energy-tutorial" target="_blank">Android BLE tutorial</a>

<a href="https://developer.android.com/guide/topics/connectivity/bluetooth-le.html" target="_blank">Android Developers, Bluetooth Low Energy</a>

<a href="http://nilhcem.com/android-things/bluetooth-low-energy" target="_blank">Nilhcem, Bluetooth Low Energy</a>
