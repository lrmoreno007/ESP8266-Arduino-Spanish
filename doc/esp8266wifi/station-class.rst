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

   -  `macAddress <#macAddress>`__
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

Meaning of parameters is as follows: 

* ``ssid`` - Una String que contiene el SSID del punto de acceso que queremos conectarno, Puede tener hasta 32 caracteres. 

* ``password`` - Contraseña del punto de acceso, un String que debe tener como mínimo una longitud de 8 caracteres y menos de 64 caracteres.

* ``channel`` Canal del AP, si queremos operar en un canal específico, de lo contrario, este parámetro puede omitirse. 

* ``bssid`` - Dirección MAC del AP, este parámetro es también opcional.

* ``connect`` - Un parámetro ``boolean`` que si se establece a ``false``, instruirá al módulo para que solo guarde los parámetros sin establecer conexión con el punto de acceso.

config
^^^^^^

Desactiva el cliente `DHCP <https://es.wikipedia.org/wiki/Protocolo_de_configuraci%C3%B3n_din%C3%A1mica_de_host>`__ y establece la configuración IP del interfaz de estación con valores definidos por el usuario. El interfaz tendrá una configuración IP estática en vez de los valores servidos por el DHCP.

.. code:: cpp

    WiFi.config(local_ip, gateway, subnet, dns1, dns2) 

Function will return ``true`` if configuration change is applied successfully. If configuration can not be applied, because e.g. module is not in station or station + soft access point mode, then ``false`` will be returned.

The following IP configuration may be provided:

-  ``local_ip`` - enter here IP address you would like to assign the ESP
   station's interface
-  ``gateway`` - should contain IP address of gateway (a router) to
   access external networks
-  ``subnet`` - this is a mask that defines the range of IP addresses of
   the local network
-  ``dns1``, ``dns2`` - optional parameters that define IP addresses of
   Domain Name Servers (DNS) that maintain a directory of domain names
   (like e.g. *www.google.co.uk*) and translate them for us to IP
   addresses

*Example code:*

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

      Serial.printf("Connecting to %s\n", ssid);
      WiFi.begin(ssid, password);
      WiFi.config(staticIP, gateway, subnet);
      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }
      Serial.println();
      Serial.print("Connected, IP address: ");
      Serial.println(WiFi.localIP());
    }

    void loop() {}

*Example output:*

::

    Connecting to sensor-net
    .
    Connected, IP address: 192.168.1.22

Please note that station with static IP configuration usually connects to the network faster. In the above example it took about 500ms (one dot `.` displayed). This is because obtaining of IP configuration by DHCP client takes time and in this case this step is skipped. If you pass all three parameter as 0.0.0.0 (local_ip, gateway and subnet), it will re enable DHCP. You need to re-connect the device to get new IPs.


Gestión de la conexión
~~~~~~~~~~~~~~~~~

reconnect
^^^^^^^^^

Reconnect the station. This is done by disconnecting from the access point an then initiating connection back to the same AP.

.. code:: cpp

    WiFi.reconnect() 

Notes: 1. Station should be already connected to an access point. If this is not the case, then function will return ``false`` not performing any action. 2. If ``true`` is returned it means that connection sequence has been successfully started. User should still check for connection status, waiting until ``WL_CONNECTED`` is reported:

.. code:: cpp

    WiFi.reconnect();
    while (WiFi.status() != WL_CONNECTED)
    {
      delay(500);
      Serial.print(".");
    }

disconnect
^^^^^^^^^^

Sets currently configured SSID and password to ``null`` values and disconnects the station from an access point.

.. code:: cpp

    WiFi.disconnect(wifioff) 

The ``wifioff`` is an optional ``boolean`` parameter. If set to ``true``, then the station mode will be turned off.

isConnected
^^^^^^^^^^^

Returns ``true`` if Station is connected to an access point or ``false`` if not.

.. code:: cpp

    WiFi.isConnected() 

setAutoConnect
^^^^^^^^^^^^^^

Configure module to automatically connect on power on to the last used access point.

.. code:: cpp

    WiFi.setAutoConnect(autoConnect) 

The ``autoConnect`` is an optional parameter. If set to ``false`` then auto connection functionality up will be disabled. If omitted or set to ``true``, then auto connection will be enabled.

