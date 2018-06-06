Tarjetas
======

Módulo Genérico ESP8266
----------------------

Estos módulos se encuentran con diferentes formas y pineado. Vea la página wiki de la comunidad ESP8266 para mas información: `Módulos de la familia ESP8266 <http://www.esp8266.com/wiki/doku.php?id=esp8266-module-family>`__.

Normalmente estos módulos no tienen resistencias de arranque en la tarjeta (pullup y pulldown), insuficientes condensadores de desacoplamiento, sin regulador de voltaje, sin circuito de reset y sin adaptador USB-Serie. Esto hace que usarlos sea algo complicado, en comparación con las placas de desarrollo que agregan estas características.

Para utilizar estos módulos, asegúrese de observar lo siguiente:

-  **Proveer energía suficiente al módulo.** Para un uso estable del ESP8266 se requiere una fuente de alimentación con 3.3V y >= 250mA. No se recomienda utilizar la alimentación del adaptador USB-Serie, Estos adaptadores normalmente no suministran corriente suficiente para correr el ESP8266 de forma segura en todas las situaciones. Es  preferible un suministro externo o regulador junto con condensadores de filtrado.

-  **Conectar resistencias de arranque** a GPIO0, GPIO2, GPIO15 de acuerdo con los esquemas a continuación.

-  **Poner el ESP8266 en modo bootloader** antes de subir código.

Adaptador Serie
--------------

Hay muchos adaptadores/tarjetas USB a Serie diferentes. Para poner ESP8266 en modo bootloader utilizando líneas de handshaking en serie, necesita el adaptador que interrumpa las salidas RTS y DTR. CTS y DSR no son necesarios para cargar (son entradas). Asegúrese de que el adaptador funciona con voltaje de 3.3V IO: debe tener un puente o un interruptor para seleccionar entre 5V y 3.3V, o estar marcado como 3.3V solamente.

Los adaptadores basados en los siguientes chips deben funcionar:

-  FT232RL
-  CP2102
-  CH340G

Los adaptadores basados en PL230 no funcionan en Mac OS X. Ver https://github.com/igrr/esptool-ck/issues/9 para mas información.

Configuración Hardware mínima para Bootloading y ejecución
------------------------------------------------

+-----------------+------------+------------------+
| PIN             | Resistencia|Adaptador Serie   |
+=================+============+==================+
| VCC             |            | VCC (3.3V)       |
+-----------------+------------+------------------+
| GND             |            | GND              |
+-----------------+------------+------------------+
| TX o GPIO2\*    |            | RX               |
+-----------------+------------+------------------+
| RX              |            | TX               |
+-----------------+------------+------------------+
| GPIO0           | PullUp     | DTR              |
+-----------------+------------+------------------+
| Reset\*         | PullUp     | RTS              |
+-----------------+------------+------------------+
| GPIO15\*        | PullDown   |                  |
+-----------------+------------+------------------+
| CH\_PD          | PullUp     |                  |
+-----------------+------------+------------------+

Note:
-  GPIO15 se puede llamar también MTDO.
-  Reset se puede llamar también RSBT o REST (añadir PullUp para mejorar la estabilidad del módulo).
-  GPIO2 es alternativo TX para el modo bootloader.
-  **Conectar directamente un pin a VCC o GND no sirve para sustituir una resistencia PullUp o PullDown, hacer esto puede afectar al control de subida y al monitor serie, también puede notar instabilidad en algunos casos.**

ESP a Serie
-------------

.. figure:: ESP_to_serial.png
   :alt: ESP to Serial

   ESP to Serial

Configuración Hardware mínima para Bootloading solo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ESPxx Hardware

+---------------+------------+------------------+
| PIN           | Resistencia| Adaptador Serie  |
+===============+============+==================+
| VCC           |            | VCC (3.3V)       |
+---------------+------------+------------------+
| GND           |            | GND              |
+---------------+------------+------------------+
| TX o GPIO2    |            | RX               |
+---------------+------------+------------------+
| RX            |            | TX               |
+---------------+------------+------------------+
| GPIO0         |            | GND              |
+---------------+------------+------------------+
| Reset         |            | RTS\*            |
+---------------+------------+------------------+
| GPIO15        | PullDown   |                  |
+---------------+------------+------------------+
| CH\_PD        | PullUp     |                  |
+---------------+------------+------------------+

Note: 
-  Si no utiliza RTS es necesario un reinicio manual de alimentación

Configuración Hardware mínima para solo ejecutar
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ESPxx Hardware

