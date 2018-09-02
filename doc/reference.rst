Referencia
=========

Digital IO
----------

Los números de los pines en Arduino se corresponden directamente con los números de los pines GPIO del ESP8266. Las funciones ``pinMode``, ``digitalRead`` y ``digitalWrite`` funciona como de costumbre, así para leer el GPIO2, llama ``digitalRead(2)``.

Los pines digitales 0—15 pueden ser ``INPUT``, ``OUTPUT`` o ``INPUT_PULLUP``. El pin 16 puede ser ``INPUT``, ``OUTPUT`` o ``INPUT_PULLDOWN_16``. Al arranque los pines están configurados como ``INPUT``.

Los pines pueden también realizar otras funciones como: Serial, I2C, SPI. Estas funciones se activan normalmente mediante su correspondiente librería. El siguiente diagrama muestra el mapeo de pines para el popular módulo ESP-12.

.. figure:: esp12.png
   :alt: Funciones de los pines

   Funciones de los pines

Los pines digitales 6—11 no se muestran en el diagrama porque se utilizan para conectar la memoria flash en la mayoría de módulos. Al intentar usar estos pines como E/S hará que el programa falle.

Nota: Algunas tarjetas y módulos (ESP-12ED, NodeMCU 1.0) también poseen los pines 9 y 11. Estos pueden usarse si el chip flash trabaja en modo DIO (en vez de QIO, que es el modo por defecto).

La interrupción de pines se realiza a través de las funciones ``attachInterrupt``, ``detachInterrupt``. Las interrupciones pueden asociarse a cualquier GPIO, excepto GPIO16. Los tipos de interrupciones estándar de Arduino soportados son: ``CHANGE``, ``RISING``, ``FALLING``.

Entrada analógica
------------

El ESP8266 tiene un solo canal ADC disponible para los usuarios. Puede utilizarse para leer el voltaje en el pin ADC o para leer el voltaje de alimentación del módulo (VCC).

Para leer el voltaje externo suministrado al pin ADC utiliza ``analogRead(A0)``. El rango de voltaje de entrada es 0 — 1.0V.

Para leer el voltaje VCC, utiliza ``ESP.getVcc()`` y el pin ADC debe mantenerse sin conectar. Adicionalmente, la siguiente línea tiene que añadirse al sketch:

.. code:: cpp

    ADC_MODE(ADC_VCC);

Esta linea tiene que estar fuera de cualquier función, por ejemplo, justo después de las líneas ``#include`` de tu sketch.

Salidas Analógicas
-------------

``analogWrite(pin, value)`` activa el PWM software en el pin seleccionado. PWM puede usarse en los pines del 0 al 16. Llamar ``analogWrite(pin, 0)`` para desactivar PWM en el pin. ``value`` puede estar en un rango desde 0 a ``PWMRANGE``, el cual es igual a 1023 por defecto. El rango PWM puede cambiarse llamando a ``analogWriteRange(new_range)``.

La frecuencia PWM es de 1kHz por defcto. Llama a ``analogWriteFreq(new_frequency)`` para cambiar la frecuencia.

Tiempos y retrasos (delays)
-----------------

``millis()`` y ``micros()`` devuelven el número de milisegundos y microsegundos transcurridos desde el reset, respectivamente.

``delay(ms)`` pausa el sketch por un número determinado de milisegundos y permite correr a las tareas WiFi y TCP/IP. ``delayMicroseconds(us)`` pausa por un número dado de microsegundos.

Recuerda que hay mucho código que necesita correr en el chip además del sketch cuando el WiFi está conectado. Las librerías WiFi y TCP/IP tienen una oportunidad de manejar cualquier evento pendiente cada vez que la función ``loop()`` se completa o cuando se llama a ``delay``. Si en algún sitio de loop tu sketch toma mucho tiempo (>50ms) sin llamar a ``delay``, deberías considerar añadir una llamada a la función ``delay`` para mantener la pila WiFi corriendo adecuadamente.

Existe también la función ``yield()``, la cual es equivalente a ``delay(0)``. La función ``delayMicroseconds``, por otro lado, no libera el control para otras tareas, por lo tanto no se recomienda realizar retrasos de mas de 20 milisegundos.

Serial
------

El objeto ``Serial`` funciona del mismo modo que en un Arduino normal. Aparte el hardware FIFO (128 bytes para TX y RX) ``Serial`` tiene un buffer adicional de 256-byte para TX y RX. Ambos transmiten y reciben por medio de una interrupción. Las funciones de lectura y escritura solo bloquean la ejecución del sketch cuando el respectivo FIFO/buffer está lleno/vacío. Nota: la longitud del buffer adicional de 256-bit puede ser personalizado.

