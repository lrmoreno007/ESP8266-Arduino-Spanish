Mi ESP se bloquea al correr el programa. ¿Como lo resuelvo?
---------------------------------------------------------

-  `Introducción <#introducción>`__
-  `Que debe decir el ESP <#que-debe-decir-el-esp>`__
-  `Pon tu hardware correctamente <#pon-tu-hardware-correctamente>`__
-  `¿Cual es la causa del reinicio? <#cual-es-la-causa-del-reinicio>`__
-  `Excepción <#excepción>`__
-  `Watchdog (Perro Guardián) <#watchdog-perro-guardián>`__
-  `Comprueba donde se bloquea el código <#comprueba-donde-se-bloquea-el-código>`__
-  `Otras causas de bloqueo <#otras-causas-de-bloqueo>`__
-  `Contra la pared, abre un Issue <#contra-la-pared-abre-un-issue>`__
-  `Conclusión <#conclusión>`__

Introducción
~~~~~~~~~~~~

Tu ESP se auto resetea. No sabes por qué y que hacer al respecto.

No te asustes.

En la mayoría de los casos el ESP proporciona suficientes pistas en el Monitor Serie que puedes interpretar para identificar la causa raíz. El primer paso es verificar qué dice el ESP en el Monitor Serie cuando falla.

Que debe decir el ESP
~~~~~~~~~~~~~~~~~~~~~

Comienza abriendo el Monitor Serie (Ctrl+Shift+M) para ver los mensajes. Un registro de bloqueo típico se ve así:

.. figure:: pictures/a02-typical-crash-log.png
   :alt: Registro de bloqueo típico

   Registro de bloqueo típico

Antes de apresurarte a copiar y pegar el código mostrado en Google, reflexiona por un momento sobre la naturaleza de los reinicios observados:

- ¿Se reinicia el ESP aleatoriamente, o bajo ciertas condiciones, como servir una página web?
- ¿Ves siempre el mismo código de excepción y el mismo volcado de pila o este cambia?
- ¿Este problema ocurre con el código de un ejemplo estándar no modificado (IDE Arduino > *Archivo > Ejemplos*)?

Si los reinicios son aleatorios o el código de excepción difiere entre reinicios, entonces el problema puede ser causado por hardware. Si el problema se produce para los ejemplos estándar y el core stable de `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__, es casi seguro que el problema se debe al hardware.

Pon tu hardware correctamente
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si sospechas del hardware, antes de resolver problemas en el software pon tu hardware a punto. No tiene mucho sentido diagnosticar el software si la tarjeta se cuelga aleatoriamente debido a que no hay suficiente energía, faltan resistencias de arranque (pull up - pull down) o hay conexiones sueltas.

Si está utilizando módulos ESP genéricos, siga las `recomendaciones <Generic% 20ESP8266% 20modules>`__ sobre la fuente de alimentación y las resistencias de arranque.

Para tarjetas con convertidor de USB a serie integrado y fuente de alimentación, por lo general es suficiente con conectarlo a un concentrador USB que proporcione 0.5A estándar y no compartir con otros dispositivos USB.

En cualquier caso, asegúrese de que su módulo pueda ejecutar sketch de ejemplo estándar stable que establezcan una conexión Wi-Fi, como p. ej. `HelloServer.ino <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer/examples/HelloServer>`__.

¿Cual es la causa del reinicio?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ya has verificado el hardware del ESP pero sigues teniendo reinicios. Así es como el ESP reacciona al comportamiento anormal de la aplicación. Si algo está mal, se reinicia para contártelo.

Hay dos escenarios típicos que desencadenan reinicios del ESP:

- **Excepción** - cuando el código está realizando una `operación ilegal <../exception_causes.rst>`__, como intentar escribir en una ubicación de memoria inexistente.
- **Watchdog** - si el código está `bloqueado <https://en.wikipedia.org/wiki/Watchdog_timer>`__ permaneciendo demasiado tiempo en un bucle o procesando alguna tarea, por lo que los procesos vitales como la comunicación Wi-Fi no pueden correr.