+----------+------------+----------------+
| PIN      | Resistencia| Fuente aliment.|
+==========+============+================+
| VCC      |            | VCC (3.3V)     |
+----------+------------+----------------+
| GND      |            | GND            |
+----------+------------+----------------+
| GPIO0    | PullUp     |                |
+----------+------------+----------------+
| GPIO15   | PullDown   |                |
+----------+------------+----------------+
| CH\_PD   | PullUp     |                |
+----------+------------+----------------+

Mínimo
-------

.. figure:: ESP_min.png
   :alt: ESP min

   ESP min

Estabilidad mejorada
------------------

.. figure:: ESP_improved_stability.png
   :alt: ESP improved stability

   ESP con estabilidad mejorada

Mensajes de arranque y modos
-----------------------

El módulo ESP comprueba en cada arranque los pines 0, 2 y 15. Arrancando basado en ellos de diferente modo:

+----------+---------+---------+------------------------------------+
| GPIO15   | GPIO0   | GPIO2   | Modo                               |
+==========+=========+=========+====================================+
| 0V       | 0V      | 3.3V    | UART Bootloader                    |
+----------+---------+---------+------------------------------------+
| 0V       | 3.3V    | 3.3V    | Inicia el sketch (SPI flash)       |
+----------+---------+---------+------------------------------------+
| 3.3V     | x       | x       | Modo SDIO (no usado para Arduino)  |
+----------+---------+---------+------------------------------------+

Al inicio el ESP imprime el modo actual de arranque, ejemplo:

::

    rst cause:2, boot mode:(3,6)

Nota: 
- GPIO2 se utiliza como salida TX y el Pullup interno está activo al arrancar.

Causas de reset
~~~~~~~~~

+----------+------------------+
| Número   | Descripción      |
+==========+==================+
| 0        | desconocido      |
+----------+------------------+
| 1        | inicio normal    |
+----------+------------------+
| 2        | reset pin        |
+----------+------------------+
| 3        | software reset   |
+----------+------------------+
| 4        | watchdog reset   |
+----------+------------------+

Modo de arranque
~~~~~~~~~

Es el primer valor respecto a la configuración de pines 0, 2 y 15.

+----------+----------+---------+---------+-------------+
| Número   | GPIO15   | GPIO0   | GPIO2   | Modo        |
+==========+==========+=========+=========+=============+
| 0        | 0V       | 0V      | 0V      | No valido   |
+----------+----------+---------+---------+-------------+
| 1        | 0V       | 0V      | 3.3V    | UART        |
+----------+----------+---------+---------+-------------+
| 2        | 0V       | 3.3V    | 0V      | No valido   |
+----------+----------+---------+---------+-------------+
| 3        | 0V       | 3.3V    | 3.3V    | Flash       |
+----------+----------+---------+---------+-------------+
| 4        | 3.3V     | 0V      | 0V      | SDIO        |
+----------+----------+---------+---------+-------------+
| 5        | 3.3V     | 0V      | 3.3V    | SDIO        |
+----------+----------+---------+---------+-------------+
| 6        | 3.3V     | 3.3V    | 0V      | SDIO        |
+----------+----------+---------+---------+-------------+
| 7        | 3.3V     | 3.3V    | 3.3V    | SDIO        |
+----------+----------+---------+---------+-------------+

Nota:
- Numero = ((GPIO15 << 2) \| (GPIO0 << 1) \| GPIO2);

Módulo Genérico ESP8285
----------------------

ESP8285 (`datasheet <http://www.espressif.com/sites/default/files/0a-esp8285_datasheet_en_v1.0_20160422.pdf>`__) es un paquete multichip el cual contiene un ESP8266 y una flash de 1MB. Todos los puntos relacionados a resistencias de arranque y circuitos recomendados arriba también se aplican a ESP8285.

Nota: Debido a que ESP8285 tiene la memoria flash SPI conectada internamente en modo DOUT, los pines 9 y 10 pueden utilizarse como pines GPIO / I2C / PWM.

ESPDuino (Módulo ESP-13)
------------------------

*TODO*

Adafruit Feather HUZZAH ESP8266
-------------------------------

El ESP8266 Adafruit Feather HUZZAH es una tarjeta de desarrollo WiFi Arduino-compatible alimentada por un módulo ESP-12S de Ai-Thinker, con reloj a 80 MHz y lógica de 3.3V. Incluye un chip USB-Serie de alta calidad SiLabs CP2104 por lo que puedes subir el código a la abrasadora velocidad de 921600 baudios, para un tiempo de desarrollo rápido. También tiene reinicio automático, por lo que no hay que tocar ningún pin y ni presionar nada para reset. Incluye un conector de batería de polímero de litio de 3.7V, por lo que es ideal para proyectos portátiles. El ESP8266 Adafruit Feather HUZZAH recargará automáticamente una batería conectada cuando la alimentación USB esté disponible.