``Serial`` utiliza UART0, el cual está mapeado a los pines GPIO1 (TX) y GPIO3 (RX). El Serial puede ser redirigido a GPIO15 (TX) y GPIO13 (RX) llamando a ``Serial.swap()`` después de ``Serial.begin``. Llamando a ``swap`` otra vez se mapea UART0 de nuevo a GPIO1 y GPIO3.

``Serial1`` utiliza UART1, el pin de TX es GPIO2. UART1 no puede utilizarse para recibir datos porque normalmente el pin de RX esta ocupado en el flaseo del chip. Para utilizar ``Serial1``, llama ``Serial1.begin(baudrate)``.

Si ``Serial1`` no se utiliza y ``Serial`` no se intercambia, TX para UART0 puede ser mapeado a GPIO2 en su lugar llamando ``Serial.set_tx(2)`` después de ``Serial.begin`` o directamente con ``Serial.begin(baud, config, mode, 2)``.

Por defecto la salida de diagnóstico de las librerías WiFi están desactivadas cuando llamas a ``Serial.begin``. Para activar la salida de debug, llama ``Serial.setDebugOutput(true)``. Para redirigir la salida de debug a ``Serial1``, llama ``Serial1.setDebugOutput(true)``.

También puedes utilizar ``Serial.setDebugOutput(true)`` para activar la salida de la función ``printf()``.

El método ``Serial.setRxBufferSize(size_t size)`` permite definir el tamaño del buffer de recepción. El valor por defecto es 256.

Ambos objetos ``Serial`` y ``Serial1`` soportan 5, 6, 7, 8 data bits, odd (O), even (E), y no (N) parity, y 1 o 2 stop bits. Para definir el modo deseado, llama ``Serial.begin(baudrate, SERIAL_8N1)``, ``Serial.begin(baudrate, SERIAL_6E2)``, etc.

Se ha implementado un nuevo método en ambos ``Serial`` y ``Serial1`` para obtener la configuración actual de velocidad. Para obtener la velocidad actual, llama ``Serial.baudRate()``, ``Serial1.baudRate()``. Obtendrá un ``int`` con la velocidad actual. Por ejemplo:

.. code:: cpp

    // Establece la velocidad a 57600
    Serial.begin(57600);

    // Obtiene la velocidad actual
    int br = Serial.baudRate();

    // Imprimirá "Serial is 57600 bps"
    Serial.printf("Serial is %d bps", br);

Ambos objetos ``Serial`` y ``Serial1`` son instancias de la clase ``HardwareSerial``.

Se ha realizado también una librería oficial de ESP8266 de la librería `Software Serial <libraries.rst#softwareserial>`__, ver este `pull request <https://github.com/plerup/espsoftwareserial/pull/22>`__.

Nota: es una implementación **solo para tarjetas basadas en ESP8266** y no funcionará con otras tarjetas Arduino.

Para detectar la velocidad en baudios desconocida de los datos que entran en el puerto serie, use ``Serial.detectBaudrate(time_t timeoutMillis)``. Este método intenta detectar la velocidad de transmisión en baudios para un máximo de tiempo timeoutMillis en ms. Devuelve cero si no se detectó la velocidad en baudios, o de lo contrario la velocidad en baudios detectada. La función ``detectBaudrate()`` se puede invocar antes de llamar a ``Serial.begin()``, ya que no necesita el búfer de recepción ni los parámetros de SerialConfig.

El UART no puede detectar otros parámetros como el start- o stopbits, número de Data bits o paridad.

La detección no cambia a si mismo la velocidad, tras la detección debes establecerla como normalmente utilizando ``Serial.begin(detectedBaudrate)``.

La detección es muy rápida, solo requiere unos pocos bytes entrantes.

SerialDetectBaudrate.ino es un ejemplo completo de uso.

Progmem
-------

Las características de la memoria de programa funcionan de la misma forma que en un Arduino normal; colocando datos de solo lectura y cadenas en la memoria de solo lectura libera la pila de su aplicación. La diferencia mas importante es que en el ESP8266 las cadenas literales no se agrupan. Esto significa que la misma cadena literal definida dentro de un `` F ("") `` y / o `` PSTR ("") `` tomará
espacio para cada instancia en el código. Por lo tanto, tendrá que administrar el duplicado de strings usted mismo.

Hay una macro de ayuda adicional para que sea más fácil pasar cadenas `` const PROGMEM`` a métodos que toman un `` __FlashStringHelper``
llamada `` FPSTR () ``. Su uso ayudará a que sea más fácil juntar cadenas. No agrupando cadenas...

.. code:: cpp

    String response1;
    response1 += F("http:");
    ...
    String response2;
    response2 += F("http:");

utilizando FPSTR sería...

.. code:: cpp

    const char HTTP[] PROGMEM = "http:";
    ...
    {
        String response1;
        response1 += FPSTR(HTTP);
        ...
        String response2;
        response2 += FPSTR(HTTP);
    }
