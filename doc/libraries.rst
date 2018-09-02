Librerías
=========

WiFi(librería ESP8266WiFi)
-------------------------

La librería ESP8266WiFi ha sido desarrollada basándose en el SDK ESP8266, utilizando la nomenclatura convencional y la filosofía de la funcionalidad de la `Librería de la Shield Arduino WiFi <https://www.arduino.cc/en/Reference/WiFi>`__. Con el tiempo, la riqueza de las funciones WiFi portadas del desde el SDK ESP8266 a esta librería superaron a las APIs de la librería de la Shield WiFi y se hizo evidente que tenemos que proporcionar documentación por separado sobre lo que es nuevo y extra.

`Documentación de la librería ESP8266WiFi <esp8266wifi/readme.rst>`__

Ticker
------

Es una librería para llamar a funciones repetidas cada cierto periodo de tiempo. `Dos ejemplos <https://github.com/esp8266/Arduino/tree/master/libraries/Ticker/examples>`_ incluidos.

Actualmente no se recomienda realizar operaciones de bloqueo de IO (network, serial, file) desde una llamada a la función Ticker. En su lugar, establece una bandera dentro de la llamada a ticker y chequea por esta bandera desde dentro de la función loop.

Aquí hay una librería que simplifica el uso de ``Ticker`` y evita el reset WDT:
`TickerScheduler <https://github.com/Toshik/TickerScheduler>`__

EEPROM
------

Este es un poco diferente de la clase estándar EEPROM. Necesitas llamar a ``EEPROM.begin(size)`` antes de iniciar a leer o escribir, size es el número de bytes que desea utilizar. Size puede estar entre 4 y 4096 bytes.

``EEPROM.write`` no escriba a la flash inmediatamente, en su lugar llame ``EEPROM.commit()`` cada vez que desee guardar cambios en la flash. ``EEPROM.end()`` también guardará y liberará la copia RAM de contenido EEPROM.

La librería EEPROM utiliza un sector de la flash justo después de SPIFFS.

`Tres ejemplos <https://github.com/esp8266/Arduino/tree/master/libraries/EEPROM>`__  incluidos.

I2C (librería Wire)
------------------

La librería Wire actualmente soporta el modo maestro hasta aproximadamente 450 KHz. antes de utilizar I2C, los pines para SDA y SCL necesitan ser establecidos llamando ``Wire.begin(int sda, int scl)``, p.ej. ``Wire.begin(0, 2)`` en ESP-01, de lo contrario, los pines por defecto son 4(SDA) y 5(SCL).

SPI
---

La librería SPI soporta por completo la API Arduino SPI incluyendo las transacciones, incluyendo la fase de ajuste (CPHA). El ajuste de Clock polarity (CPOL) no está soportado, todavía (SPI\_MODE2 y SPI\_MODE3 no funciona).

Los pines SPI usuales son: 

- ``MOSI`` = GPIO13
- ``MISO`` = GPIO12
- ``SCLK`` = GPIO14

Hay un modo extendido donde puedes intercambiar los pines normales a los pines hardware SPI0. Esto se activa llamando ``SPI.pins(6, 7, 8, 0)`` antes de llamar a ``SPI.begin()``. Los pines cambiarían a:

- ``MOSI`` = SD1
- ``MISO`` = SD0
- ``SCLK`` = CLK
- ``HWCS`` = GPIO0

De este modo se liberan los pines SPI con el controlador que lee el código del programa desde la flash y se controla por un árbitro de hardware (la flash tiene siempre alta prioridad). De este modo el CS será controlado por hardware y como ya no puede manejar la línea CS con un GPIO nunca sabrá realmente cuándo el árbitro le otorgará acceso al bus, por este motivo debe dejar que maneje el CS automáticamente.


SoftwareSerial
--------------

Una librería portada a ESP8266 de la librería SoftwareSerial hecha por Peter Lerup (@plerup) soporta velocidades hasta 115200 y múltiples instancias de SoftwareSerial. Ver https://github.com/plerup/espsoftwareserial si desea sugerir una mejora o abrir un issue relacionado con SoftwareSerial.

APIs especificas de ESP 
-----------------

Algunas APIs especificas de ESP relacionadas a deep sleep, RTC y memoria flash están disponibles en el objeto ``ESP``.

