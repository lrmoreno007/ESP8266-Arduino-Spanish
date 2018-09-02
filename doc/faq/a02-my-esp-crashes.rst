Mi ESP se bloquea al correr el programa. ¿Como lo resuelvo?
---------------------------------------------------------

-  `Introducción <#introducción>`__
-  `Que debe decir el ESP <#que-debe-decir-el-esp>`__
-  `Pon tu hardware correctamente <#pon-tu-hardware-correctamente>`__
-  `¿Cual es la causa del reinicio? <#cual-es-la-causa-del-reinicio>`__
-  `Excepción <#excepción>`__
-  `Watchdog (Perro Guardián) <#watchdog-perro-guardián>`__
-  `Decodificador de excepciones <#decodificador-de-excepciones>`__
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

En cualquier caso, asegúrese de que su módulo pueda ejecutar sketch de ejemplo estándar stable que establezcan una conexión WiFi, como p. ej. `HelloServer.ino <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer/examples/HelloServer>`__.

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

- **Software Watchdog** - proporcionado por el `SDK <http://bbs.espressif.com/viewforum.php?f=46>`__, forma parte del core  `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__ cargado en el módulo junto a su aplicación.
- **Hardware Watchdog** - construido dentro del hardware ESP8266 y actúa si el watchdog software está desactivado durante demasiado tiempo, en caso de que falle o si no se proporciona en absoluto.

El reinicio por watchdog está claramente identificado por el ESP en el Monitor Seri.

A continuación, se muestra un ejemplo de bloqueo de la aplicación provocada por un WDT software.

.. figure:: pictures/a02-sw-watchdog-example.png
   :alt: Ejemplo de reinicio debido a watchdog software

   Ejemplo de reinicio debido a watchdog software

El reinicio debido a software watchdog es generalmente mas fácil de resolver porque el registro incluye el volcado de pila. El volcado de pila puede usarse para encontrar la línea en particular en el código donde salta el WDT.

A continuación se muestra un reset debido al WDT hardware.

.. figure:: pictures/a02-hw-watchdog-example.png
   :alt: Ejemplo de reset debido a watchgog hardware

   Ejemplo de reset debido a watchdog hardware

El WDT hardware es el último recurso del ESP para decirte que la aplicación está bloqueada (si el temporizador WDT software está desactivado o no funciona).

Ten en cuenta que para los reinicios iniciados por WDT hardware, no hay un volcado de pila que te ayude a identificar el lugar en el código donde ocurrió el bloqueo. En tal caso, para identificar el lugar de bloqueo, debes confiar en los mensajes de depuración como ``Serial.print`` distribuidos en la aplicación. Luego, al observar cuál fue el último mensaje de depuración antes de reiniciar, deberías poder reducir parte del código que disparó el reinicio WDT hardware. Si la aplicación diagnosticada o la librería tiene una opción de depuración, enciéndela para ayudar en la solución de problemas.

Decodificador de excepciones
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La decodificación del volcado de pila del ESP es ahora mas fácil y está disponible para todos gracias al gran `Arduino ESP8266/ESP32 Exception Stack Trace Decoder <https://github.com/me-no-dev/EspExceptionDecoder>`__ desarrollado por @me-no-dev.

La instalación en el IDE Arduino es rápida y sencilla siguiendo las instrucciones de `instalación <https://github.com/me-no-dev/EspExceptionDecoder#installation>`__.

Si no tienes ningún sketch que genere un WDT para intentar solucionarlo, usa el siguiente ejemplo:

::

    void setup()
    {
      Serial.begin(115200);
      Serial.println();
      Serial.println("Vamos a provocar el disparo del WDT...");
      
      // provoca un OOM (Out of memory - Fuera de memoria), será grabado como el último ocurrido
      char* out_of_memory_failure = (char*)malloc(1000000);
      //
      // Espera el WDT debido al siguiente bucle infinito
      //
      while(true);
      //
      Serial.println("Esta línea no debe verse nunca");
    }

    void loop(){}