getAutoConnect
^^^^^^^^^^^^^^

This is "companion" function to ``setAutoConnect()``. It returns ``true`` if module is configured to automatically connect to last used access point on power on.

.. code:: cpp

    WiFi.getAutoConnect()

If auto connection functionality is disabled, then function returns ``false``.

setAutoReconnect
^^^^^^^^^^^^^^^^

Set whether module will attempt to reconnect to an access point in case it is disconnected.

.. code:: cpp

    WiFi.setAutoReconnect(autoReconnect)  

If parameter ``autoReconnect`` is set to ``true``, then module will try to reestablish lost connection to the AP. If set to ``false`` then module will stay disconnected.

Note: running ``setAutoReconnect(true)`` when module is already disconnected will not make it reconnect to the access point. Instead ``reconnect()`` should be used.

waitForConnectResult
^^^^^^^^^^^^^^^^^^^^

Wait until module connects to the access point. This function is intended for module configured in station or station + soft access point mode.

.. code:: cpp

    WiFi.waitForConnectResult()  

Function returns one of the following connection statuses: \* ``WL_CONNECTED`` after successful connection is established \* ``WL_NO_SSID_AVAIL``\ in case configured SSID cannot be reached \* ``WL_CONNECT_FAILED`` if password is incorrect \* ``WL_IDLE_STATUS`` when Wi-Fi is in process of changing between statuses \* ``WL_DISCONNECTED`` if module is not configured in station mode

Configuración
~~~~~~~~~~~~~

macAddress
^^^^^^^^^^

Get the MAC address of the ESP station's interface.

.. code:: cpp

    WiFi.macAddress(mac) 

Function should be provided with ``mac`` that is a pointer to memory location (an ``uint8_t`` array the size of 6 elements) to save the mac address. The same pointer value is returned by the function itself.

*Example code:*

.. code:: cpp

    if (WiFi.status() == WL_CONNECTED)
    {
      uint8_t macAddr[6];
      WiFi.macAddress(macAddr);
      Serial.printf("Connected, mac address: %02x:%02x:%02x:%02x:%02x:%02x\n", macAddr[0], macAddr[1], macAddr[2], macAddr[3], macAddr[4], macAddr[5]);
    }

*Example output:*

::

    Mac address: 5C:CF:7F:08:11:17

If you do not feel comfortable with pointers, then there is optional version of this function available. Instead of the pointer, it returns a formatted ``String`` that contains the same mac address.

.. code:: cpp

    WiFi.macAddress() 

*Example code:*

.. code:: cpp

    if (WiFi.status() == WL_CONNECTED)
    {
      Serial.printf("Connected, mac address: %s\n", WiFi.macAddress().c_str());
    }

localIP
^^^^^^^

Function used to obtain IP address of ESP station's interface.

.. code:: cpp

    WiFi.localIP() 

The type of returned value is `IPAddress <https://github.com/esp8266/Arduino/blob/master/cores/esp8266/IPAddress.h>`__. There is a couple of methods available to display this type of data. They are presented in examples below that cover description of ``subnetMask``, ``gatewayIP`` and ``dnsIP`` that return the IPAdress as well.

*Example code:*

.. code:: cpp

    if (WiFi.status() == WL_CONNECTED)
    {
      Serial.print("Connected, IP address: ");
      Serial.println(WiFi.localIP());
    }

*Example output:*

::

    Connected, IP address: 192.168.1.10

subnetMask
^^^^^^^^^^

Get the subnet mask of the station's interface.

.. code:: cpp

    WiFi.subnetMask()

Module should be connected to the access point to obtain the subnet mask.

*Example code:*

.. code:: cpp

    Serial.print("Subnet mask: ");
    Serial.println(WiFi.subnetMask());

*Example output:*

::

    Subnet mask: 255.255.255.0

gatewayIP
^^^^^^^^^

Get the IP address of the gateway.

.. code:: cpp

    WiFi.gatewayIP()

*Example code:*

.. code:: cpp

    Serial.printf("Gataway IP: %s\n", WiFi.gatewayIP().toString().c_str());

*Example output:*

::

    Gataway IP: 192.168.1.9

dnsIP
^^^^^