``ESP.deepSleep(microseconds, mode)`` pondrá el chip en deep sleep (sueño profundo - ahorro de energía). ``mode`` puede ser ``WAKE_RF_DEFAULT``, ``WAKE_RFCAL``, ``WAKE_NO_RFCAL``, ``WAKE_RF_DISABLED``. (Se necesita unir GPIO16 a RST para despertar del deep sleep). El chip puede dormir como mucho ``ESP.deepSleepMax()`` microsegundos.

``ESP.deepSleepInstant(microseconds, mode)`` funciona similar a ``ESP.deepSleep`` pero duerme instantaneamente sin esperar a que el WiFi se apague.

``ESP.rtcUserMemoryWrite(offset, &data, sizeof(data))`` y ``ESP.rtcUserMemoryRead(offset, &data, sizeof(data))`` permite a los datos ser almacenados y recuperados de la memoria de usuario RTC del chip, respectivamente. El tamaño total de la memoria de usuario RTC es 512 bytes, así que ``offset + sizeof(data)`` no debe exceder 512. Los datos deben tener una alineación de 4-byte. Los datos almacenados pueden retenerse entre ciclos deep sleep. Sin embargo, los datos se pierden tras un ciclo de realimentación del chip.

``ESP.restart()`` reinicia la CPU.

``ESP.getResetReason()`` devuelve un String conteniendo la última razón de reset en un formato leíble por un humano.

``ESP.getFreeHeap()`` devuelve el tamaño libre de la pila.

``ESP.getChipId()`` devuelve el ID del chip ESP8266 como un 32-bit integer.

``ESP.getCoreVersion()`` devuelve un String con la versión del core.

``ESP.getSdkVersion()`` devuelve la versión del SDK como un char.

``ESP.getCpuFreqMHz()`` devuelve la frecuencia de la CPU en MHz como un unsigned 8-bit integer.

``ESP.getSketchSize()`` devuelve el tamaño del actual sketch como un unsigned 32-bit integer.

``ESP.getFreeSketchSpace()`` devuelve el espacio libre de sketch como un unsigned 32-bit integer.

``ESP.getSketchMD5()`` devuelve una String con el MD5 (en minúscula) del actual sketch sketch.

``ESP.getFlashChipId()`` devuelve el ID del chip flash como un 32-bit integer.

``ESP.getFlashChipSize()`` devuelve el tamaño del chip flash, en bytes, como lo ve el SDK (puede ser menor que el tamaño real).

``ESP.getFlashChipRealSize()`` devuelve el tamaño real del chip, en bytes, basado en el ID del chip flash.

``ESP.getFlashChipSpeed(void)`` devuelve la frecuencia del chip flash, en Hz.

``ESP.getCycleCount()`` devuelve la cuenta de ciclos de instrucciones de  la CPU desde el arranque como un unsigned 32-bit. Esto es Esto es útil para tiempos precisos de acciones muy cortas, como bit banging.

``ESP.getVcc()`` puede usarse para medir el voltaje suministrado. ESP necesita reconfigurar el ADC al inicio para poder tener esta caracteristica disponible. Añade la siguiente línea en lo alto de tu sketch para utilizar ``getVcc``:

.. code:: cpp

    ADC_MODE(ADC_VCC);

El pin TOUT debe estar desconectado en este modo.

Nota: por defecto ADC está configurado para leer del pin TOUT pin utilizando ``analogRead(A0)``, y ``ESP.getVcc()`` no está disponible.

Respondedor mDNS y DNS-SD (librería ESP8266mDNS)
-----------------------------------------------

Permite al sketch responder a llamadas multicast DNS para nombres de dominios como "foo.local", y llamadas DNS-SD (descubrimiento de servicios). Ver el ejemplo incluido para mas detalle.

Respondedor SSDP (ESP8266SSDP)
----------------------------

SSDP es otro protocolo de servicio de descubrimiento, suportado en Windows. Ver ejemplo incluido para referencia.

Servidor DNS (librería DNSServer)
------------------------------

Implementa un servidor simple DNS que puede usarse en ambos modos STA y AP. Actualmente el servidor DNS soporta solo un dominio (para otros dominios responde con NXDOMAIN o un código de estatus personalizado). Con esto, los clientes pueden abrir un servidor web corriendo en el ESP8266 utilizando un nombre de dominio, en vez de una dirección IP.