Por favor, compruebe a continuación cómo reconocer los escenarios `excepción <#excepción>`__ y `watchdog <#watchdog>`__ y qué hacer al respecto.

Excepción
^^^^^^^^^

El reinicio típico debido una excepción, se ve como a continuación:

.. figure:: pictures/a02-exception-cause-decoding.png
   :alt: Decodificación de la causa de excepción

   Decodificación de la causa de excepción

Comienza buscando el código de excepción en la tabla `Causas de excepción (EXCCAUSE) <../exception_causes.rst>`__ para comprender qué tipo de problema tienes. Si no tienes pistas de qué se trata y dónde ocurre, utiliza el `Decodificador del volcado de pila de excepciones para Arduino ESP8266/ESP3226 <https://github.com/me-no-dev/EspExceptionDecoder>`__ para averiguar en qué línea de la aplicación se desencadena. Consulta `Comprobar dónde se bloquea el código <#check-where-the-code-crashes>`__ mas adelante para ver un ejemplo rápido de cómo hacerlo.

Watchdog (Perro Guardián)
^^^^^^^^^^^^^^^^^^^^^^^^^

ESP proporciona dos temporizadores de vigilancia (WDT) que observan el bloqueo de la aplicación.

- **Software Watchdog** - proporcionado por el `SDK <http://bbs.espressif.com/viewforum.php?f=46>`__, forma parte del core  `ESP8266/Arduino <https://github.com/esp8266/Arduino> `__ cargado en el módulo junto a su aplicación.
- **Hardware Watchdog** - construido dentro del hardware ESP8266 y actúa si el watchdog software está desactivado durante demasiado tiempo, en caso de que falle o si no se proporciona en absoluto.

El reinicio por watchdog está claramente identificado por el ESP en el Monitor Seri.

A continuación, se muestra un ejemplo de bloqueo de la aplicación provocada por el software wdt.

.. figure:: pictures/a02-sw-watchdog-example.png
   :alt: Ejemplo de reinicio debido a watchdog software

   Ejemplo de reinicio debido a watchdog software

El reinicio debido a software watchdog es generalmente mas facil de resolver porque el registro incluye el volcado de pila. El volcado de pila puede usarse para encontrar la linea en particular en el código donde salta el WDT.

A continuación se muestra un reset debido al watchdog hardware.

.. figure:: pictures/a02-hw-watchdog-example.png
   :alt: Example of restart by h/w watchdog

   Example of restart by h/w watchdog

Hardware wdt is the last resort of ESP to tell you that application is locked up (if s/w wdt timer is disabled or not working).

Please note that for restarts initialized by h/w wdt, there is no stack trace to help you identify the place in code where the lockup has happened. In such case, to identify the place of lock up, you need to rely on debug messages like ``Serial.print`` distributed across the application. Then by observing what was the last debug message printed out before restart, you should be able to narrow down part of code firing the h/w wdt reset. If diagnosed application or library has debug option then switch it on to aid this troubleshooting.

Comprueba donde se bloquea el código
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Decoding of ESP stack trace is now easy and available to everybody thanks to great `Arduino ESP8266/ESP32 Exception Stack Trace
Decoder <https://github.com/me-no-dev/EspExceptionDecoder>`__ developed by @me-no-dev.

Installation for Arduino IDE is quick and easy following the `installation <https://github.com/me-no-dev/EspExceptionDecoder#installation>`__
instructions.

If you don't have any code for troubleshooting, use the example below:

::

    void setup()
    {
      Serial.begin(115200);
      Serial.println();
      Serial.println("Let's provoke the s/w wdt firing...");
      //
      // wait for s/w wdt in infinite loop below
      //
      while(true);
      //
      Serial.println("This line will not ever print out");
    }

    void loop(){}

Upload this code to your ESP (Ctrl+U) and start Serial Monitor (Ctrl+Shift+M). You should shortly see ESP restarting every couple of seconds and ``Soft WDT reset`` message together with stack trace showing up on each restart. Click the Autoscroll check-box on Serial Monitor to stop the messages scrolling up. Select and copy the stack trace, go to the *Tools* and open the *ESP Exception Decoder*.

