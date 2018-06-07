Instalación
==========

Gestor de Tarjetas
--------------

Este es el método elegido para los usuarios finales.

Prerequisitos
~~~~~~~~~~~~~

-  Arduino 1.6.8, descárgalo de la página `web de Arduino <https://www.arduino.cc/en/Main/OldSoftwareReleases#previous>`__ :
-  Conexión a internet

Instrucciones
~~~~~~~~~~~~

-  Inicia Arduino y abre la ventana Preferencias.
-  Introduce ``http://arduino.esp8266.com/stable/package_esp8266com_index.json`` en la casilla *Gestor de URLs Adicionales de Tarjetas*. Puedes añadir múltiples URLs separándolas con comas.
-  Abre en gestor de tarjetas desde Herramientas > "Última tarjeta seleccionada" > Gestor de tarjetas y busca la plataforma *esp8266*.
-  Seleccione la versión que desee de la lista.
-  Click en el botón *Instalar*.
-  No olvides seleccionar tu tarjeta ESP8266 desde Herramientas > Menú de tarjetas tras la instalación.

Opcionalmente puedes usar el paquete *staging* desde el siguiente link:
``http://arduino.esp8266.com/staging/package_esp8266com_index.json``.
Este contiene nuevas características, pero a la misma vez algo puede no funcionar.

Usando la versión git
-----------------

Esta es la instalación sugerida para contribuidores y desarrolladores de librerías.

Prerequisitos
~~~~~~~~~~~~~

-  Arduino 1.6.8 (o mas moderno, si sabes lo que estás haciendo)
-  git
-  python 2.7
-  terminal, console, o interprete de comandos (dependiendo de tu OS)
-  Conexión a Internet

Instrucciones
~~~~~~~~~~~~

-  Abre un terminal y ve al directorio de Arduino. Este puede ser tu directorio *sketchbook* (normalmente ``<Documents>/Arduino``), o el directorio de instalación de Arduino mismamente, es tu elección.
-  Clona este repositorio al directorio hardware/esp8266com/esp8266.
   Alternativamente, clonalo en otro lugar y crea un enlace simbólico, si tu sistema operativo lo admite.

   .. code:: bash

       cd hardware
       mkdir esp8266com
       cd esp8266com
       git clone https://github.com/esp8266/Arduino.git esp8266

   Deberías acabar con la siguiente estructura de directorios:

   .. code:: bash

       Arduino
       |
       --- hardware
           |
           --- esp8266com
               |
               --- esp8266
                   |
                   --- bootloaders
                   --- cores
                   --- doc
                   --- libraries
                   --- package
                   --- tests
                   --- tools
                   --- variants
                   --- platform.txt
                   --- programmers.txt
                   --- README.md
                   --- boards.txt
                   --- LICENSE

-  Descarga las herramientas binarias:

   .. code:: bash

       cd esp8266/tools
       python get.py

-  Reinicia Arduino.