Servo
-----

Esta biblioteca permite la capacidad de controlar motores servo RC (hobby). Admite hasta 24 servos en cualquier pin de salida disponible. Por definición, los primeros 12 servos usarán Timer0 y actualmente esto no interferirá con ningún otro soporte. Los conteos de servos superiores a 12 utilizarán Timer1 y las funciones que lo utilizan se verán afectadas. Si bien muchos servomotores RC aceptarán el pin de datos IO de 3.3V de un ESP8266, la mayoría no podrá funcionar a 3.3v y requerirá otra fuente de alimentación que coincida con sus especificaciones. Asegúrese de conectar los cables entre el ESP8266 y la fuente de alimentación del servomotor.

Librería mejorada EEPROM para ESP (ESP_EEPROM)
--------------------------------------------

Una biblioteca mejorada para la EEPROM de ESPxxxx. Utiliza la memoria flash de acuerdo con la biblioteca estándar ESP EEPROM, pero reduce el reflash, por lo que reduce el desgaste y mejora el rendimiento de commit().

Como las acciones en el flash deben detener las interrupciones, un reflash de la EEPROM podría afectar notoriamente cualquier cosa usando PWM, etc.

Otras librerías (no incluidas con el IDE)
-------------------------------------------

Las bibliotecas que no dependen del acceso a bajo nivel a los registros AVR deberían funcionar bien. Aquí hay algunas bibliotecas que se verificó que funcionan:

-  `Adafruit\_ILI9341 <https://github.com/Links2004/Adafruit_ILI9341>`__ - Adafruit ILI9341 para el ESP8266
-  `arduinoVNC <https://github.com/Links2004/arduinoVNC>`__ - Cliente VNC para Arduino
-  `arduinoWebSockets <https://github.com/Links2004/arduinoWebSockets>`__ - Servidor y cliente WebSocket compatible con ESP8266 (RFC6455)
-  `aREST <https://github.com/marcoschwartz/aREST>`__ - Manejador de la librería REST API.
-  `Blynk <https://github.com/blynkkk/blynk-library>`__ - IoT framework sencillo para Makers (comprueba la `página de inicio rápido <https://www.blynk.cc/getting-started/>`__).
-  `DallasTemperature <https://github.com/milesburton/Arduino-Temperature-Control-Library.git>`__
-  `DHT-sensor-library <https://github.com/adafruit/DHT-sensor-library>`__ - Librería Arduino para el sensor DHT11/DHT22 de temperatura y humedad. Descarga la última librería v1.1.1 y no serán necesarios cambios. Las versiones antiguas deben inicializar el DHT como sigue: ``DHT dht(DHTPIN, DHTTYPE, 15)``
-  `DimSwitch <https://github.com/krzychb/DimSwitch>`__ - Control electrónico regulable de balastros para luces de tubo fluorescentes remotamente como si se usara un interruptor de pared.
-  `Encoder <https://github.com/PaulStoffregen/Encoder>`__ - Librería Arduino para encoders rotatorios. Versión 1.4 soporta ESP8266.
-  `esp8266\_mdns <https://github.com/mrdunk/esp8266_mdns>`__ - Llamadas y respuestas mDNS en esp8266. O dicho de otro modo: Un cliente mDNS o librería de cliente Bonjour para el ESP8266.
-  `ESPAsyncTCP <https://github.com/me-no-dev/ESPAsyncTCP>`__ - Librería asíncrona TCP para ESP8266 y ESP32/31B
-  `ESPAsyncWebServer <https://github.com/me-no-dev/ESPAsyncWebServer>`__ - Librería de Servidor Web asíncrono para ESP8266 y ESP32/31B
-  `Homie for ESP8266 <https://github.com/marvinroger/homie-esp8266>`__ - Arduino framework para ESP8266 implementando Homie, una convención MQTT para IoT.
-  `NeoPixel <https://github.com/adafruit/Adafruit_NeoPixel>`__ - Librería de Neopixel de Adafruit, ahora con soporte para el ESP8266 (utiliza la versión 1.0.2 o superior desde el Gestor de librerías de Arduino).
-  `NeoPixelBus <https://github.com/Makuna/NeoPixelBus>`__ - Librería de Neopixel para Arduino compatible con ESP8266. Utiliza el "DmaDriven" o "UartDriven" branches para ESP8266. Incluye soporte de color HSL y mas.
-  `PubSubClient <https://github.com/Imroy/pubsubclient>`__ - Librería MQTT por @Imroy.
-  `RTC <https://github.com/Makuna/Rtc>`__ - Librería Arduino para DS1307 y DS3231 compatible con ESP8266.
-  `Souliss, Smart Home <https://github.com/souliss/souliss>`__ - Framework para Smart Home basado en Arduino, Android y openHAB.
-  `ST7735 <https://github.com/nzmichaelh/Adafruit-ST7735-Library>`__ - Librería de ST7735 de Adafruit modificada para ser compatible con ESP8266. Solo asegúrate de modificar los pines en el ejemplo por los todavía específicos de AVR.
-  `Task <https://github.com/Makuna/Task>`__ - Librería no preventiva de multitarea de Arduino. Si bien es similar a la biblioteca Ticker incluida, esta librería fue diseñada para mantener la compatibilidad con Arduino.
-  `TickerScheduler <https://github.com/Toshik/TickerScheduler>`__ - Librería que provee un simple planificador para ``Ticker`` para prevenir el reset WDT.
-  `Teleinfo <https://github.com/hallard/LibTeleinfo>`__ - Librería del contador de energía genérico francés para leer los datos de monitorización de la energía Teleinfo como son consumo, contrato, potencia, periodo, ... Esta librería es de plataforma cruzada ESP8266, Arduino, Particle, y simple C++. `Post <https://hallard.me/libteleinfo/>`__  dedicado francés en el blog del autor y toda la información `Teleinfo <https://hallard.me/category/tinfo/>`__ también disponible.
-  `UTFT-ESP8266 <https://github.com/gnulabis/UTFT-ESP8266>`__ - Librería para pantallas UTFT con soporte para ESP8266. Solo pantallas con soporte serial interface (SPI) por ahora (no 8-bit parallel mode, etc). También incluye soporte para el controlador hardware SPI de el ESP8266.
-  `WiFiManager <https://github.com/tzapu/WiFiManager>`__ - Gestor de conexión WiFi con portal cautivo Web. Si no puede conectarse, se iniciará en modo AP y un portal de configuración donde podrás introducir tus credenciales WiFi.
-  `OneWire <https://github.com/PaulStoffregen/OneWire>`__ - Librerías para chips Dallas/Maxim 1-Wire.
-  `Adafruit-PCD8544-Nokia-5110-LCD-Library <https://github.com/WereCatf/Adafruit-PCD8544-Nokia-5110-LCD-library>`__ - Librería de PCD8544 de Adafruit para el ESP8266.
-  `PCF8574\_ESP <https://github.com/WereCatf/PCF8574_ESP>`__ - Una librería muy simple para utilizar el expansor de GPIOs PCF857//PCF8574A I2C 8-pin.
-  `Dot Matrix Display Library 2 <https://github.com/freetronics/DMD2>`__ - Librería Freetronics DMD y pantalla Generic 16 x 32 P10 style Dot Matrix.
-  `SdFat-beta <https://github.com/greiman/SdFat-beta>`__ - Librería para tarjetas SD con soporte para nombres largos, SPI basado en software y hardware y mucho mas.
-  `FastLED <https://github.com/FastLED/FastLED>`__ - Una librería para controlar fácil y eficientemente una amplia variedad de chipsets LED, como el Neopixel (WS2812B), DotStar, LPD8806 y algunos mas. Incluye desvanecimiento, gradiente, funciones de conversión de color.
-  `OLED <https://github.com/klarsys/esp8266-OLED>`__ - Una librería para controlar pantallas OLED conectadas con I2C. Testeado con pantallas OLED gráficas de 0.96 pulgadas.
-  `MFRC522 <https://github.com/miguelbalboa/rfid>`__ - Una librería para utilizar el lector/escritor de tags RFID Mifare RC522.
-  `Ping <https://github.com/dancol90/ESP8266Ping>`__ - Permite al ESP8266 hacer ping a una máquina remota.
-  `AsyncPing <https://github.com/akaJes/AsyncPing>`__ - Librería totalmente asíncrona de Ping (tiene estadísticas completas ping y direcciones hardware MAC).
