Clase Station
-------------

La cantidad de características proporcionadas por ESP8266 en el modo de estación es mucho más extensa que la que se cubre en la libreríaa original `Arduino WiFi <https://www.arduino.cc/en/Reference/WiFi>`__. Por lo tanto, en lugar de complementar la documentación original, hemos decidido escribir una nueva desde cero.

La descripción de la clase de estación se ha dividido en cuatro partes. Primero analiza los métodos para establecer la conexión a un punto de acceso. Segundo proporciona métodos para administrar la conexión, como p. ej. ``reconnect`` o ``isConnected``. En tercer lugar, se cubren las propiedades para obtener información sobre la conexión, como la dirección MAC o IP. Finalmente, la cuarta sección proporciona métodos alternativos para conectarse, como p. ej. Configuración protegida WiFi (WPS).

Tabla de contenidos
-----------------

-  `Comienza aquí <#comienza-aquí>`__

   -  `begin <#begin>`__
   -  `config <#config>`__

-  `Gestión de la conexión <#gestión-de-la-conexión>`__

   -  `reconnect <#reconnect>`__
   -  `disconnect <#disconnect>`__
   -  `isConnected <#isconnected>`__
   -  `setAutoConnect <#setautoconnect>`__
   -  `getAutoConnect <#getautoconnect>`__
   -  `setAutoReconnect <#setautoreconnect>`__
   -  `waitForConnectResult <#waitforconnectresult>`__

-  `Configuración <#configuración>`__

   -  `macAddress <#macaddress>`__
   -  `localIP <#localip>`__
   -  `subnetMask <#subnetmask>`__
   -  `gatewayIP <#gatewayip>`__
   -  `dnsIP <#dnsip>`__
   -  `hostname <#hostname>`__
   -  `status <#status>`__
   -  `SSID <#ssid>`__
   -  `psk <#psk>`__
   -  `BSSID <#bssid>`__
   -  `RSSI <#rssi>`__

-  `Conexiones diferentes <#conexiones-diferentes>`__

   -  `WPS <#wps>`__
   -  `Smart Config <#smart-config>`__

Los puntos siguientes proporcionan descripciones y retazos de código sobre como usar métodos particulares.

Para obtener más muestras de código, consulte la sección separada con: doc:`ejemplos <station-examples>` dedicado específicamente a la Clase Station.

Comienza aquí
~~~~~~~~~~

El cambio del módulo al modo estación se realiza con la función ``begin``. Los parámetros típicos de ``begin`` incluyen SSID y contraseña, por lo que el módulo se puede conectar a un punto de acceso específico.

.. code:: cpp

    WiFi.begin(ssid, password)

Por defecto, ESP intentará volver a conectarse a la red WiFi cada vez que se desconecte. No hay necesidad de manejar esto con código separado. Una buena forma de simular la desconexión sería resetear el punto de acceso. ESP reportará la desconexión y luego tratará de reconectarse automáticamente.

begin
^^^^^

Hay varias versiones (sobrecargas) de la función ``begin``. Una presentada justo ahora: ``WiFi.begin(ssid, password)``. Sobrecargas que proveen flexibilidad en el número o tipos de parámetros aceptados.

La sobrecarga mas simple de ``begin`` es la siguiente:

.. code:: cpp

    WiFi.begin()

Al llamarla se le indicará al módulo que cambie al modo estación y se conecte al último punto de acceso utilizado basándose en la configuración guardada en la memoria flash.

A continuación se muestra la sintaxis de otra sobrecarga de ``begin`` con todos los parámetros posibles:

.. code:: cpp

    WiFi.begin(ssid, password, channel, bssid, connect)

El significado de los parámetros es el siguiente:

* ``ssid`` - Una String que contiene el SSID del punto de acceso que queremos conectarno, Puede tener hasta 32 caracteres. 

