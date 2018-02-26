Librería ESP8266WiFi
===================

Todo sobre el WiFi de ESP8266. Si está ansioso por conectar su nuevo módulo ESP8266 a la red WiFi para comenzar a enviar y recibir datos, este es un buen lugar para comenzar. Si está buscando detalles más en profundidad sobre cómo programar una funcionalidad de red WiFi específica, también se encuentra en el lugar correcto.

Introducción
------------

La `librería WiFi para ESP8266 <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ ha sido desarrollada basándose en el `SDK de ESP8266 <http://bbs.espressif.com/viewtopic.php?f=51&t=1023>`__, usando nombres convencionales y la filosofía de funcionalidades generales de la `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__. Con el tiempo, la riqueza de las funciones WiFi del SDK de ESP8266 pasadas a `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__ superan a la  `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__ y se hizo evidente que tenemos que proporcionar documentación por separado sobre lo que es nuevo y extra.

Esta documentación lo guiará a través de varias clases, métodos y propiedades de la librería `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__. Si eres nuevo en C++ y Arduino, no te preocupes. Comenzaremos por conceptos generales y luego pasaremos a la descripción detallada de los miembros de cada clase en particular, incluidos los ejemplos de uso.

El alcance de la funcionalidad que ofrece la biblioteca `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ es bastante extensa y por lo tanto, esta descripción se ha dividido en documentos separados marcados con :arrow_right:.

Comienzo rápido
~~~~~~~~~~~

Esperamos que ya esté familiarizado con la carga del sketch `Blink.ino <https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/Blink.bin>`__ en el módulo ESP8266 y obtenga el LED parpadeando. De lo contrario, compruebe `este tutorial <https://learn.adafruit.com/adafruit-huzzah-esp8266-breakout/using-arduinoide>`__ de Adafruit o este `otro gran tutorial <https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/introduction>`__ desarrollado por Sparkfun.

Para conectar el módulo ESP al WiFi (como conectar un teléfono móvil a un punto caliente), solo necesita un par de líneas de código:

.. code:: cpp

    #include <ESP8266WiFi.h>

    void setup()
    {
      Serial.begin(115200);
      Serial.println();

      WiFi.begin("nombre-red", "contraseña-red");
      
      Serial.print("Conectando");
      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }
      Serial.println();
      
      Serial.print("Conectado, dirección IP: ");
      Serial.println(WiFi.localIP());
    }

    void loop() {}

En la línea ``WiFi.begin("nombre-red", "contraseña-red")`` reemplace ``nombre-red`` y ``contraseña-red`` con el nombre y contraseña a la red WiFi que quiere conectarse. Entonces suba el sketch al módulo ESP y abra el Monitor Serie. Deberías ver algo como:

.. figure:: pictures/wifi-simple-connect-terminal.png
   :alt: Registro de conexión en el Monitor Serie del IDE Arduino

   Registro de conexión en el Monitor Serie del IDE Arduino

¿Como funciona? En la primera línea del boceto ``#include <ESP8266WiFi.h>`` estamos incluyendo la librería `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__. Esta librería proporciona las rutinas específicas WiFi de ESP8266 a las que llamamos para conectarse a la red.

La conexión real a WiFi se inicia llamando al:

.. code:: cpp

      WiFi.begin("nombre-red", "contraseña-red");

El proceso de conexión puede demorar unos segundos, así que comprobamos que esto se complete en el siguiente ciclo:

.. code:: cpp

      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }

El bucle ``while()`` seguirá en bucle mientras ``WiFi.status()`` no es ``WL_CONNECTED``. El ciclo saldrá solo si el estado cambia a ``WL_CONNECTED``.

La última línea imprimirá la dirección IP asignada al módulo ESP por `DHCP <http://whatismyipaddress.com/dhcp>`__:

.. code:: cpp

    Serial.println(WiFi.localIP());

Si no ve la última línea sino solo más y más puntos ``......... ``, entonces probablemente el nombre o la contraseña de la red WiFi en el sketch son incorrectos. Verifique el nombre y la contraseña conectando desde cero a esta WiFi una PC o un teléfono móvil.