Get the IP addresses of Domain Name Servers (DNS).

.. code:: cpp

    WiFi.dnsIP(dns_no)

With the input parameter ``dns_no`` we can specify which Domain Name Server's IP we need. This parameter is zero based and allowed values are none, 0 or 1. If no parameter is provided, then IP of DNS #1 is returned.

*Example code:*

.. code:: cpp

    Serial.print("DNS #1, #2 IP: ");
    WiFi.dnsIP().printTo(Serial);
    Serial.print(", ");
    WiFi.dnsIP(1).printTo(Serial);
    Serial.println();

*Example output:*

::

    DNS #1, #2 IP: 62.179.1.60, 62.179.1.61

hostname
^^^^^^^^

Get the DHCP hostname assigned to ESP station.

.. code:: cpp

    WiFi.hostname()

Function returns ``String`` type. Default hostname is in format ``ESP_24xMAC``\ where 24xMAC are the last 24 bits of module's MAC address.

The hostname may be changed using the following function:

.. code:: cpp

    WiFi.hostname(aHostname) 

Input parameter ``aHostname`` may be a type of ``char*``, ``const char*`` or ``String``. Maximum length of assigned hostname is 32 characters. Function returns either ``true`` or ``false`` depending on result. For instance, if the limit of 32 characters is exceeded, function will return ``false`` without assigning the new hostname.

*Example code:*

.. code:: cpp

    Serial.printf("Default hostname: %s\n", WiFi.hostname().c_str());
    WiFi.hostname("Station_Tester_02");
    Serial.printf("New hostname: %s\n", WiFi.hostname().c_str());

*Example output:*

::

    Default hostname: ESP_081117
    New hostname: Station_Tester_02

status
^^^^^^

Return the status of Wi-Fi connection.

.. code:: cpp

    WiFi.status()

Function returns one of the following connection statuses: \* ``WL_CONNECTED`` after successful connection is established \* ``WL_NO_SSID_AVAIL``\ in case configured SSID cannot be reached \* ``WL_CONNECT_FAILED`` if password is incorrect \* ``WL_IDLE_STATUS`` when Wi-Fi is in process of changing between statuses \* ``WL_DISCONNECTED`` if module is not configured in station mode

Returned value is type of ``wl_status_t`` defined in `wl\_definitions.h <https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/src/include/wl_definitions.h>`__

*Example code:*

.. code:: cpp

    #include <ESP8266WiFi.h>

    void setup(void)
    {
      Serial.begin(115200);
      Serial.printf("Connection status: %d\n", WiFi.status());
      Serial.printf("Connecting to %s\n", ssid);
      WiFi.begin(ssid, password);
      Serial.printf("Connection status: %d\n", WiFi.status());
      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }
      Serial.printf("\nConnection status: %d\n", WiFi.status());
      Serial.print("Connected, IP address: ");
      Serial.println(WiFi.localIP());
    }

    void loop() {}

*Example output:*

::

    Connection status: 6
    Connecting to sensor-net
    Connection status: 6
    ......
    Connection status: 3
    Connected, IP address: 192.168.1.10

Particular connection statuses 6 and 3 may be looked up in `wl\_definitions.h <https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/src/include/wl_definitions.h>`__ as follows:

::

    3 - WL_CONNECTED
    6 - WL_DISCONNECTED

Basing on this example, when running above code, module is initially disconnected from the network and returns connection status ``6 - WL_DISCONNECTED``. It is also disconnected immediately after running ``WiFi.begin(ssid, password)``. Then after about 3 seconds (basing on number of dots displayed every 500ms), it finally gets connected returning status ``3 - WL_CONNECTED``.

SSID
^^^^

Return the name of Wi-Fi network, formally called `Service Set Identification (SSID) <http://www.juniper.net/techpubs/en_US/network-director1.1/topics/concept/wireless-ssid-bssid-essid.html#jd0e34>`__.

.. code:: cpp

    WiFi.SSID()

Returned value is of the ``String`` type.

*Example code:*

.. code:: cpp

    Serial.printf("SSID: %s\n", WiFi.SSID().c_str());

*Example output:*

::

    SSID: sensor-net

psk
^^^

Return current pre shared key (password) associated with the Wi-Fi network.