* ``password`` - Contraseña del punto de acceso, un String que debe tener como mínimo una longitud de 8 caracteres y menos de 64 caracteres.

* ``channel`` - Canal del AP, si queremos operar en un canal específico, de lo contrario, este parámetro puede omitirse. 

* ``bssid`` - Dirección MAC del AP, este parámetro es también opcional.

* ``connect`` - Un parámetro ``boolean`` que si se establece a ``false``, instruirá al módulo para que solo guarde los parámetros sin establecer conexión con el punto de acceso.

config
^^^^^^

Desactiva el cliente `DHCP <https://es.wikipedia.org/wiki/Protocolo_de_configuraci%C3%B3n_din%C3%A1mica_de_host>`__ y establece la configuración IP del interfaz de estación con valores definidos por el usuario. El interfaz tendrá una configuración IP estática en vez de los valores servidos por el DHCP.

.. code:: cpp

    WiFi.config(local_ip, gateway, subnet, dns1, dns2) 

Función que devuelve ``true`` si la configuración se aplica satisfactoriamente. Si la configuración no puede aplicarse, porque p. ej. el módulo no está en modo estación o estacion + punto de acceso, entonces devolverá ``false``.

La siguiente configuración IP debe proveerse:

*  ``local_ip`` - Introduce aquí la dirección IP que quieras asignar al interfaz estación del ESP.

*  ``gateway`` - Debe contener la dirección IP de la puerta de enlace (getaway - de un router) para acceder a redes externas.

*  ``subnet`` - Esta es la máscara de subred que define el rango de direcciones IP de la red local.

*  ``dns1``, ``dns2`` - Parámetro opcional que define la dirección IP del servidor de dominio (DNS) que mantiene un directorio de nombres de dominio (como p.ej. *www.google.es*) y nos las traduce a direcciones IP.

*Código de ejemplo:*