Activa la opción Out-Of-Memory (*OOM*) en el menú de debug (en el menú *Herramientas > Debug Level*), compila y sube este código a tu ESP (Ctrl+U) e inicia el Monitor Serie (Ctrl+Shift+M). Deberías ver en breve al ESP reiniciando cada dos segundos y el mensaje `Soft WDT reset`` junto con el volcado de pila en cada reinicio. Desactiva la casilla de verificación Autoscroll en Monitor Serie para detener el desplazamiento de los mensajes. Selecciona y copia, incluyendo la linea ``last failed alloc call: ...``, ve a *Herramientas* y abre *ESP Exception Decoder*.

.. figure:: pictures/a02-decode-stack-tace-1-2.png
   :alt: Decodifica el volcado de pila, pasos 1 y 2

   Decodifica el volcado de pila, pasos 1 y 2

Ahora pega el volcado de pila en la ventana del decodificador de excepciones. En la parte inferior de esta ventana, deberías ver una lista de líneas decodificadas del boceto que acabas de cargar en tu ESP. En la parte superior de la lista, como en la parte superior del volcado de pila, hay una referencia a la última línea ejecutada justo antes de que se disparara el temporizador de WDT software, lo que provocó el reinicio del ESP. Verifique el número de esta línea y búsquela en el boceto. Debería ser la línea ``Serial.println("Vamos a provocar el disparo del WDT...")``, que está justo antes de ``while(true)`` el cual hizo que watchdog se disparara (ignorar las líneas con comentarios, son descartadas por el compilador).

.. figure:: pictures/a02-decode-stack-tace-3-6.png
   :alt: Decodifica el volcado de pila, pasos del 3 al 6

   Decodifica el volcado de pila, pasos del 3 al 6

Armado con `Arduino ESP8266/ESP32 Exception Stack Trace Decoder <https://github.com/me-no-dev/EspExceptionDecoder>`__ puedes rastrear dónde se bloquea el módulo cada vez que se genere un volcado de pila. El mismo procedimiento se aplica a los bloqueos causados por excepciones.

    Nota: Para decodificar la línea exacta de código donde se colgó la aplicación, debes usar el Decodificador de Excepciones ESP en el contexto del sketch que acabas de cargar en el módulo para el diagnóstico. El decodificador no puede decodificar correctamente el volcado de pila generado por alguna otra aplicación no compilada y cargada desde su IDE de Arduino.

Otras causas de bloqueo
~~~~~~~~~~~~~~~~~~~~~~~~

Rutinas de servicio de interrupción (ISR)
   Por defecto, todas las funciones se compilan en la flash, lo que significa que la memoria caché puede activarse para ese código. Sin embargo, la memoria caché actualmente no se puede usar durante las interrupciones de hardware. Eso significa que, si utilizas un ISR hardware, como ``attachInterrupt(gpio, myISR, CHANGE)`` para un cambio de GPIO, el ISR debe tener el atributo ``ICACHE_RAM_ATTR`` declarado. No solo eso, sino que todo el árbol de funciones llamado desde el ISR también debe tener el ``ICACHE_RAM_ATTR`` declarado. Ten en cuenta que cada función que tiene este atributo reduce la memoria disponible.

   Además, no es posible ejecutar ``delay()`` o ``yield()`` desde un ISR, o realizar operaciones de bloqueo, o operaciones que deshabilitan las interrupciones, por ejemplo, leer un DHT.

   Finalmente, un ISR tiene restricciones muy altas en el tiempo para el código ejecutado, lo que significa que el código ejecutado no debería tomar más de unos pocos microsegundos. Se considera mejor práctica establecer un flag dentro del ISR y luego desde dentro del loop() verificar y borrar ese flag, y ejecutar el código.

Callbacks asíncronos
   Los CB asíncronos, como los de Ticker o ESPAsync* libs, tienen restricciones más flexibles que los ISR, pero también se aplican algunas restricciones. No es posible ejecutar ``delay()`` o ``yield()`` desde un callback asíncrono. El tiempo no está tan ajustado como en un ISR, pero debe permanecer por debajo de unos pocos milisegundos. Esta es una guía. Los requisitos de tiempo difíciles dependen de la configuración WiFi y la cantidad de tráfico. En general, la CPU no debe ser acaparada por el código de usuario, ya que mientras más tiempo esté sin atender la pila de WiFi, es más probable que la corrupción de la memoria pueda producirse.

Memoria, memoria, memoria
   Quedarse sin pila es la **causa más común del bloqueo**. Debido a que el proceso de compilación para el ESP deja de lado las excepciones (usan memoria), las asignaciones de memoria que fallan lo harán en silencio. Un ejemplo típico es cuando se establece o concatena una cadena grande. Si la asignación ha fallado internamente, la copia de cadena interna puede dañar los datos y el ESP se bloqueará.
   
Además, al realizar muchas concatenaciones de cadenas en secuencia, por ejemplo, al usar el operador+() varias veces, se producirá la fragmentación de la memoria. Cuando eso sucede, las asignaciones pueden fallar silenciosamente a pesar de que hay suficiente pila total disponible. El motivo del fallo es que una asignación requiere encontrar un único bloque de memoria libre que sea lo suficientemente grande para el tamaño que se solicita. Una secuencia de concatenaciones de cadenas causa muchas asignaciones/deasignaciones/reasignaciones, que hacen "agujeros" en el mapa de memoria. Después de muchas operaciones de este tipo, puede suceder que todos los agujeros disponibles sean demasiado pequeños para cumplir con el tamaño solicitado, aunque la suma de todos los agujeros sea mayor que el tamaño solicitado.

Entonces, ¿por qué existen estos fallos silenciosos? Por un lado, hay interfaces específicos que deben cumplirse. Por ejemplo, los métodos del objeto String no permiten el manejo de errores a nivel de aplicación de usuario (es decir, no se devuelve ningún error de la vieja escuela). Por otro lado, algunas librerías no tienen el código de asignación accesible para su modificación. Por ejemplo, std::vector está disponible para su uso. Las implementaciones estándar se basan en excepciones para el manejo de errores, que no están disponibles para el ESP y en cualquier caso no hay acceso al código subyacente.