*Nota:* si la conexión se establece y luego se pierde por algún motivo, ESP se reconectará automáticamente al último punto de acceso utilizado una vez que vuelva a estar en línea. Esto se hará automáticamente mediante la librería WiFi, sin intervención del usuario.

Eso es todo lo que necesita para conectar su ESP8266 al WiFi. En los siguientes capítulos, explicaremos qué cosas interesantes se pueden hacer con ESP una vez conectados.

Quien es quien
~~~~~~~~~~

Los dispositivos que se conectan a la red WiFi se llaman estaciones (STA). La conexión a WiFi es proporcionada por un punto de acceso (AP), que actúa como un centro para una o más estaciones. El punto de acceso en el otro extremo está conectado a una red cableada. Un punto de acceso generalmente se integra con un router para proporcionar acceso desde la red WiFi a Internet. Cada punto de acceso es reconocido por un SSID (**S**\ervice **S**\et **ID**\entifier), que esencialmente es el nombre de la red que usted selecciona cuando conecta un dispositivo (estación) al WiFi.

El módulo ESP8266 puede funcionar como una estación, por lo que podemos conectarlo a la red WiFi. Y también puede funcionar como un punto de acceso wireless (SoftAP), para establecer su propia red WiFi. Por lo tanto, podemos conectar otras estaciones a dicho módulo ESP. ESP8266 también puede operar tanto en modo estación como en modo punto de acceso. Esto proporciona la posibilidad de construir, p. ej. `redes de malla <https://en.wikipedia.org/wiki/Mesh_networking>`__.

.. figure:: pictures/esp8266-station-soft-access-point.png
   :alt: ESP8266 operando en modo Estación + Punto de Acceso

   ESP8266 operando en modo Estación + Punto de Acceso

La biblioteca `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ proporciona una amplia colección de `métodos C++ <https://es.wikipedia.org/wiki/M%C3%A9todo_(inform%C3%A1tica)>`__ y `propiedades o atributos <https://es.wikipedia.org/wiki/Atributo_(inform%C3%A1tica)>`__ para configurar y operar un módulo ESP8266 en modo estación y/o punto de acceso. Se describen en los siguientes capítulos.

Descripción de la clase
-----------------

La librería `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ se divide en varias clases. En la mayoría de los casos, al escribir el código, el usuario no está interesado en esta clasificación. Lo usamos para dividir la descripción de esta librería en piezas más manejables.

.. figure:: pictures/doxygen-class-index.png
   :alt: Índice de clases de la librería ESP8266WiFi

   Índice de clases de la librería ESP8266WiFi

Los siguientes capítulos describen todas las llamadas a funciones ( `métodos <https://es.wikipedia.org/wiki/M%C3%A9todo_(inform%C3%A1tica)>`__ y `propiedades <https://es.wikipedia.org/wiki/Atributo_(inform%C3%A1tica)>`__ en términos C++) enumerados en clases particulares de `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__. La descripción se ilustra con ejemplos de aplicaciones y fragmentos de código para mostrar cómo usar las funciones en la práctica. La mayoría de esta información se divide en documentos separados. Por favor, sigue para acceder a ellos.

Station
~~~~~~~

El modo estación (STA) se utiliza para conectar el módulo ESP a una red WiFi establecida por un punto de acceso.

.. figure:: pictures/esp8266-station.png
   :alt: ESP8266 operando en modo estación

   ESP8266 operando en modo estación

La clase de estación tiene varias características para facilitar la administración de la conexión WiFi. En caso de que se pierda la conexión, el ESP8266 se volverá a conectar automáticamente al último punto de acceso utilizado, una vez que esté nuevamente disponible. Lo mismo ocurre en el reinicio del módulo. Esto es posible ya que ESP guarda las credenciales al último punto de acceso utilizado en la memoria flash (no volátil). Usando los datos guardados, ESP también se volverá a conectar si se modificó el sketch, si el código no altera el modo WiFi o las credenciales.

:doc:`Documentación clase Station <station-class>`

Echa un vistazo a la sección separada con :doc:`ejemplos <station-examples>`.

Punto de Acceso Wireless
~~~~~~~~~~~~~~~~~

