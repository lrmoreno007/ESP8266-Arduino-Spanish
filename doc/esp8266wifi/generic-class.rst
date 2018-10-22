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

Administración de energía WiFi, DTIM
~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code:: cpp
    bool setSleepMode (WiFiSleepType_t type, int listenInterval = 0)
    
El tipo de modo de suspensión es ``WIFI_NONE_SLEEP``, ``WIFI_LIGHT_SLEEP`` o ``WIFI_MODEM_SLEEP``. (``listenInterval`` apareció en esp8266-arduino core v2.5.0 utilizando la última revisión de V2 de nonos-sdk antes de V3)

Citando la hoja de datos de nonos-sdk:

* ``NONE``: desactiva el ahorro de energía

* ``LIGHT`` o ``MODEM``: tasa de temporizador TCP aumentada de 250 ms a 3 segundos

Cuando ``listenInterval`` se establece en 1..10, en el modo ``LIGHT`` o ``MODEM``, la estación se activa cada (DTIM-interval * ``listenInterval``). Esto salva energía pero la interfaz de la estación puede perder datos de transmisión. De lo contrario (valor predeterminado 0), la estación se activa a cada intervalo DTIM (configurado en el punto de acceso).

Citando Wikipedia:
Un mapa de indicación de tráfico de entrega (DTIM) es un tipo de mapa de indicación de tráfico (TIM) que informa a los clientes sobre la presencia de búferes de datos de multicast/broadcast en el punto de acceso. Se genera dentro de la beacon periódica a una frecuencia especificada por el intervalo DTIM. Las bbeacon son paquetes enviados por un punto de acceso para sincronizar una red inalámbrica.

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
