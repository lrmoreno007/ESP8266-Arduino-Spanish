Clase UDP
---------

Metodos documentados para la `Clase WiFiUDP <https://www.arduino.cc/en/Reference/WiFiUDPConstructor>`__ en `Arduino <https://github.com/arduino/Arduino>`__

1.  `begin() <https://www.arduino.cc/en/Reference/WiFiUDPBegin>`__
2.  `available() <https://www.arduino.cc/en/Reference/WiFiUDPAvailable>`__
3.  `beginPacket() <https://www.arduino.cc/en/Reference/WiFiUDPBeginPacket>`__
4.  `endPacket() <https://www.arduino.cc/en/Reference/WiFiUDPEndPacket>`__
5.  `write() <https://www.arduino.cc/en/Reference/WiFiUDPWrite>`__
6.  `parsePacket() <https://www.arduino.cc/en/Reference/WiFiUDPParsePacket>`__
7.  `peek() <https://www.arduino.cc/en/Reference/WiFiUDPPeek>`__
8.  `read() <https://www.arduino.cc/en/Reference/WiFiUDPRead>`__
9.  `flush() <https://www.arduino.cc/en/Reference/WiFiUDPFlush>`__
10. `stop() <https://www.arduino.cc/en/Reference/WiFIUDPStop>`__
11. `remoteIP() <https://www.arduino.cc/en/Reference/WiFiUDPRemoteIP>`__
12. `remotePort() <https://www.arduino.cc/en/Reference/WiFiUDPRemotePort>`__

Los métodos y propiedades descritos más abajo son específicos de ESP8266. No están cubiertos en la documentación de `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__. Antes de que estén complétamente documentados, consulte la información a continuación.

Multicast UDP
~~~~~~~~~~~~~

.. code:: cpp

    uint8_t  beginMulticast (IPAddress interfaceAddr, IPAddress multicast, uint16_t port) 
    virtual int  beginPacketMulticast (IPAddress multicastAddress, uint16_t port, IPAddress interfaceAddress, int ttl=1) 
    IPAddress  destinationIP () 
    uint16_t  localPort ()

La clase ``WiFiUDP`` admite el envío y recepción de paquetes de multidifusión en la interfaz STA. Al enviar un paquete de multidifusión, reemplace ``udp.beginPacket (addr, port)`` con ``udp.beginPacketMulticast (addr, port, WiFi.localIP ())``. Al escuche paquetes de multidifusión, reemplace ``udp.begin(port)`` con ``udp.beginMulticast(WiFi.localIP(), multicast_ip_addr, port)``. Puede usar ``udp.destinationIP()`` para saber si el paquete recibido se envió a la dirección de multidifusión o unicast.

Consulte la sección separada con `ejemplos <udp-examples.rst>`__ dedicados específicamente a la clase UDP.
