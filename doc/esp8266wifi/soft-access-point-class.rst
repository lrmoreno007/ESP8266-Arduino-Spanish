Clase Soft Access Point
-----------------------

Los métodos y propiedades descritos más abajo son específicos de ESP8266. No están cubiertos en la documentación de librería WiFi de Arduino. Antes de que estén complétamente documentados, consulte la información a continuación.

La siguiente sección es específica para ESP8266, porque la documentación de la `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__ no cubre los puntos de acceso wireless. La descripción de la API está dividida en tres capítulos cortos. Cubren como: configurar soft-AP, gestionar la conexión y obtener información sobre la configuración del interfaz soft-AP.

Tabla de contenidos
-----------------

-  `Establecer la red <#establecer-la-red>`__

   -  `softAP <#softap>`__
   -  `softAPConfig <#softapconfig>`__

-  `Gestión de red <#gestión-de-red>`__

   -  `softAPdisconnect <#softapdisconnect>`__
   -  `softAPgetStationNum <#softapgetstationnum>`__

-  `Configuración de red <#configuración-de-red>`__

   -  `softAPIP <#softapip>`__
   -  `softAPmacAddress <#softapmacaddress>`__

Establecer la red
~~~~~~~~~~~~~~

Esta sección describe las funciones para establecer y configurar ESP8266 en el modo punto de acceso wireless (soft-AP).

softAP
^^^^^^

Establece un punto de acceso wireless para crear una red WiFi.

La versión mas simple de esta función requiere solo un parámetro y se utiliza para establecer una red WiFi abierta.

.. code:: cpp

    WiFi.softAP(ssid)

Para establecer una red protegida con contraseña, o para configurar parámetros de red adicionales, utilice la siguiente sobrecarga:

.. code:: cpp

    WiFi.softAP(ssid, password, channel, hidden, max_connection)

El primer parámetro de esta función es obligatorio, los otros cuatro son opcionales.

El significado de todos los parámetros es el siguiente:

* ``ssid`` - cadena de caracteres que contiene el SSID de la red (máximo 63 caracteres)

* ``password`` - cadena de caracteres opcional con una contraseña. Para la red WPA2-PSK, debe tener al menos 8 caracteres de longitud. Si no se especifica, el punto de acceso estará abierto para que cualquiera pueda conectarse.

* ``channel`` - parámetro opcional para establecer el canal de WiFi, del 1 aa 13. Canal predeterminado = 1.

* ``hidden`` - parámetro opcional, si se establece en ``true`` se ocultará el SSID

* ``max_connection`` - parámetro opcional para establecer el número máximo de estaciones conectadas simultaneamente, `de 0 a 8 <https://bbs.espressif.com/viewtopic.php?f=46&t=481&p=1832&hilit=max_connection#p1832>`__. Por defecto 4. Una vez que el número máximo se ha alcanzado, cualquier otra estación que quiera conectarse se verá forzada a esperar hasta que alguna estación conectada se desconecte.

La función devolverá ``true`` o ``false`` según el resultado de configurar el soft-AP.

Notas: 

* La red establecida por softAP tendrá una dirección IP predeterminada de 192.168.4.1. Esta dirección puede cambiarse usando ``softAPConfig`` (ver a continuación). 

* A pesar de que ESP8266 puede operar en modo soft-AP + station, en realidad solo tiene un canal de hardware. Por lo tanto, en el modo soft-AP + station, el canal soft-AP se predeterminará al número utilizado por la estación. Para obtener más información sobre cómo puede esto afectar al funcionamiento de las estaciones conectadas al soft-AP de ESP8266, consulte `esta entrada de preguntas frecuentes <http://bbs.espressif.com/viewtopic.php?f=10&t=324>`__ en el foro de Espressif.

softAPConfig
^^^^^^^^^^^^

Configura el interfaz de red del punto de acceso wireless.

.. code:: cpp

    softAPConfig (local_ip, gateway, subnet) 

Todos los parametros son del tipo ``IPAddress`` y se definen de la siguiente manera:

* ``local_ip`` - Dirección IP del punto de acceso

* ``gateway`` - Dirección IP del gateway o puerta de enlace

* ``subnet`` - Máscara de subred

La función devolverá ``true`` o ``false`` dependiendo del resultado de cambiar la configuración.

*Código de ejemplo:*