.. figure:: pictures/a02-decode-stack-tace-1-2.png
   :alt: Decode the stack trace, steps 1 and 2

   Decode the stack trace, steps 1 and 2

Now paste the stack trace to Exception Decoder's window. At the bottom of this window you should see a list of decoded lines of sketch you have just uploaded to your ESP. On the top of the list, like on the top of the stack trace, there is a reference to the last line executed just before the software watchdog timer fired causing the ESP's restart. Check the number of this line and look it up on the sketch. It should be the line ``Serial.println("Let's provoke the s/w wdt firing...")``, that happens to be just before ``while(true)`` that made the watchdog fired (ignore the lines with comments, that are discarded by compiler).

.. figure:: pictures/a02-decode-stack-tace-3-6.png
   :alt: Decode the stack trace, steps 3 through 6

   Decode the stack trace, steps 3 through 6

Armed with `Arduino ESP8266/ESP32 Exception Stack Trace Decoder <https://github.com/me-no-dev/EspExceptionDecoder>`__ you can track down where the module is crashing whenever you see the stack trace dropped. The same procedure applies to crashes caused by exceptions.

    Note: To decode the exact line of code where the application crashed, you need to use ESP Exception Decoder in context of sketch you have just loaded to the module for diagnosis. Decoder is not able to correctly decode the stack trace dropped by some other application not compiled and loaded from your Arduino IDE.


Otras causas de bloqueo
~~~~~~~~~~~~~~~~~~~~~~~~

Interrupt Service Routines
   By default, all functions are compiled into flash, which means that the cache may kick in for that code. However, the cache currently can't be used during hardware interrupts. That means that, if you use a hardware ISR, such as attachInterrupt(gpio, myISR, CHANGE) for a GPIO change, the ISR must have the ICACHE_RAM_ATTR attribute declared. Not only that, but the entire function tree called from the ISR must also have the ICACHE_RAM_ATTR declared. Be aware that every function that has this attribute reduces available memory.

   In addition, it is not possible to execute delay() or yield() from an ISR, or do blocking operations, or operations that disable the interrupts, e.g.: read a DHT.

   Finally, an ISR has very high restrictions on timing for the executed code, meaning that executed code should not take longer than a very few microseconds. It is considered best practice to set a flag within the ISR, and then from within the loop() check and clear that flag, and execute code.

Asynchronous Callbacks
   Asynchronous CBs, such as for the Ticker or ESPAsync* libs, have looser restrictions than ISRs, but some restrictions still apply. It is not possible to execute delay() or yield() from an asynchronous callback. Timing is not as tight as an ISR, but it should remain below a few milliseconds. This is a guideline. The hard timing requirements depend on the WiFi configuration and amount of traffic. In general, the CPU must not be hogged by the user code, as the longer it is away from servicing the WiFi stack, the more likely that memory corruption can happen.

Memory, memory, memory
   Running out of heap is the most common cause for crashes. Because the build process for the ESP leaves out exceptions (they use memory), memory allocations that fail will do so silently. A typical example is when setting or concatenating a large String. If allocation has failed internally, then the internal string copy can corrupt data, and the ESP will crash.

   In addition, doing many String concatenations in sequence, e.g.: using operator+() multiple times, will cause memory fragmentation. When that happens, allocations may silently fail even though there is enough total heap available. The reason for the failure is that an allocation requires finding a single free memory block that is large enough for the size being requested. A sequence of String concatenations causes many allocations/deallocations/reallocations, which makes "holes" in the memory map. After many such operations, it can happen that all available holes are too small to comply with the requested size, even though the sum of all holes is greater than the requested size.

   So why do these silent failures exist? On the one hand, there are specific interfaces that must be adhered to. For example, the String object methods don't allow for error handling at the user application level (i.e.: no old-school error returns). On the other hand, some libraries don't have the allocation code accessible for modification. For example, std::vector is available for use. The standard implementations rely on exceptions for error handling, which is not available for the ESP, and in any case there is no access to the underlying code.

