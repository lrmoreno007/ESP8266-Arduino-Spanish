Preguntas - FAQ
=================

El propósito de este apartado "Preguntas - FAQ" es responder a las preguntas mas comunes en la sección `Issues <https://github.com/esp8266/Arduino/issues>`__ y en el `Foro de la comunidad ESP8266 <http://www.esp8266.com/>`__.

Siempre que sea posible, vamos directamente a la respuesta y la proporcionamos dentro de uno o dos párrafos. Si la respuesta es mas larga, verá un enlace para leer mas detalles.

Siéntase libre de contribuir si cree que alguna pregunta frecuente no se encuentra cubierta.

Obtengo el error "espcomm\_sync failed" cuando intento subir a mi ESP. ¿Como resuelvo este problema?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Este mensaje indica un problema al subir al módulo ESP mediante conexión por puerto serie. Existen varias posibles causas que dependen del tipo de módulo y de si tiene un convertidor serie independiente.

`Leer mas <a01-espcomm_sync-failed.rst>`__.

¿Porqué no aparece esptool en el menú "Programador"? ¿Como subo al ESP sin él?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No te preocupes por el menú "Programador" del IDE Arduino. No importa qué se seleccione en él, siempre está predeterminado para usar esptool.

Ref. `#138 <https://github.com/esp8266/Arduino/issues/138>`__, `#653 <https://github.com/esp8266/Arduino/issues/653>`__ y `#739 <https://github.com/esp8266/Arduino/issues/739>`__.

Mi ESP se bloquea al correr el programa. ¿Como lo resuelvo?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El programa puede bloquearse por un error software o hardware. Antes de abrir un nuevo issue, por favor realice una serie de comprobaciones iniciales.

`Leer mas <a02-my-esp-crashes.rst>`__.

¿Como puedo obtener algunos KBs extra en la flash?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La utilización de ``*printf()`` con floats está activada por defecto. Puedes salvar algunos KBs de flash utilizando la opción ``--nofloat`` con el generador de tarjetas:

``./tools/boards.txt.py --nofloat --boardsgen``

 Utiliza la opción del nivel de debug ``NoAssert-NDEBUG`` (en el menú Herramientas).
 
 `Leer mas <a05-board-generator.rst>`__.
 
Sobre WPS
~~~~~~~~~~

A partir de la versión 2.4.2 y superiores, no utilizar WPS libera ~4.5KB estra en heap.

En la versión 2.4.2 solo, WPS está desactivado por defecto y se requiere el Generador de tarjetas para activarlo:

``./tools/boards.txt.py --allowWPS --boardsgen``

`Leer mas <a05-board-generator.rst>`__.

Para platformIO (y posíblemente en otros entornos de desarrollo), también necesitas añadir la bandera de compilación (build flag): -D NO_EXTRA_4K_HEAP

La selección manual no es necesaria a partir de la versión 2.5.0 (y en la versión git). El WPS está siempre disponible y no usarlo libera ~4.5KB comparado con las versiones anteriores a 2.4.1 (incluida).

Esta librería de Arduino no funciona en ESP. ¿Como la hago funcionar?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Te gustaría usar una librería de Arduino con ESP8266 y no funciona. Si no se encuentra entre las bibliotecas verificadas para trabajar con ESP8266:

`Leer mas <a03-library-does-not-work.rst>`__.

En el IDE, para ESP-12E que tiene una flash de 4M, puedo seleccionar 4M (1M SPIFFS) o 4M (3M SPIFFS). No importa lo que seleccione, el IDE me dice que la capacidad máxima es de 1M. ¿Donde va mi flash?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

La razón de que no podamos tener mas de 1MB de código en la flash tiene que ver con limitaciones hardware. El hardware de la cache flash en el ESP8266 solo permite mapear 1MB de código en el espacio de direcciones de la CPU en cualquier momento dado. Puedes cambiar el desplazamiento de mapeo, por lo que técnicamente puede tener más de 1 MB total, pero cambiar esos "bancos" sobre la marcha no es fácil y eficiente, así que no nos molestamos en hacerlo. Además, nadie se ha quejado hasta ahora de que los aproximadamente 1 MB de espacio de código sea insuficiente para fines prácticos.

