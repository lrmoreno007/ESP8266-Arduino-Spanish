Clase Scan
~~~~~~~~~~

Esta clase está representada en la `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__ por la función `scanNetworks() <https://www.arduino.cc/en/Reference/WiFiScanNetworks>`__. Los desarrolladores del core ESP8266/Arduino han extendido esta funcionalidad añadiendo métodos y propiedades.

La documentación de esta clase está dividida en dos partes. La primera cubre el escaneo de redes disponibles. la segunda describe que información se colecciona durante el proceso de escaneo y como acceder a ella.

Escaneo de redes
~~~~~~~~~~~~~~~~~

El escaneo en busca de redes tarda cientos de milisegundos en completarse. Esto se puede hacer en una sola ejecución cuando estamos activando el proceso de escaneo, esperando la finalización y proporcionando resultados, todo con una sola función. Otra opción es dividir esto en pasos, cada uno hecho por una función separada. De esta forma podemos ejecutar otras tareas mientras el escaneo está en progreso. Esto se llama escaneo asincrónico. Ambos métodos de escaneo se documentan a continuación.

scanNetworks
^^^^^^^^^^^^

Busca redes Wi-Fi disponibles en una sola ejecución y devuelve la cantidad de redes descubiertas.

.. code:: cpp

    WiFi.scanNetworks() 

Existe una `sobrecarga <https://en.wikipedia.org/wiki/Function_overloading>`__ de esta función que acepta dos parámetros opcionales para proporcionar una funcionalidad extendida de escaneo asincrónico, así como también la búsqueda de redes ocultas.

.. code:: cpp

    WiFi.scanNetworks(async, show_hidden) 

Ambos parámetros son de tipo ``boolean``. Proporcionan la siguiente funcionalidad: 

* ``asysnc`` - si se establece en ``true``, la exploración se iniciará en segundo plano y la función se cerrará sin esperar el resultado. Para comprobar el resultado utilice la función separada ``scanComplete`` que se describe a continuación. 

* ``show_hidden`` - establézcalo en ``true`` para incluir las redes con SSID oculto en los resultados de escaneo.

scanComplete
^^^^^^^^^^^^

Comprueba los resultados del escaneo asíncrono.

.. code:: cpp

    WiFi.scanComplete() 

Cuando se completa la función devuelve la cantidad de redes descubiertas.

Si el escaneo aún no ha acabado, el valor devuelto es < 0 de la siguiente manera: 

* Escaneo todavía en progreso: -1 

* El escaneo no se ha activado: -2

scanDelete
^^^^^^^^^^

Borra los últimos resultados de escaneo de la memoria.

.. code:: cpp

    WiFi.scanDelete() 

scanNetworksAsync
^^^^^^^^^^^^^^^^^

Comienza el escaneo de redes WiFi disponibles. Al completar ejecuta otra función.

.. code:: cpp

    WiFi.scanNetworksAsync(onComplete, show_hidden) 

Parámetros de la función:

* ``onComplete`` - el evento que se ejecutara cuando acabe el escaneo
* ``show_hidden`` - parámetro opcional ``boolean``, establecelo a ``true`` para escanear redes ocultas.

*Código de ejemplo:*

.. code:: cpp

    #include "ESP8266WiFi.h"

    void prinScanResult(int networksFound){
      Serial.printf("%d redes esncontradas\n", networksFound);
      for (int i = 0; i < networksFound; i++)
      {
        Serial.printf("%d: %s, Ch:%d (%ddBm) %s\n", i + 1, WiFi.SSID(i).c_str(), WiFi.channel(i), WiFi.RSSI(i), WiFi.encryptionType(i) == ENC_TYPE_NONE ? "open" : "");
      }
    }


    void setup()
    {
      Serial.begin(115200);
      Serial.println();

      WiFi.mode(WIFI_STA);
      WiFi.disconnect();
      delay(100);

      WiFi.scanNetworksAsync(prinScanResult);
    }


    void loop() {}

*Ejemplo de salida:*

::

    5 redes encontradas
    1: Tech_D005107, Ch:6 (-72dBm)
    2: HP-Print-A2-Photosmart 7520, Ch:6 (-79dBm)
    3: ESP_0B09E3, Ch:9 (-89dBm) open
    4: Hack-4-fun-net, Ch:9 (-91dBm)
    5: UPC Wi-Free, Ch:11 (-79dBm)

Mostrando resultados
~~~~~~~~~~~~

Las funciones a continuación brindan acceso al resultado del escaneo. No importa si el escaneo se ha realizado en modo síncrono o asíncrono, los resultados del escaneo están disponibles utilizando la misma API.

Los resultados individuales son accesibles al proporcionar un ``networkItem`` que identifica el índice (basado en cero) de la red descubierta.