.. code:: cpp

    #include <ESP8266WiFi.h>

    const char* ssid = "********";
    const char* password = "********";

    IPAddress staticIP(192,168,1,22);
    IPAddress gateway(192,168,1,9);
    IPAddress subnet(255,255,255,0);

    void setup(void)
    {
      Serial.begin(115200);
      Serial.println();

      Serial.printf("Conectando a %s\n", ssid);
      WiFi.begin(ssid, password);
      WiFi.config(staticIP, gateway, subnet);
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

*Ejemplo de salida:*

::

    Conectando a sensor-net
    .
    Conectado, dirección IP: 192.168.1.22

Tenga en cuenta que la estación con configuración de IP estática por lo general se conecta a la red más rápido. En el ejemplo anterior, tomó aproximadamente 500 ms (se muestra un punto `.`). Esto se debe a que la obtención de la configuración IP por parte del cliente DHCP lleva tiempo y en este caso este paso se omite. Si pasa los tres parámetros como 0.0.0.0 (local_ip, gateway y subred), volverá a habilitar DHCP. Necesita volver a conectar el dispositivo para obtener nuevas direcciones IP.

Gestión de la conexión
~~~~~~~~~~~~~~~~~

reconnect
^^^^^^^^^

Reconecta la estación. Esto se hace por desconexión del punto de acceso y entonces volviendo a conectar al mismo punto de acceso.

.. code:: cpp

    WiFi.reconnect() 

Notes: 

1. La estación debe estar ya conectada a un punto de acceso. Si no es el caso, la función devolverá ``false`` sin realizar ninguna acción.

2. Si se devuelve ``true`` significa que la secuencia de conexión se ha iniciado con éxito. El usuario aún debe verificar el estado de la conexión, esperando hasta que se informe ``WL_CONNECTED``:

.. code:: cpp

    WiFi.reconnect();
    while (WiFi.status() != WL_CONNECTED)
    {
      delay(500);
      Serial.print(".");
    }

disconnect
^^^^^^^^^^

Establece la SSID y contraseña actualmente configurada al valor ``null`` y desconecta la estación del punto de acceso.

.. code:: cpp

    WiFi.disconnect(wifioff) 

El parámetro ``wifioff`` es de tipo ``boolean`` opcional. Si se establece a ``true``, el modo estación será apagado.

isConnected
^^^^^^^^^^^

Devuelve ``true`` si la estación está conecta a un punto de acceso, en caso contrario devuelve ``false``.

.. code:: cpp

    WiFi.isConnected() 

setAutoConnect
^^^^^^^^^^^^^^

Configura el módulo para conectarse automáticamente tras encenderse al último punto de acceso utilizado.

.. code:: cpp

    WiFi.setAutoConnect(autoConnect) 

El parámetro ``autoConnect`` es opcional. Si se establece a ``false`` la funcionalidad de autoconexión será desactivada. Si se omite o se establece a ``true``, la autoconexión se activará.

getAutoConnect
^^^^^^^^^^^^^^

Es una función "compañera" a ``setAutoConnect()``. Si devuelve ``true`` el módulo está configurado para conectar al último punto de acceso tras encenderse.

.. code:: cpp

    WiFi.getAutoConnect()

Si la funcionalidad de autoconexión está desactivada, la función devuelve ``false``.

setAutoReconnect
^^^^^^^^^^^^^^^^

Establece si el módulo intentará volver a conectarse a un punto de acceso en caso de que esté desconectado.

.. code:: cpp

    WiFi.setAutoReconnect(autoReconnect)  

Si el parámetro ``autoReconnect`` está establecido en ``true``, el módulo intentará restablecer la conexión perdida al punto de acceso. Si se establece en ``false``, el módulo permanecerá desconectado.

Nota: ejecutar ``setAutoReconnect(true)`` cuando el módulo ya está desconectado no lo hará volver a conectarse al punto de acceso. En cambio, debería utilizarse ``reconnect()``.

waitForConnectResult
^^^^^^^^^^^^^^^^^^^^

Espera hasta que el módulo se conecte al punto de acceso. Esta función está destinada para el módulo configurado modo estación o estación + punto de acceso SoftAP.

.. code:: cpp

    WiFi.waitForConnectResult()  

La función devuelve uno de los siguientes estados de conexión:

* ``WL_CONNECTED`` - Después de establecida una conexión exitosa.

* ``WL_NO_SSID_AVAIL`` - En caso de que no se pueda alcanzar el SSID configurado.

* ``WL_CONNECT_FAILED`` - Si la contraseña es incorrecta.

* ``WL_IDLE_STATUS`` - Cuando WiFi está en proceso de cambio entre estados.

* ``WL_DISCONNECTED`` - Si el módulo no está configurado en modo de estación.

Configuración
~~~~~~~~~~~~~

macAddress
^^^^^^^^^^

Obtiene la dirección MAC de la interfaz de estación ESP.

.. code:: cpp

    WiFi.macAddress(mac) 

Se debe proporcionar a la función la variable ``mac`` que es un puntero a la ubicación de la memoria (una matriz ``uint8_t`` de tamaño de 6 elementos) para guardar la dirección MAC. El mismo valor de puntero es devuelto por la función.

*Código de ejemplo:*

.. code:: cpp

    if (WiFi.status() == WL_CONNECTED)
    {
      uint8_t macAddr[6];
      WiFi.macAddress(macAddr);
      Serial.printf("Conectado, dirección MAC: %02x:%02x:%02x:%02x:%02x:%02x\n", macAddr[0], macAddr[1], macAddr[2], macAddr[3], macAddr[4], macAddr[5]);
    }

*Ejemplo de salida:*

::

    Conectado, dirección MAC: 5C:CF:7F:08:11:17

Si no se siente cómodo con los punteros, existe una versión opcional de esta función disponible. En lugar del puntero, devuelve un ``String`` que contiene la misma dirección MAC.

.. code:: cpp

    WiFi.macAddress() 

*Código de ejemplo:*

.. code:: cpp

    if (WiFi.status() == WL_CONNECTED)
    {
      Serial.printf("Conectado, dirección MAC: %s\n", WiFi.macAddress().c_str());
    }

localIP
^^^^^^^

Función utilizada para obtener la dirección IP de la interfaz de estación ESP.

.. code:: cpp

    WiFi.localIP() 

El tipo de valor devuelto es `IPAddress <https://github.com/esp8266/Arduino/blob/master/cores/esp8266/IPAddress.h>`__. Hay un par de métodos disponibles para mostrar este tipo de datos. Se presentan en ejemplos a continuación que cubren la descripción de ``subnetMask``, ``gatewayIP`` y ``dnsIP`` que también devuelven valores IPAdress.

*Código de ejemplo:*

.. code:: cpp

    if (WiFi.status() == WL_CONNECTED)
    {
      Serial.print("Conectado, dirección IP: ");
      Serial.println(WiFi.localIP());
    }

*Ejemplo de salida:*

::

    Conectado, dirección IP: 192.168.1.10

subnetMask
^^^^^^^^^^

Obtiene la máscara de subred de la interfaz de estación.

.. code:: cpp

    WiFi.subnetMask()

El módulo debe estar conectado al punto de acceso para obtener la máscara de subred.

*Código de ejemplo:*

.. code:: cpp

    Serial.print("Máscara de subred: ");
    Serial.println(WiFi.subnetMask());

*Ejemplo de salida:*

::

    Máscara de subred: 255.255.255.0

gatewayIP
^^^^^^^^^

Obtiene la dirección IP de la puerta de acceso o gateway.

.. code:: cpp

    WiFi.gatewayIP()

*Código de ejemplo:*

.. code:: cpp

    Serial.printf("Getaway IP: %s\n", WiFi.gatewayIP().toString().c_str());

*Eejemplo de salida:*

::

    Getaway IP: 192.168.1.9

dnsIP
^^^^^

Obtiene la dirección IP del Servidor de Nombres de Dominio (DNS).

.. code:: cpp

    WiFi.dnsIP(dns_no)

Con el parámetro de entrada ``dns_no`` podemos especificar qué IP del servidor de nombres de dominio necesitamos. Este parámetro está basado en cero y los valores permitidos son ninguno, 0 o 1. Si no se proporciona ningún parámetro, se devuelve la IP del DNS n.° 1.

*Código de ejemplo:*

.. code:: cpp

    Serial.print("DNS: #1, #2 IP: ");
    WiFi.dnsIP().printTo(Serial);
    Serial.print(", ");
    WiFi.dnsIP(1).printTo(Serial);
    Serial.println();

*Ejemplo de salida:*

::

    DNS: #1, #2 IP: 62.179.1.60, 62.179.1.61

hostname
^^^^^^^^

Obtiene el nombre del servidor DHCP asignado a la estación ESP.

.. code:: cpp

    WiFi.hostname()

La función devuelve un valor del tipo ``String``. El nombre de host predeterminado está en formato ``ESP_24xMAC`` donde 24xMAC son los últimos 24 bits de la dirección MAC del módulo.

El nombre del servidor DHCP puede cambiarse usando la siguiente función:

.. code:: cpp

    WiFi.hostname(aHostname) 

El parámetro de entrada ``aHostname`` puede ser del tipo ``char*``, ``const char*`` o ``String``. La longitud máxima del nombre de host asignado es de 32 caracteres. La función devuelve ``true`` o ``false`` según el resultado. Por ejemplo, si se excede el límite de 32 caracteres, la función devolverá ``falso`` sin asignar el nuevo nombre de host.

*Código de ejemplo:*

.. code:: cpp

    Serial.printf("Hostname por defecto: %s\n", WiFi.hostname().c_str());
    WiFi.hostname("Station_Tester_02");
    Serial.printf("Nuevo hostname: %s\n", WiFi.hostname().c_str());

*Ejemplo de salida:*

::

    Hostname por defecto: ESP_081117
    Nuevo hostname: Station_Tester_02

status
^^^^^^

Devuelve el estado de la conexión WiFi.

.. code:: cpp

    WiFi.status()

La función devuelve uno de los siguientes estados de conexión:

* ``WL_CONNECTED`` - Después de que se establece una conexión exitosa.

* ``WL_NO_SSID_AVAIL`` - En caso de que no se pueda alcanzar el SSID configurado.

* ``WL_CONNECT_FAILED`` - Si la contraseña es incorrecta.

* ``WL_IDLE_STATUS`` - Cuando WiFi está cambiando de estado.

* ``WL_DISCONNECTED`` - Si el módulo no está configurado en modo de estación.

El valor devuelto es del tipo ``wl_status_t`` definido en `wl\_definitions.h <https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/src/include/wl_definitions.h>`__

*Código de ejemplo:*

.. code:: cpp

    #include <ESP8266WiFi.h>

    void setup(void)
    {
      Serial.begin(115200);
      Serial.printf("Estado de la conexión: %d\n", WiFi.status());
      Serial.printf("Conectando a %s\n", ssid);
      WiFi.begin(ssid, password);
      Serial.printf("Estado de la conexión: %d\n", WiFi.status());
      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }
      Serial.printf("\nEstado de la conexión: %d\n", WiFi.status());
      Serial.print("Conectado, dirección IP: ");
      Serial.println(WiFi.localIP());
    }

    void loop() {}