La opción de seleccionar 3M o 1M SPIFFS es para optimizar el tiempo de subida. Subir 3MB toma mas tiempo que subir 1MB. Otras capacidades de flash 2MB también pueden utilizarse con las APIs ``ESP.flashRead`` y ``ESP.flashWrite`` si es necesario.

He observado un caso en que ESP.restart() no funciona. ¿Cual es la razón para esto?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Verá este problema solo si después de subir el programa mediante puerto serie no realiza un reset físico (por ejemplo, reinicio de la alimentación). Para un dispositivo que se encuentre en ese estado, ``ESP.restart`` no funcionará. Aparentemente, el problema está causado por `uno de los registros internos que no se actualiza correctamente hasta el reseteo físico <https://github.com/esp8266/Arduino/issues/1017#issuecomment-200605576>`__. Este problema solo afecta a las subidas mediante puerto serie. Las subidas mediante OTA no se ven afectadas. Si está utilizando ``ESP.restart``, solo reinicie ESP físicamente una vez después de cada subida por puerto serie.

Ref. `#1017 <https://github.com/esp8266/Arduino/issues/1017>`__, `#1107 <https://github.com/esp8266/Arduino/issues/1107>`__, `#1782 <https://github.com/esp8266/Arduino/issues/1782>`__

¿Como solucionar el error "Board generic (platform esp8266, package esp8266) is unknown"?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Este error puede aparecer al cambiar entre los paquetes de instalación de ESP8266/Arduino `staging <https://github.com/esp8266/Arduino#staging-version->`__ y `stable <https://github.com/esp8266/Arduino#stable-version->`__, o tras actualizar de versión de paquete.

`Leer mas <a04-board-generic-is-unknown.rst>`__.

¿Cómo borrar PCBs TCP en estado de espera de tiempo?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Esto ya no es necesario:

Los PCBs en tiempo de estado de espera están limitados a 5 y se eliminan cuando ese número es excedido.

Ref.  `lwIP-v1.4 <https://github.com/esp8266/Arduino/commit/07f4d4c241df2c552899857f39a4295164f686f2#diff-f8258e71e25fb9985ca3799e3d8b88ecR399>`__,
`lwIP-v2 <https://github.com/d-a-v/esp82xx-nonos-linklayer/commit/420960dfc0dbe07114f7364845836ac333bc84f7>`__

Como información:

El estado Time-wait PCB ayuda al TCP a no confundir dos conexiones consecutivas con el mismo: IP de origen ip, puerto de origen, IP de destino y puerto de destino, cuando el primero ya está cerrado pero aún están llegando tarde durante segundos paquetes duplicados perdidos en internet. Limpiarlos artificialmente es una solución alternativa para ayudar a salvar heap preciosos.

La líneas siguientes son compatibles con ambas versiones de lwIP:

.. code:: cpp

    // no need for #include
    struct tcp_pcb;
    extern struct tcp_pcb* tcp_tw_pcbs;
    extern "C" void tcp_abort (struct tcp_pcb* pcb);
    
    void tcpCleanup (void) {
      while (tcp_tw_pcbs)
        tcp_abort(tcp_tw_pcbs);
    }

Ref.  `#1923 <https://github.com/esp8266/Arduino/issues/1923>`__

¿Por qué hay un generador de tarjetas y para que sirve?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El generador de tarjetas es una secuencia de comandos python originalmente destinada a facilitar el archivo de configuración `boards.txt` de Arduino IDE sobre la multitud de tarjetas disponibles, especialmente cuando los parámetros comunes deben actualizarse para todos ellos.

Este script también se usa para administrar opciones poco comunes que actualmente no están disponibles en el menú IDE.

`Leer mas <a05-board-generator.rst>`__.