Un `punto de acceso inalámbrico (AP) <https://es.wikipedia.org/wiki/Punto_de_acceso_inal%C3%A1mbrico>`__ es un dispositivo que proporciona acceso a la red WiFi a otros dispositivos (estaciones) y los conecta a una red cableada. ESP8266 puede proporcionar una funcionalidad similar, excepto que no tiene interfaz para una red cableada. Tal modo de operación se llama punto de acceso SoftAP. La cantidad máxima de estaciones conectadas al SoftAP es de cinco.

.. figure:: pictures/esp8266-soft-access-point.png
   :alt: ESP8266 operando en modo Punto de acceso SoftAP

   ESP8266 operando en modo Punto de acceso SoftAP

El modo SoftAP se usa a menudo y es un paso intermedio antes de conectar ESP a una red WiFi en modo estación. Esto es cuando el SSID y la contraseña de dicha red no se conocen por adelantado. ESP primero arranca en modo SoftAP, para que podamos conectarnos a él usando un ordenador portátil o un teléfono móvil. Luego, podemos proporcionar credenciales a la red objetivo. Una vez hecho esto, ESP se cambia al modo estación y se puede conectar al WiFi objetivo.

Otra aplicación práctica del modo SoftAP es configurar una `red mallada <https://es.wikipedia.org/wiki/Red_inal%C3%A1mbrica_mallada>`__. ESP puede funcionar tanto en modo SoftAP como en modo Estación para que pueda actuar como un nodo de una red mallada.

:doc:`Documentación clase Soft Access Point <soft-access-point-class>`

Echa un vistazo a la sección separada con :doc:`ejemplos <soft-access-point-examples>`.

Scan
~~~~

Para conectar un teléfono móvil a un punto de acceso público, normalmente abre la aplicación de configuración de Wi-Fi, enumera las redes disponibles y elige el punto de acceso que necesita. Luego ingresa una contraseña (o no) y estás dentro. Puedes hacer lo mismo con ESP. La clase de escaneo implementa la funcionalidad del escaneo y la lista de redes disponibles en el rango.

:doc:`Documentación clase Scan <scan-class>`.

Echa un vistazo a la sección separada con :doc:`ejemplos <scan-examples>`.

Client
~~~~~~

La clase Client crea `clientes <https://es.wikipedia.org/wiki/Cliente_(inform%C3%A1tica)>`__ que puede acceder a servicios proporcionados por `servidores <https://es.wikipedia.org/wiki/Servidor>`__ para enviar, recibir y procesar datos.

.. figure:: pictures/esp8266-client.png
   :alt: ESP8266 operando como Cliente

   ESP8266 operando como Cliente

Echa un vistazo a la sección separada con :doc:`ejemplos <client-examples>` / :doc:`Lista de funciones <client-class>`

Client Secure
~~~~~~~~~~~~~

Client Secure es una extensión de la `clase Client <#client>`__ donde la conexión y el intercambio de datos con los servidores se hace usando un `protocolo seguro <https://es.wikipedia.org/wiki/Transport_Layer_Security>`__. Es compatible con `TLS 1.1 <https://es.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.1>`__. El `TLS 1.2 <https://es.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.2>`__ no es compatible.

.. figure:: pictures/esp8266-client-secure.png
   :alt: ESP8266 operando como Cliente seguro

   ESP8266 operando como Cliente seguro

Las aplicaciones seguras tienen una sobrecarga adicional de memoria (y procesamiento) debido a la necesidad de ejecutar algoritmos de criptografía. Cuanto más fuerte sea la clave del certificado, más gastos generales se necesitan. En la práctica, no es posible ejecutar más de un único cliente seguro a la vez. El problema se refiere a la memoria RAM que no podemos agregar, el tamaño de la memoria flash por lo general no es el problema. Si desea aprender cómo se ha desarrollado `la librería de Client Secure <https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/src/WiFiClientSecure.h>`__, qué servidores se han probado y cómo se han superado las limitaciones de la memoria, lea el fascinante informe de problemas `#43 <https://github.com/esp8266/Arduino/issues/43>`__.