*Ejemplo de salida:*

::

    Estado de la conexión: 6
    Conectando a sensor-net
    Estado de la conexión: 6
    ......
    Estado de la conexión: 3
    Conectado, dirección IP: 192.168.1.10

Los estatus de conexión 6 y 3 como puede verse en `wl\_definitions.h <https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/src/include/wl_definitions.h>`__ son:

::

    3 - WL_CONNECTED
    6 - WL_DISCONNECTED

Según este ejemplo, cuando se ejecuta el código anterior, el módulo está desconectado inicialmente de la red y devuelve el estado de conexión ``6 - WL_DISCONNECTED``. También está desconectado inmediatamente después de ejecutar ``WiFi.begin(ssid, password)``. Luego, después de unos 3 segundos (basándose en el número de puntos que se muestran cada 500 ms), finalmente se activa el estado de conexión ``3 - WL_CONNECTED``.

SSID
^^^^

Devuelve el nombre de la red WiFi, formalmente llamada `Service Set Identifier (SSID) <https://es.wikipedia.org/wiki/SSID>`__.

.. code:: cpp

    WiFi.SSID()

El valor devuelto es del tipo ``String``.

*Código de ejemplo:*

.. code:: cpp

    Serial.printf("SSID: %s\n", WiFi.SSID().c_str());