.. code:: cpp

    #include <ESP8266WiFi.h>

    IPAddress local_IP(192,168,4,22);
    IPAddress gateway(192,168,4,9);
    IPAddress subnet(255,255,255,0);

    void setup()
    {
      Serial.begin(115200);
      Serial.println();

      Serial.print("Estableciendo configuración Soft-AP... ");
      Serial.println(WiFi.softAPConfig(local_IP, gateway, subnet) ? "Listo" : "Falló!");

      Serial.print("Estableciendo modo Soft-AP... ");
      Serial.println(WiFi.softAP("ESPsoftAP_01") ? "Listo" : "Falló!");

      Serial.print("Dirección IP Soft-AP = ");
      Serial.println(WiFi.softAPIP());
    }

    void loop() {}

*Ejemplo de salida:*

::

    Estableciendo configuración Soft-AP... Listo
    Estableciendo modo Soft-AP... Listo
    Dirección IP Soft-AP =  192.168.4.22

Gestión de red
~~~~~~~~~~~~~~

Una vez que Soft-AP está establecido puedes comprobar el número de estaciones conectadas, o desactivarlo, utilizando las siguientes funciones.

softAPgetStationNum
^^^^^^^^^^^^^^^^^^^

Obtiene el número de estaciones que están conectadas al interfaz Soft-AP.

.. code:: cpp

    WiFi.softAPgetStationNum() 

*Código de ejemplo:*

.. code:: cpp

    Serial.printf("Estaciones conectadas a soft-AP = %d\n", WiFi.softAPgetStationNum());

*Ejemplo de salida:*

::

    Estaciones conectadas a soft-AP = 2

Nota: el número máximo de estaciones que pueden estar conectadas al Soft-AP ESP8266 es 4 por defecto. Esto puede cambiarse de 1 a 8 mediante el argumento ``max_connection`` del método softAP.

softAPdisconnect
^^^^^^^^^^^^^^^^

Desconecta estaciones de la red establecida por el soft-AP.

.. code:: cpp

    WiFi.softAPdisconnect(wifioff) 

La función establece un SSID y password del soft-AP a valores nulos (null). El parámetro ``wifioff`` es opcional. Si se establece a ``true`` se cambiará el soft-AP a modo apagado (off).

La función devuelve ``true`` si la operación se realizó satisfactoriamente o ``false`` en caso contrario.

Configuración de red
~~~~~~~~~~~~~~~~~~~~~

La siguientes funciones permiten obtener la dirección IP y MAC del Soft-AP de ESP8266.

softAPIP
^^^^^^^^

Devuelve la dirección IP del interfaz de red del punto de acceso.

.. code:: cpp

    WiFi.softAPIP() 

El retorno es un valor del tipo ``IPAddress``.

*Código de ejemplo:*

.. code:: cpp

    Serial.print("Dirección IP Soft-AP = ");
    Serial.println(WiFi.softAPIP());

*Ejemplo de salida:*

::

    Dirección IP Soft-AP = 192.168.4.1

softAPmacAddress
^^^^^^^^^^^^^^^^

Devuelve la dirección MAC del punto de acceso. Esta función tiene dos versiones, que se diferencian por el tipo de valor devuelto. El primero devuelve un puntero, el segundo un ``String``.

MAC como puntero
''''''''''''''

.. code:: cpp

    WiFi.softAPmacAddress(mac)

La función acepta un parámetro ``mac`` que es un puntero a la dirección de memoria (un ``uint8_t`` array de tamaño 6 elementos) para guardar la dirección MAC. El mismo puntero es devuelto por la función a si misma.

*Codigo de ejemplo:*

.. code:: cpp

    uint8_t macAddr[6];
    WiFi.softAPmacAddress(macAddr);
    Serial.printf("MAC address = %02x:%02x:%02x:%02x:%02x:%02x\n", macAddr[0], macAddr[1], macAddr[2], macAddr[3], macAddr[4], macAddr[5]);

*Ejemplo de salida:*

::

    MAC address = 5e:cf:7f:8b:10:13

MAC como una String
'''''''''''''''

Opcionalmente puedes utilizar esta función sin ningún parámetro que devuelva el valor tipo ``String``.

.. code:: cpp

    WiFi.softAPmacAddress()

*Example code:*

.. code:: cpp

    Serial.printf("MAC address = %s\n", WiFi.softAPmacAddress().c_str());

*Example output:*

::

    MAC address = 5E:CF:7F:8B:10:13

Consulte la sección separada con `ejemplos <soft-access-point-examples.rst>`__ dedicados específicamente a la clase Client.
