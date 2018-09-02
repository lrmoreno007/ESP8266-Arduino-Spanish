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

    WiFi.persistent(persistent) 


El ESP8266 puede volver a conectarse a la red Wi-Fi utilizada por última vez al encenderse o establece el mismo punto de acceso al encender o restablecer. De forma predeterminada, estas configuraciones se escriben en sectores específicos de la flash cada vez que se cambian en ``WiFi.begin(ssid, password)`` o ``WiFi.softAP(ssid, passphrase, channel)``, y cuando se invoca a ``WiFi.disconnect`` o ``WiFi.softAPdisconnect``. Llamar con frecuencia a estas funciones podría ocasionar desgaste en la memoria flash (consulte el issue `#1054 <https://github.com/esp8266/Arduino/issues/1054>`__).

Si establece ``WiFi.persistent(false)``, entonces ``WiFi.begin``, ``WiFi.disconnect``, ``WiFi.softAP`` o ``WiFi.softAPdisconnect`` solo cambian los ajustes actuales de Wi-Fi en memoria y no afecta la configuración de Wi-Fi almacenada en la memoria flash.


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

Documentación para las funciones anteriores aún no está preparada.

Consulte la sección separada con `ejemplos <generic-examples.rst>`__ dedicados específicamente a la clase genérica.