*Ejemplo de salida:*

::

    SSID: sensor-net

psk
^^^

Devuelve la clave precompartida actual (contraseña) asociada a la red WiFi.

.. code:: cpp

    WiFi.psk()

El valor devuelto es del tipo ``String``.

BSSID
^^^^^

Devuelve la dirección MAC del punto de acceso al que está conectado el módulo ESP. Esta dirección se denomina formalmente `Basic Service Set Identifier (BSSID) <https://es.wikipedia.org/wiki/BSSID>`__.

.. code:: cpp

    WiFi.BSSID()

La función ``BSSID()`` devuelve un puntero a la ubicación de la memoria (una matriz ``uint8_t`` con tamaño de 6 elementos) donde se guarda el BSSID.

A continuación se muestra una función similar, pero que devuelve BSSID como del tipo ``String``.

.. code:: cpp

    WiFi.BSSIDstr()  

*Código de ejemplo:*

.. code:: cpp

    Serial.printf("BSSID: %s\n", WiFi.BSSIDstr().c_str());

*Ejemplo de salida:*

::

    BSSID: 00:1A:70:DE:C1:68

RSSI
^^^^

Devuelve la potencia de la señal WiFi, que formalmente se llama `Received Signal Strength Indicator (RSSI) <https://es.wikipedia.org/wiki/Indicador_de_fuerza_de_la_se%C3%B1al_recibida>`__.