Página del producto: https://www.adafruit.com/product/2821

ESPresso Lite 1.0
-----------------

ESPresso Lite 1.0 (versión beta) es una tarjeta de desarrollo WiFi Arduino-compatible alimentada por su propio módulo Epressif System's WROOM-02. Posee un amigable pineado tipo breadboard con un LED integrado, dos botones reset/flash y un botón programable por el usuario. El voltaje de trabajo es 3.3VDC, regulado con corriente máxima 800mA. Una característica distintiva especial es que posee un pad integrado I2C pads el cual permite una conexión directa a un LCD OLED y tarjetas de sensores.

ESPresso Lite 2.0
-----------------

ESPresso Lite 2.0 es una tarjeta de desarrollo WiFi Arduino-compatible basada en la V1 (beta versión). Rediseñada junto con Cytron Technologies, La nueva/revisada ESPresso Lite V2.0 posee la función de auto carga/auto programación, eliminando la anterior necesidad de resetear la tarjeta manualmente tras flasear un nuevo programa. También posee dos botones programables por el usuario y un botón de reset. El distintivo especial es que posee pads integrados para I2C sensor.

Phoenix 1.0
-----------

Página del producto: http://www.espert.co

Phoenix 2.0
-----------

Página del producto: http://www.espert.co

NodeMCU 0.9 (Módulo ESP-12)
---------------------------

Mapa de pines
~~~~~~~~~~~

La numeración de pines impresa en la tarjeta no se corresponde con la numeración GPIO del ESP8266. Se han definido constantes para facilitar un uso sencillo:

.. code:: c++

    static const uint8_t D0   = 16;
    static const uint8_t D1   = 5;
    static const uint8_t D2   = 4;
    static const uint8_t D3   = 0;
    static const uint8_t D4   = 2;
    static const uint8_t D5   = 14;
    static const uint8_t D6   = 12;
    static const uint8_t D7   = 13;
    static const uint8_t D8   = 15;
    static const uint8_t D9   = 3;
    static const uint8_t D10  = 1;

Si deseas usar el pin 5 del NodeMCU, utiliza "D5" como número del pin y será traducido al real GPIO pin 14.

NodeMCU 1.0 (Módulo ESP-12E)
----------------------------

Este módulo se vende con muchos nombres en AliExpress por menos de 6.5$ y es uno de los mas baratos, posee soluciones completamente integradas en el ESP8266.

Se trata de un diseño Open Hardware con un core ESP-12E y una flash SPI de 4 MB.

De acuerdo con el fabricante, "con un micro cable USB, puedes conectar el kit de desarrollo NodeMCU a tu portátil y flasearlo sin ningún problema". Este es mas o menos verdad: la tarjeta viene con un adaptador integrado USB-Serie CP2102 el cual funciona bien la mayoría de veces. Algunas veces falla y necesitas resetear la tarjeta pulsando y manteniendo FLASH y RST, soltando FLASH y entonces soltando RST. Esto fuerza al dispositivo CP2102 a realizar un ciclo de alimentación y a ser reenumerado en Linux.

La tarjeta también integra un regulador de voltaje NCP1117, un LED azul en GPIO16 y un divisor de voltaje 220k/100k Ohm en el pin de entrada ADC.

El pinout completo y esquema en PDF, se encuentra `aquí <https://github.com/nodemcu/nodemcu-devkit-v1.0>`__

Olimex MOD-WIFI-ESP8266(-DEV)
-----------------------------

Esta tarjeta tiene flash SPI de 2 MB y accesorios adicionales (p.ej. tarjeta de evaluación ESP8266-EVB o BAT-BOX para baterías).

El módulo básico tiene 3 jumpers soldados que te permiten cambiar el modo de operación entre SDIO, UART y FLASH.

La tarjeta se envía en el modo de operación FLASH, con jumpers TD0JP=0, IO0JP=1, IO2JP=1.

Como el jumper IO0JP está vinculado a GPIO0, que es PIN 21, tendrás que conectarlo a tierra antes de programarlo con un adaptador de USB a Serie y reiniciar la placa apagándola.

Los pines UART para programación y E/S Serial son GPIO1 (TXD, pin 3) y GPIO3 (RXD, pin 4).