SSID
^^^^

Devuelve el SSID de una red descubierta durante el escaneo.

.. code:: cpp

    WiFi.SSID(networkItem) 

El retorno de SSID es del tipo ``String``. ``networkItem`` es un índice basado en cero de las redes descubiertas durante el escaneo.

encryptionType
^^^^^^^^^^^^^^

Devuelve el tipo de encriptado de una red descubierta durante el escaneo.

.. code:: cpp

    WiFi.encryptionType(networkItem) 

La función devuelve un número de tipo de encriptado, de la siguiente manera: 

* 5 : ``ENC_TYPE_WEP`` - WEP 

* 2 : ``ENC_TYPE_TKIP`` - WPA / PSK 

* 4 : ``ENC_TYPE_CCMP`` - WPA2 / PSK

* 7 : ``ENC_TYPE_NONE`` - open network

* 8 : ``ENC_TYPE_AUTO`` - WPA / WPA2 / PSK

``networkItem`` es un índice basado en cero de las redes descubiertas durante el escaneo.

RSSI
^^^^

Devuelve el `RSSI <https://en.wikipedia.org/wiki/Received_signal_strength_indication>`__ (Indicación de la fuerza de la señal recivida) de una red descubierta durante el escaneo.

.. code:: cpp

    WiFi.RSSI(networkItem) 

El retorno de RSSI es del tipo ``int32_t``. ``networkItem`` es un índice basado en cero de las redes descubiertas durante el escaneo.

BSSID
^^^^^

Devuelve el `BSSID <https://es.wikipedia.org/wiki/BSSID>`__ con la dirección MAC de una red descubierta durante el escaneo.

.. code:: cpp

    WiFi.BSSID(networkItem) 

La función devuelve un puntero a la dirección de la memoria (un array ``uint8_t`` con el tamaño de 6 elementos) donde se guarda la BSSID.

Si no te gustan los punteros, hay otra versión que devuelve un ``String``.

.. code:: cpp

    WiFi.BSSIDstr(networkItem) 

``networkItem`` es un índice basado en cero de las redes descubiertas durante el escaneo.

channel
^^^^^^^

Devuelve el canal de una red descubierta durante el escaneo.

.. code:: cpp

    WiFi.channel(networkItem) 

El retorno del canal es del tipo ``int32_t``. ``networkItem`` es un índice basado en cero de las redes descubiertas durante el escaneo.

isHidden
^^^^^^^^

Devuelve información de si una red descubierta durante el escaneo es oculta o no.

.. code:: cpp

    WiFi.isHidden(networkItem)

El retorno es un valor del tipo ``bolean`` y ``true`` significa que la red es oculta. ``networkItem`` es un índice basado en cero de las redes descubiertas durante el escaneo.

getNetworkInfo
^^^^^^^^^^^^^^

Devuelve toda la información discutida anteriormente en este capítulo en una sola llamada a una función.

.. code:: cpp

    WiFi.getNetworkInfo(networkItem, &ssid, &encryptionType, &RSSI, *&BSSID, &channel, &isHidden) 

``networkItem`` es un índice basado en cero de las redes descubiertas durante el escaneo. Todos los demás parámetros de entrada se pasan a la función por referencia. Por lo tanto, se actualizarán con los valores reales recuperados para un ``networkItem`` particular. La función en sí devuelve un ``boolean``, con `` true`` o `` false`` para confirmar si la recuperación de información fue exitosa o no.

*Código de ejemplo:*

.. code:: cpp

    int n = WiFi.scanNetworks(false, true);

    String ssid;
    uint8_t encryptionType;
    int32_t RSSI;
    uint8_t* BSSID;
    int32_t channel;
    bool isHidden;

    for (int i = 0; i < n; i++)
    {
      WiFi.getNetworkInfo(i, ssid, encryptionType, RSSI, BSSID, channel, isHidden);
      Serial.printf("%d: %s, Ch:%d (%ddBm) %s %s\n", i + 1, ssid.c_str(), channel, RSSI, encryptionType == ENC_TYPE_NONE ? "open" : "", isHidden ? "hidden" : "");
    }

*Ejemplo de salida:*

::

    6 network(s) found
    1: Tech_D005107, Ch:6 (-72dBm)
    2: HP-Print-A2-Photosmart 7520, Ch:6 (-79dBm)
    3: ESP_0B09E3, Ch:9 (-89dBm) open
    4: Hack-4-fun-net, Ch:9 (-91dBm)
    5: , Ch:11 (-77dBm)  hidden
    6: UPC Wi-Free, Ch:11 (-79dBm)

Consulte la sección separada con `ejemplos <scan-examples.rst>`__ dedicados específicamente a la clase Scan.