Echa un vistazo a la sección separada con :doc:`ejemplos <client-secure-examples>` / :doc:`lista de funciones <client-secure-class>`

Server
~~~~~~

La clase de Server crea `Servidores <https://es.wikipedia.org/wiki/Servidor>`__ que proporcionan funcionalidad a otros programas o dispositivos, llamados `Clientes <https://es.wikipedia.org/wiki/Cliente_(inform%C3%A1tica)>`__.

.. figure:: pictures/esp8266-server.png
   :alt: ESP8266 operando como Servidor

   ESP8266 operando como Servidor

Los clientes se conectan para enviar y recibir datos y acceder a la funcionalidad provista.

Echa un vistazo a la sección separada con :doc:`ejemplos <server-examples>` / :doc:`lista de funciones <server-class>`.

UDP
~~~

La clase UDP permite el envío y recepción de mensajes `User Datagram Protocol (UDP) <https://es.wikipedia.org/wiki/User_Datagram_Protocol>`__. El UDP usa un modelo de transmisión simple de "disparar y olvidar" sin garantía de entrega, pedido o protección duplicada. UDP proporciona sumas de comprobación para la integridad de datos y números de puertos para direccionar diferentes funciones a la fuente y el destino del datagrama.

Echa un vistazo a la sección separada con :doc:`ejemplos <udp-examples>` / :doc:`lista de funciones <udp-class>`.

Generic
~~~~~~~

Hay varias funciones ofrecidas por el `SDK <http://bbs.espressif.com/viewtopic.php?f=51&t=1023>`__ de ESP8266 y no están presentes en la biblioteca `Arduino WiFi <https://www.arduino.cc/en/Reference/WiFi>`__. Si una función no encaja en una de las clases discutidas anteriormente, probablemente estará en la Clase Genérica. Entre ellas se encuentra el controlador para gestionar eventos WiFi como conexión, desconexión u obtención de una IP, cambios en el modo WiFi, funciones para gestionar el modo de suspensión del módulo, nombre de host para una resolución de dirección IP, etc.

Echa un vistazo a la sección separada con :doc:`ejemplos <generic-examples>` / :doc:`lista de funciones <generic-class>`.

Diagnóstico
-----------

Hay varias técnicas disponibles para diagnosticar y solucionar problemas al conectarse a WiFi y mantener la conexión activa.

Comprobar los códigos devueltos
~~~~~~~~~~~~~~~~~~

Casi todas las funciones descritas en los capítulos anteriores devuelven información de diagnóstico.

Tal diagnóstico se puede proporcionar como un simple ``booleano``, ``true`` o ``false``, para indicar el resultado de la operación. Puede verificar este resultado como se describe en los ejemplos, por ejemplo:

.. code:: cpp

    Serial.printf("Modo WiFi establecido a WIFI_STA %s\n", WiFi.mode(WIFI_STA) ? "" : "Falló!");

Algunas funciones proporcionan más que solo una información binaria. Un buen ejemplo es ``WiFi.status()``.

.. code:: cpp

    Serial.printf("Estado de la conexión: %d\n", WiFi.status());

Esta función devuelve los siguientes códigos para describir lo que está sucediendo con la conexión WiFi:

* 0 : ``WL_IDLE_STATUS`` cuando WiFi está en proceso de cambio de estado
* 1 : ``WL_NO_SSID_AVAIL`` en caso de que no se pueda alcanzar el SSID configurado
* 3 : ``WL_CONNECTED`` después de establecida una conexión exitosa
* 4 : ``WL_CONNECT_FAILED`` si la contraseña es incorrecta
* 6 : ``WL_DISCONNECTED`` si el módulo no está configurado en modo estación

Es una buena práctica mostrar y verificar la información devuelta por las funciones. El desarrollo de aplicaciones y la resolución de problemas serán así más fáciles.

Usar printDiag
~~~~~~~~~~~~~

Hay una función específica disponible para imprimir la información clave de diagnóstico del WiFi:

.. code:: cpp

    WiFi.printDiag(Serial);

Una salida de muestra de esta función se ve de la siguiente manera:

::

    Mode: STA+AP
    PHY mode: N
    Channel: 11
    AP id: 0
    Status: 5
    Auto connect: 1
    SSID (10): sensor-net
    Passphrase (12): 123!$#0&*esP
    BSSID set: 0

Utilice esta función para proporcionar una instantánea del estado de Wi-Fi en partes del código de la aplicación, que sospecha que puede estar fallando.

Activar el diagnóstico WiFi
~~~~~~~~~~~~~~~~~~~~~~~

Por defecto, la salida de diagnóstico de las librerías WiFi están desactivadas cuando se invoca ``Serial.begin``. Para habilitar nuevamente la salida de depuración, llame a ``Serial.setDebugOutput(true)``. Para redirigir la salida de depuración a ``Serial1``, llame a ``Serial1.setDebugOutput(true)``. Para obtener más detalles sobre el diagnóstico con puertos serie, consulte :doc:`the documentation <../reference>`.

A continuación se muestra un ejemplo de salida para el sketch de muestra discutido mas arriba en `Inicio rápido <#inicio rápido>`__ con ``Serial.setDebugOutput(true)``:

::

    Conectandoscandone
    state: 0 -> 2 (b0)
    state: 2 -> 3 (0)
    state: 3 -> 5 (10)
    add 0
    aid 1
    cnt 

    connected with sensor-net, channel 6
    dhcp client start...
    chg_B1:-40
    ...ip:192.168.1.10,mask:255.255.255.0,gw:192.168.1.9
    .
    Conectado, dirección IP: 192.168.1.10

El mismo sketch sin ``Serial.setDebugOutput(true)`` imprimirá lo siguiente:

::

    Conectando....
    Conectado, dirección IP: 192.168.1.10

Activar Debug en el IDE
~~~~~~~~~~~~~~~~~~~~~~~

El IDE Arduino provee métodos para `activar la depuración <../Troubleshooting/debugging.rst>`__ para librerías específicas.

¿Que hay dentro?
--------------

Si desea analizar en detalle qué hay dentro de la librería ESP8266WiFi, vaya directamente a la carpeta `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi/src>`__ del repositorio ESP8266/Arduino en GitHub.

Para facilitar el análisis, en lugar de buscar en el encabezado individual o en los archivos fuente, use una de las herramientas gratuitas para generar documentación automáticamente. El índice de clase en el capítulo de mas arriba `Descripción de clase <#descripción-de-la-clase>`__ ha sido preparado en muy poco tiempo usando el gran `Doxygen <http://www.stack.nl/~dimitri/doxygen/>`__, que es el herramienta estándar de facto para generar documentación a partir de fuentes anotadas de C++.

.. figure:: pictures/doxygen-esp8266wifi-documentation.png
   :alt: Ejemplo de documentación preparada con Doxygen

   Ejemplo de documentación preparada con Doxygen

La herramienta rastrea todos los archivos de encabezado y fuente, recopilando información de los bloques de comentarios formateados. Si el desarrollador de una clase particular anotó el código, lo verá como en los ejemplos a continuación.

.. figure:: pictures/doxygen-example-station-begin.png
   :alt: Ejemplo de documentación para el método begin STA por Doxygen

   Ejemplo de documentación para el método begin STA por Doxygen

.. figure:: pictures/doxygen-example-station-hostname.png
   :alt: Ejemplo de documentación para la propiedad hostname por Doxygen

   Ejemplo de documentación para la propiedad hostname por Doxygen

Si el código no está anotado, aún verá el prototipo de la función, incluidos los tipos de argumentos y puede usar los enlaces proporcionados para ir directamente al código fuente para verificarlo por su cuenta. Doxygen proporciona una navegación realmente excelente entre los miembros de la librería.

.. figure:: pictures/doxygen-example-udp-begin.png
   :alt: Ejemplo de documentación para el método begin UDP (no anotado en el código) por Doxygen

   Ejemplo de documentación para el método begin UDP (no anotado en el código) por Doxygen

Varias clases de `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ no están anotadas. Al preparar este documento, `Doxygen <http://www.stack.nl/~dimitri/doxygen/>`__ ha sido de gran ayuda para navegar rápidamente a través de casi 30 archivos que componen esta librería.