*Some techniques for reducing memory usage*

   * Don't use const char * with literals. Instead, use const char[] PROGMEM. This is particularly true if you intend to, e.g.: embed html strings.
   * Don't use global static arrays, such as uint8_t buffer[1024]. Instead, allocate dynamically. This forces you to think about the size of the array, and its scope (lifetime), so that it gets released when it's no longer needed. If you are not certain about dynamic allocation, use std libs (e.g.: std:vector, std::string), or smart pointers. They are slightly less memory efficient than dynamically allocating yourself, but the provided memory safety is well worth it.
   * If you use std libs like std::vector, make sure to call its ::reserve() method before filling it. This allows allocating only once, which reduces mem fragmentation, and makes sure that there are no empty unused slots left over in the container at the end.

Stack
   The amount of stack in the ESP is tiny at only 4KB. For normal developement in large systems, it is good practice to use and abuse the stack, because it is faster for allocation/deallocation, the scope of the object is well defined, and deallocation automatically happens in reverse order as allocation, which means no mem fragmentation. However, with the tiny amount of stack available in the ESP, that practice is not really viable, at least not for big objects.
      * Large objects that have internally managed memory, such as String, std::string, std::vector, etc, are ok on the stack, because they internally allocate their buffers on the heap.
      * Large arrays on the stack, such as uint8_t buffer[2048] should be avoided on the stack and be dynamically allocated (consider smart pointers).
      * Objects that have large data members, such as large arrays, should be avoided on the stack, and be dynamicaly allocated (consider smart pointers).


Contra la pared, abre un Issue
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using the procedure above you should be able to troubleshoot all the code you write. It may happen that ESP is crashing inside some library or code you are not familiar enough to troubleshoot. If this is the case then contact the application author by writing an issue report.

Follow the guidelines on issue reporting that may be provided by the author of code in his / her repository.

If there are no guidelines, include in your report the following:

-  [ ] Exact steps by step instructions to reproduce the issue
-  [ ] Your exact hardware configuration including the schematic
-  [ ] If the issue concerns standard, commercially available ESP board with power supply and USB interface, without extra h/w attached, then provide just the board type or link to description
-  [ ] Configuration settings in Arduino IDE used to upload the application
-  [ ] Error log & messages produced by the application (enable debugging for more details)
-  [ ] Decoded stack trace
-  [ ] Copy of your sketch
-  [ ] Copy of all the libraries used by the sketch
-  [ ] If you are using standard libraries available in Library Manager, then provide just version numbers
-  [ ] Version of `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__ core
-  [ ] Name and version of your programming IDE and O/S

With plenty of ESP module types available, several versions of libraries or `esp8266 / Arduino <https://github.com/esp8266/Arduino>`__ core, types and versions of O/S, you need to provide exact information what your application is about. Only then people willing to look into your issue may be able to refer it to configuration they have. If you are lucky, they may even attempt to reproduce your issue on their equipment. This will be far more difficult if you are providing only vague details, so somebody would need to ask you to find out what is really happening.

On the other hand if you flood your issue report with hundreds lines of code, you may also have difficulty to find somebody willing to analyze it. Therefore reduce your code to the bare minimum that is still causing the issue. It will help you as well to isolate the issue and pin done the root cause.

Conclusión
~~~~~~~~~~

Do not be afraid to troubleshoot ESP exception and watchdog restarts. `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__ core provides detailed diagnostics that will help you pin down the issue. Before checking the s/w, get your h/w right. Use `ESP Exception Decoder <https://github.com/me-no-dev/EspExceptionDecoder>`__ to find out where the code fails. If you do you homework and still unable to identify the root cause, enter the issue report. Provide enough details. Be specific and isolate the issue. Then ask community for support. There are plenty of people that like to work with ESP and willing to help with your problem.

`FAQ :back: <readme.rst>`__