Instrumentar el código con la opción de depuración OOM y las llamadas a ``ESP.getFreeHeap()`` ayudarán al proceso de búsqueda de fugas. Ahora es el momento de volver a leer sobre el `decodificador de excepción <# exception-decoder>`__.

*Algunas técnicas para reducir el uso de memoria*

* No uses ``const char*`` con literales. En su lugar, utiliza const char[] PROGMEM. Esto es particularmente cierto si tienes la intención, por ejemplo de incrustar cadenas HTML.

* No utilices matrices estáticas globales, como uint8_t buffer [1024]. En cambio, asigna dinámicamente. Esto te obliga a pensar en el tamaño de la matriz y su alcance (duración), para que se libere cuando ya no se necesite. Si no estás seguro acerca de la asignación dinámica, usa std libs (por ejemplo, std:vector, std::string) o punteros inteligentes. Son ligeramente menos eficientes en cuanto a la memoria que asignarlo dinámicamente tu mismo, pero la seguridad de la memoria proporcionada vale la pena.

* Si usas std libs como std::vector, asegúrate de llamar a su método ::reserve() antes de completarlo. Esto permite asignar solo una vez, lo que reduce la fragmentación de la memoria y garantiza que no queden espacios vacíos sin usar en el contenedor al final.

Apilado
   La cantidad de pila en el ESP es pequeña, solo 4 KB. Para un desarrollo normal en sistemas grandes, es una buena práctica usar y abusar de la pila, porque es más rápido para la asignación/deasignación, el alcance del objeto está bien definido y la deasignación ocurre automáticamente en orden inverso a la asignación, lo que significa que no hay fragmentación de las memoria. Sin embargo, con una pequeña cantidad de pila disponible en ESP, esa práctica no es realmente viable, al menos no para objetos grandes.
   
* Los objetos grandes que tienen memoria administrada internamente, como String, std::string, std::vector, etc., están bien en la pila, porque asignan internamente sus búferes en el montón.
      
* Las grandes matrices, como uint8_t buffer[2048] deben evitarse en la pila y asignarse dinámicamente (considere los punteros inteligentes).
      
* Los objetos que tienen grandes datos, como grandes matrices, deben evitarse en la pila y asignarse dinámicamente (considere los punteros inteligentes).
      
Contra la pared, abre un Issue
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Usando el procedimiento anterior, debería poder solucionar todos los códigos que escriba. Puede suceder que ESP se bloquee dentro de una librería o código que no esté familiarizado a solucionarlo. Si este es el caso, póngase en contacto con el autor de la aplicación escribiendo un informe de problema.

Siga las pautas sobre informes de problemas que pueda proporcionar el autor del código en su repositorio.

Si no hay pautas, incluya en su informe lo siguiente:

- [ ] Instrucciones exactas paso a paso para reproducir el problema

- [ ] Su configuración exacta de hardware, incluido el esquema

- [ ] Si el problema se refiere a una tarjeta ESP estándar disponible en el mercado con fuente de alimentación e interfaz USB, sin hardware adicional conectado, proporcione solo el tipo de placa o enlace a la descripción

- [ ] Configuración y ajustes en el IDE Arduino utilizados para cargar la aplicación

- [ ] Registro de errores y mensajes producidos por la aplicación (habilite la depuración para obtener más detalles)

- [ ] Desglose de la pila descifrada

- [ ] Copia de tu sketch

- [ ] Copia de todas las librerías utilizadas por el sketch

- [ ] Si está utilizando librerías estándar disponibles en el Gestor de librerías, proporcione solo los números de la versión

- [ ] Versión del core `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__

- [ ] Nombre y versión de su programador IDE y sistema operativo

Existen muchos tipos de módulos ESP disponibles, varias versiones de librerías o cores `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__, tipos y versiones de SO, debes proporcionar información exacta sobre su aplicación. Solo entonces las personas dispuestas a analizar su problema pueden referirlo a la configuración que tienes. Si tienes suerte, incluso pueden intentar reproducir tu problema en sus equipos. Esto será mucho más difícil si proporcionas solo detalles vagos, por lo que alguien debería pedirte que averigües qué está sucediendo realmente.

Por otro lado, si inundas tu informe de problemas con cientos de líneas de código, también puedes tener dificultades para encontrar a alguien dispuesto a analizarlo. Por lo tanto, reduce tu código al mínimo que sigue causando el problema. También te ayudará a aislar el problema y fijar la raíz del mismo.

Conclusión
~~~~~~~~~~

No tengas miedo de solucionar problemas de excepción del ESP o reinicios por watchdog. El core `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__ proporciona diagnósticos detallados que te ayudarán a determinar el problema. Antes de verificar el software, asegúrate de tener perfectamente el hardware. Utiliza el `ESP Exception Decoder <https://github.com/me-no-dev/EspExceptionDecoder>`__ para averiguar dónde falla el código. Si haces tus tarea y aún así no puedes identificar la causa raíz, haz un informe de problema. Proporciona detalles suficientes. Se específico y aísla el problema. Luego pide ayuda a la comunidad. Hay muchas personas a las que les gusta trabajar con ESP y están dispuestas a ayudarte con tu problema.

`FAQ :back: <readme.rst>`__
