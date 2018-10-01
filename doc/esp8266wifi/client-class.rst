Clase Client
------------

Metodos documentados para `Client <https://www.arduino.cc/en/Reference/WiFiClientConstructor>`__ en `Arduino <https://github.com/arduino/Arduino>`__

1.  `WiFiClient() <https://www.arduino.cc/en/Reference/WiFiClient>`__
2.  `connected() <https://www.arduino.cc/en/Reference/WiFiClientConnected>`__
3.  `connect() <https://www.arduino.cc/en/Reference/WiFiClientConnect>`__
4.  `write() <https://www.arduino.cc/en/Reference/WiFiClientWrite>`__
5.  `print() <https://www.arduino.cc/en/Reference/WiFiClientPrint>`__
6.  `println() <https://www.arduino.cc/en/Reference/WiFiClientPrintln>`__
7.  `available() <https://www.arduino.cc/en/Reference/WiFiClientAvailable>`__
8.  `read() <https://www.arduino.cc/en/Reference/WiFiClientRead>`__
9.  `flush() <https://www.arduino.cc/en/Reference/WiFiClientFlush>`__
10. `stop() <https://www.arduino.cc/en/Reference/WiFIClientStop>`__

Los métodos y propiedades descritos más abajo son específicos de ESP8266. No están cubiertos en la documentación de `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__. Antes de que estén complétamente documentados, consulte la información a continuación.

flush y stop
~~~~~~~~~~~~~~
``flush(timeoutMs)`` y ``stop(timeoutMs)`` ambos tienen ahora argumentos opcionales: ``timeout`` en millisegundos y ambos devuelven un booleano.

El valor de entrada por defecto es 0 significa que el valor efectivo se deja a discreción del implementador.
``flush()`` devolviendo ``true`` indica que los datos de salida han sido efectivamente enviados y ``false`` que se ha superado el tiempo de espera.

``stop()`` devuelve ``false`` en caso de problemas cuando se cierra un cliente (por ejemplo por fuera de tiempo ``flush``). Dependiendo de la implementación, el parámetro puede pasársele a ``flush()``.

setNoDelay
~~~~~~~~~~

.. code:: cpp

    setNoDelay(nodelay)

Con ``nodelay`` establecido a ``true``, esta función desactivará el `Algoritmo de Nagle <https://es.wikipedia.org/wiki/Algoritmo_de_Nagle>`__.

Este algoritmo está destinado a reducir el tráfico TCP/IP de pequeños paquetes enviados a través de la red combinando varios mensajes salientes pequeños y enviándolos todos a la vez. La desventaja de tal enfoque es que efectivamente se retrasan los mensajes individuales hasta que se ensamble un paquete lo suficientemente grande.

*Ejemplo:*

.. code:: cpp

    client.setNoDelay(true);

getNoDelay
~~~~~~~~~~
Devuelve si NoDelay está habilitado o no para la conexión actual.

setSync
~~~~~~~
Esta es una API experimental que configurará al cliente en modo sincronizado.

En este modo, cada ``write()`` se vacía. Significa que después de una llamada a ``write()``, los datos están asegurados para ser recibidos donde fueron enviados (es decir ``flush`` semántica).

Cuando se establece en ``true`` en la implementación ``WiFiClient``:
- Ralentiza las transferencias e implícitamente desactiva el algoritmo de Nagle.
- También permite evitar una copia temporal de datos que de otra manera consumiría al menos `` TCP_SND_BUF`` = (2 * `` MSS``) bytes por conexión.

getSync
~~~~~~~
Devuelve si la sincronización está habilitada o no para la conexión actual.

setDefaultNoDelay and setDefaultSync
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Esto establece el valor predeterminado para ``setSync`` y ``setNoDelay`` para cada instancia futura de ``WiFiClient`` (incluidos los que vienen de ``WiFiServer.available()`` por defecto).

Los valores predeterminados son false para ``NoDelay`` y ``Sync``.

Esto significa que Nagle está habilitado de forma predeterminada *para todas las conexiones nuevas*.

getDefaultNoDelay and getDefaultSync
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Devuelve los valores que se utilizarán como predeterminados para NoDelay y Sync para todas las conexiones futuras.

Otras llamadas a funciones
~~~~~~~~~~~~~~~~~~~~

.. code:: cpp

    uint8_t  status () 
    virtual size_t  write (const uint8_t *buf, size_t size) 
    size_t  write_P (PGM_P buf, size_t size) 
    size_t  write (Stream &stream) 
    size_t  write (Stream &stream, size_t unitSize) __attribute__((deprecated)) 
    virtual int  read (uint8_t *buf, size_t size) 
    virtual int  peek () 
    virtual size_t  peekBytes (uint8_t *buffer, size_t length) 
    size_t  peekBytes (char *buffer, size_t length) 
    virtual  operator bool () 
    IPAddress  remoteIP () 
    uint16_t  remotePort () 
    IPAddress  localIP () 
    uint16_t  localPort () 

La documentación para las funciones anteriores aún no se ha realizado.

Consulte la sección separada con `ejemplos <client-examples.rst>`__ dedicados específicamente a la clase Client.