.. code:: cpp

    WiFi.psk()

Function returns value of the ``String`` type.

BSSID
^^^^^

Return the mac address the access point where ESP module is connected to. This address is formally called `Basic Service Set Identification (BSSID) <http://www.juniper.net/techpubs/en_US/network-director1.1/topics/concept/wireless-ssid-bssid-essid.html#jd0e47>`__.

.. code:: cpp

    WiFi.BSSID()

The ``BSSID()`` function returns a pointer to the memory location (an ``uint8_t`` array with the size of 6 elements) where the BSSID is saved.

Below is similar function, but returning BSSID but as a ``String`` type.

.. code:: cpp

    WiFi.BSSIDstr()  

*Example code:*

.. code:: cpp

    Serial.printf("BSSID: %s\n", WiFi.BSSIDstr().c_str());

*Example output:*

::

    BSSID: 00:1A:70:DE:C1:68

RSSI
^^^^

Return the signal strength of Wi-Fi network, that is formally called `Received Signal Strength Indication (RSSI) <https://en.wikipedia.org/wiki/Received_signal_strength_indication>`__.

.. code:: cpp

    WiFi.RSSI() 

Signal strength value is provided in dBm. The type of returned value is ``int32_t``.

*Example code:*

.. code:: cpp

    Serial.printf("RSSI: %d dBm\n", WiFi.RSSI());

*Example output:*

::

    RSSI: -68 dBm

Connect Different
~~~~~~~~~~~~~~~~~

`ESP8266 SDK <http://bbs.espressif.com/viewtopic.php?f=51&t=1023>`__ provides alternate methods to connect ESP station to an access point. Out of them `esp8266 / Arduino <https://github.com/esp8266/Arduino>`__ core implements `WPS <#wps>`__ and `Smart Config <#smart-config>`__ as described in more details below.

WPS
^^^

The following ``beginWPSConfig`` function allows connecting to a network using `Wi-Fi Protected Setup (WPS) <https://en.wikipedia.org/wiki/Wi-Fi_Protected_Setup>`__. Currently only `push-button configuration <http://www.wi-fi.org/knowledge-center/faq/how-does-wi-fi-protected-setup-work>`__ (``WPS_TYPE_PBC`` mode) is supported (SDK 1.5.4).

.. code:: cpp

    WiFi.beginWPSConfig()

Depending on connection result function returns either ``true`` or ``false`` (``boolean`` type).

*Example code:*

.. code:: cpp

    #include <ESP8266WiFi.h>

    void setup(void)
    {
      Serial.begin(115200);
      Serial.println();

      Serial.printf("Wi-Fi mode set to WIFI_STA %s\n", WiFi.mode(WIFI_STA) ? "" : "Failed!");
      Serial.print("Begin WPS (press WPS button on your router) ... ");
      Serial.println(WiFi.beginWPSConfig() ? "Success" : "Failed");

      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }
      Serial.println();
      Serial.print("Connected, IP address: ");
      Serial.println(WiFi.localIP());
    }

    void loop() {}

*Example output:*

::

    Wi-Fi mode set to WIFI_STA 
    Begin WPS (press WPS button on your router) ... Success
    .........
    Connected, IP address: 192.168.1.102

Smart Config
^^^^^^^^^^^^

The Smart Config connection of an ESP module an access point is done by sniffing for special packets that contain SSID and password of desired AP. To do so the mobile device or computer should have functionality of broadcasting of encoded SSID and password.

The following three functions are provided to implement Smart Config.

Start smart configuration mode by sniffing for special packets that contain SSID and password of desired Access Point. Depending on result either ``true`` or \`false is returned.

.. code:: cpp

    beginSmartConfig() 

Query Smart Config status, to decide when stop configuration. Function returns either ``true`` or ``false of``\ boolean\` type.

.. code:: cpp

    smartConfigDone()

Stop smart config, free the buffer taken by ``beginSmartConfig()``. Depending on result function return either ``true`` or ``false`` of ``boolean`` type.

.. code:: cpp

    stopSmartConfig() 

For additional details regarding Smart Config please refer to `ESP8266 API User Guide <http://bbs.espressif.com/viewtopic.php?f=51&t=1023>`__.