.. code:: cpp

    WiFi.RSSI() 

El valor de intensidad de la señal se proporciona en dBm (decibelios). El tipo del valor devuelto es ``int32_t``.

*Código de ejemplo:*

.. code:: cpp

    Serial.printf("RSSI: %d dBm\n", WiFi.RSSI());

*Ejemplo de salida:*

::

    RSSI: -68 dBm

Conexiones diferentes
~~~~~~~~~~~~~~~~~

El `SDK de ESP8266 <http://bbs.espressif.com/viewtopic.php?f=51&t=1023>`__ proporciona métodos para conectar la estación ESP a un punto de acceso. Aparte el core `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__ implementa `WPS <#wps>`__ y `Smart Config <#smart-config>`__ como se describe con más detalle a continuación.


WPS
^^^

La siguiente función ``beginWPSConfig`` permite conectarse a una red usando `Wi-Fi Protected Setup (WPS) <https://es.wikipedia.org/wiki/Wi-Fi_Protected_Setup>`__. Actualmente solo se permite el método `PBC (Push Button Configuration) <https://es.wikipedia.org/wiki/Wi-Fi_Protected_Setup>`__ (``modo WPS_TYPE_PBC``) compatible (SDK 1.5.4).

.. code:: cpp

    WiFi.beginWPSConfig()

Dependiendo del resultado de la conexión, la función devuelve ``true`` o ``false`` (tipo ``boolean``).

*Código de ejemplo:*

.. code:: cpp

    #include <ESP8266WiFi.h>

    void setup(void)
    {
      Serial.begin(115200);
      Serial.println();

      Serial.printf("Modo WiFi establecido a WIFI_STA %s\n", WiFi.mode(WIFI_STA) ? "" : "Failed!");
      Serial.print("Comienzo WPS (presione el botón WPS en su router) ... ");
      Serial.println(WiFi.beginWPSConfig() ? "OK" : "Fallo");

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

*Ejemplo de salida:*

::

    Modo WiFi establecido a WIFI_STA 
    Comienzo WPS (presione el botón WPS en su router) ...  OK
    .........
    Conectado, dirección IP: 192.168.1.102

Smart Config
^^^^^^^^^^^^

La conexión Smart Config de un módulo ESP y un punto de acceso se realiza olfateando (sniffing) los paquetes especiales que contienen SSID y la contraseña del AP deseado. Para hacerlo, el dispositivo móvil o la computadora deben tener la funcionalidad de transmisión de SSID y contraseña codificados.

Las siguientes tres funciones se proporcionan para implementar Smart Config.

Inicie el modo Smart Config olfateando los paquetes especiales que contienen el SSID y la contraseña del punto de acceso deseado. Dependiendo del resultado, se devuelve ``true`` o ``false``.

.. code:: cpp

    beginSmartConfig() 

Consulta el estado de Smart Config, para decidir cuándo detener la configuración. La función devuelve ``true`` o ``false`` de tipo ``boolean``.

.. code:: cpp

    smartConfigDone()

Detiene Smart Config y libera el buffer tomado por ``beginSmartConfig()``. Dependiendo del resultado de la función, devuelve ``true`` o `` false`` de tipo `` boolean``.

.. code:: cpp

    stopSmartConfig() 

Para obtener más información acerca de Smart Config, consulte la guía de usuario de la `API ESP8266 <http://bbs.espressif.com/viewtopic.php?f=51&t=1023>`__.
