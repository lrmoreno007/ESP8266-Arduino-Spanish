Clase Server
------------

Métodos documentados para la `clase Server <https://www.arduino.cc/en/Reference/WiFiServerConstructor>`__ en `Arduino <https://github.com/arduino/Arduino>`__

1. `WiFiServer() <https://www.arduino.cc/en/Reference/WiFiServer>`__
2. `begin() <https://www.arduino.cc/en/Reference/WiFiServerBegin>`__
3. `available() <https://www.arduino.cc/en/Reference/WiFiServerAvailable>`__
4. `write() <https://www.arduino.cc/en/Reference/WiFiServerWrite>`__
5. `print() <https://www.arduino.cc/en/Reference/WiFiServerPrint>`__
6. `println() <https://www.arduino.cc/en/Reference/WiFiServerPrintln>`__

Los métodos y propiedades que se describen en esta sección son específicos de ESP8266. No están cubiertos en la documentación de la `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__. Antes de que estén complétamente documentados, consulte la información a continuación.

setNoDelay
~~~~~~~~~~

.. code:: cpp

    setNoDelay(nodelay)

Con ``nodelay`` establecido a ``true``, esta función desactivará el `Algoritmo de Nagle <https://es.wikipedia.org/wiki/Algoritmo_de_Nagle>`__.

Este algoritmo está destinado a reducir el tráfico TCP/IP de pequeños paquetes enviados a través de la red combinando varios mensajes salientes pequeños y enviándolos todos a la vez. La desventaja de tal enfoque es que efectivamente se retrasan los mensajes individuales hasta que se ensamble un paquete lo suficientemente grande.

*Ejemplo:*

.. code:: cpp

    server.begin();
    server.setNoDelay(true);

Por defecto, el valor de ``nodelay`` dependerá de ``WiFiClient::getDefaultNoDelay()`` global (actualmente falso de forma predeterminada).

Sin embargo, una llamada a ``wiFiServer.setNoDelay()`` anulará ``NoDelay`` para todos los ``WiFiClient`` nuevos proporcionados por la instancia de llamada (``wiFiServer``).

Otras llamas a funciones
~~~~~~~~~~~~~~~~~~~~

.. code:: cpp

    bool  hasClient () 
    bool  getNoDelay () 
    virtual size_t  write (const uint8_t *buf, size_t size) 
    uint8_t  status () 
    void  close () 
    void  stop ()

La documentación para las funciones anteriores aún no se ha realizado.

Consulte la sección separada con `ejemplos <server-examples.rst>`__ dedicados específicamente a la clase Server.
