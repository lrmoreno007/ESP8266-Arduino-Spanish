Clase genérica
-------------

Los métodos y propiedades que se describen en esta sección son específicos de ESP8266. No están cubiertos en la documentación de la librería WiFi de Arduino. Antes de que estén completamente documentados, consulte la información a continuación.

onEvent
~~~~~~~

.. code:: cpp

    void  onEvent (WiFiEventCb cb, WiFiEvent_t event=WIFI_EVENT_ANY) __attribute__((deprecated)) 

Para ver como se utiliza ``onEvent`` comprueba el sketch de ejemplo `WiFiClientEvents.ino <https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/examples/WiFiClientEvents/WiFiClientEvents.ino>`__ disponible dentro de la carpeta de ejemplos de la librería ESP8266WiFi.

WiFiEventHandler
~~~~~~~~~~~~~~~~

.. code:: cpp

    WiFiEventHandler  onStationModeConnected (std::function< void(const WiFiEventStationModeConnected &)>) 
    WiFiEventHandler  onStationModeDisconnected (std::function< void(const WiFiEventStationModeDisconnected &)>) 
    WiFiEventHandler  onStationModeAuthModeChanged (std::function< void(const WiFiEventStationModeAuthModeChanged &)>) 
    WiFiEventHandler  onStationModeGotIP (std::function< void(const WiFiEventStationModeGotIP &)>) 
    WiFiEventHandler  onStationModeDHCPTimeout (std::function< void(void)>) 
    WiFiEventHandler  onSoftAPModeStationConnected (std::function< void(const WiFiEventSoftAPModeStationConnected &)>) 
    WiFiEventHandler  onSoftAPModeStationDisconnected (std::function< void(const WiFiEventSoftAPModeStationDisconnected &)>) 

Para ver un ejemplo de aplicación con ``WiFiEventHandler``, comprueba la sección separada con `ejemplos <generic-examples.rst>`__ dedicados específicamente a la clase genérica.

persistent
~~~~~~~~~~

.. code:: cpp

    WiFi.persistent (persistent) 


El módulo puede volver a conectarse a la red Wi-Fi utilizada por última vez al encenderse o restablecerse basándose en configuraciones almacenadas en sectores específicos de la memoria flash. De forma predeterminada, estas configuraciones se escriben en la flash cada vez que se usan en funciones como ``WiFi.begin(ssid, password)``. Esto sucede sin importar si el SSID o la contraseña han sido cambiados realmente.

Esto podría ocasionar un desgaste de la memoria flash dependiendo de la frecuencia con la que se invocan dichas funciones.

Configurando ``persistent`` a ``false`` hará que SSID / contraseña se grabe solo si los valores actualmente utilizados no coinciden con los que ya están almacenados en flash.

Tenga en cuenta que las funciones ``WiFi.disconnect`` o ``WiFi.softAPdisconnect`` restablecen el SSID/contraseña actualmente utilizado. Si ``persistent`` está establecido en ``false``, entonces el uso de estas funciones no afectará el SSID/contraseña almacenado en flash.

Para obtener más información sobre esta funcionalidad y por qué se ha introducido, consulte el informe de problemas `#1054 <https://github.com/esp8266/Arduino/issues/1054>`__.

mode
~~~~

.. code:: cpp

    WiFi.mode(m) 
    WiFi.getMode() 

-  ``WiFi.mode(m)``: establece el modo a ``WIFI_AP``, ``WIFI_STA``, ``WIFI_AP_STA`` o ``WIFI_OFF``
-  ``WiFi.getMode()``: devuelve el modo WiFi actual (alguno de los cuatro anteriores)

Otras llamadas a funciones
~~~~~~~~~~~~~~~~~~~~

.. code:: cpp

    int32_t  channel (void) 
    bool  setSleepMode (WiFiSleepType_t type) 
    WiFiSleepType_t  getSleepMode () 
    bool  setPhyMode (WiFiPhyMode_t mode) 
    WiFiPhyMode_t  getPhyMode () 
    void  setOutputPower (float dBm) 
    WiFiMode_t  getMode () 
    bool  enableSTA (bool enable) 
    bool  enableAP (bool enable) 
    bool  forceSleepBegin (uint32 sleepUs=0) 
    bool  forceSleepWake () 
    int  hostByName (const char *aHostname, IPAddress &aResult)

Documentation for the above functions is not yet prepared.

Consulte la sección separada con `ejemplos <generic-examples.rst>`__ dedicados específicamente a la clase genérica.