Puedes ver el esquema de la tarjeta `aquí <https://github.com/OLIMEX/ESP8266/blob/master/HARDWARE/MOD-WIFI-ESP8266-DEV/MOD-WIFI-ESP8266-DEV_schematic.pdf>`__

SparkFun ESP8266 Thing
----------------------

Página del producto: https://www.sparkfun.com/products/13231

SparkFun ESP8266 Thing Dev
--------------------------

Página del producto: https://www.sparkfun.com/products/13711

SweetPea ESP-210
----------------

*TODO*

WeMos D1 R2 & mini
------------------

Página del producto: https://www.wemos.cc/

WeMos D1 mini Pro
-----------------

Página del producto: https://www.wemos.cc/

WeMos D1 mini Lite
------------------

Página del producto: https://www.wemos.cc/

WeMos D1 R1
-----------

Página del producto: https://www.wemos.cc/

ESPino (Módulo ESP-12)
----------------------

ESPino integra el módulo ESP-12 con un regulador de 3.3v, adaptador USB-Serie CP2104 y un conector micro USB para una fácil programación. Está diseñado para adaptarse a una a breadboard ay tiene un led RGB y 2 botones para prototipado fácil.

Mas información sobre el hardware, pinout, diagrama y procedimiento de programación, por favor vea el `datasheet <https://github.com/makerlabmx/ESPino-tools/raw/master/Docs/ESPino-Datasheet-EN.pdf>`__.

Página del producto: http://www.espino.io/en

ThaiEasyElec's ESPino
---------------------

ESPino by ThaiEasyElec utiliza el módulo WROOM-02 de Espressif Systems con flash de 4 MB.

Se actualizará una descripción pronto. - Página del producto: 
http://thaieasyelec.com/products/wireless-modules/wifi-modules/espino-wifi-development-board-detail.html
- Esquema:
www.thaieasyelec.com/downloads/ETEE052/ETEE052\_ESPino\_Schematic.pdf -
Dimensiones:
http://thaieasyelec.com/downloads/ETEE052/ETEE052\_ESPino\_Dimension.pdf
- Pinouts:
http://thaieasyelec.com/downloads/ETEE052/ETEE052\_ESPino\_User\_Manual\_TH\_v1\_0\_20160204.pdf (Ver pág.. 8)

WifInfo
-------

WifInfo integra el módulo ESP-12 o el ESP-07+Ext antena con un regulador de 3.3v y el hardware es capaz de medir la telemetría francesa a partir de la salida en serie del medidor de potencia ERDF. Tiene un conector USB para alimentación, un Led RGB WS2812, conector I2C de 4 pines para adaptarse a OLED o sensor, y dos botones + conector FTDI y función de reinicio automático.

Mas información sobre WifInfo, ver el siguiente `blog <http://hallard.me/category/wifinfo/>`__ , `Github <https://github.com/hallard/WifInfo>`__ y  `foro de la comunidad <https://community.hallard.me/category/16/wifinfo>`__.

Arduino
-------

*TODO*

4D Systems gen4 IoD Range
-------------------------

gen4-IoD Range de ESP8266 tiene módulos de pantalla de 4D Systems.

2.4", 2.8" y 3.2" TFT LCD con uSD card socket y tactil resistivo. Chip de antena + conector uFL.

Datasheet y descargas asociadas pueden encontrarse en la página del producto de 4D Systems.

La gama de productos gen4-IoD puede programarse utilizando el IDE de Arduino y también IDE 4D Systems Workshop4, el cual incorpora muchos beneficios gráficos adicionales. La librería GFX4d está disponible, junto con una serie de aplicaciones de demostración.

- Página del producto: http://www.4dsystems.com.au/product/gen4-IoD

Digistump Oak
-------------

El Oak requiere un `Adaptador Serie`_ para la conexión serie o flaseado; Su puerto micro USB es solo para alimentación.

Para realizar la conexión serie, conecte el adaptador **TX a P3**, **RX a P4** y **GND a GND**. Alimentar 3.3v desde el adaptador serie si no está ya alimentado mediante el USB.

Para poner la tarjeta en modo bootloader, configure una conexión serie como anteriormente y conecte **P2 a GND**, después vuelva a alimentar.  Una vez que el flaseado se ha completado, elimine la conexión de P2 a GND, entonces vuelva a alimentar para iniciar en modo normal.

WiFiduino
---------

- Página del producto: https://wifiduino.com/esp8266

Amperka WiFi Slot
-----------------

- Página del producto: http://wiki.amperka.ru/wifi-slot
