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

Opcionalmente puedes usar el paquete *staging* desde el siguiente link: ``http://arduino.esp8266.com/staging/package_esp8266com_index.json``. Este contiene nuevas características, pero a la misma vez algo puede no funcionar.

Para mas información sobre el Gestor de Tarjetas, ver:

- https://www.arduino.cc/es/guide/cores

Usando la versión git
-----------------

Esta es la instalación sugerida para contribuidores y desarrolladores de librerías.

Prerequisitos
~~~~~~~~~~~~~

-  Arduino 1.6.8 (o mas moderno, versión actual funcionando 1.8.5)
-  git
-  Python_ 2.7 (http://python.org)
-  terminal, console, o interprete de comandos (dependiendo de tu OS)
-  Conexión a Internet

Instrucciones - Windows 10
~~~~~~~~~~~~
- Primero, asegúrese de que no tiene la biblioteca ESP8266 instalada con el Gestor de Tarjetas (ver arriba)

- Instala git para Windows (si no lo tiene ya; ver https://git-scm.com/download/win)

-  Abre un terminal y ve al directorio de Arduino por defecto. Este suele ser tu directorio *sketchbook* (normalmente ``C:\users\{username}\Documents\Arduino``), la variable de entorno ``%USERPROFILE%`` normalmente contiene ``C:\users\{username}``).

-  Clona este repositorio al directorio hardware/esp8266com/esp8266.

   .. code:: bash

       cd %USERPROFILE%\Documents\Arduino\
       if not exist hardware mkdir hardware
       cd hardware
       if not exist esp8266com mkdir esp8266com
       cd esp8266com
       git clone https://github.com/esp8266/Arduino.git esp8266
       
   Deberías acabar con la siguiente estructura de directorios en ``C:\Users\{your username}\Documents\``:

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

-  Inicialice los submodulos
   
   .. code:: bash

       cd %USERPROFILE%\Documents\hardware\esp8266com\esp8266
       git submodule update --init   
  
   Si tiene mensajes de error sobre archivos faltantes relacionados con ``SoftwareSerial`` durante el proceso de compilación, debe ser porque omitió este paso y es necesario.
  
-  Descarga las herramientas binarias:

   .. code:: bash

       cd esp8266/tools
       python get.py

-  Reinicia Arduino.

- Si utiliza el IDE de Arduino para Visual Studio (https://www.visualmicro.com/), asegúrese de hacer clic en Herramientas - Visual Micro - Volver a analizar las cadenas de herramientas y las bibliotecas

- Cuando actualice más tarde su biblioteca local, vaya al directorio esp8266 y haga un git pull:

  .. code:: bash

       cd %USERPROFILE%\Documents\hardware\esp8266com\esp8266
       git status
       git pull

Tenga en cuenta que podría, en teoría, instalar en ``C:\Archivos de programa (x86)\Arduino\hardware`` sin embargo, esto tiene implicaciones de seguridad, por no mencionar que el directorio a menudo se pierde al reinstalar el IDE de Arduino. Tiene la ventaja (o el inconveniente, según su perspectiva) de estar disponible para todos los usuarios en su PC que utilizan Arduino.

Instrucciones - Otros Sistemas Operativos
~~~~~~~~~~~~

-  Abra un terminal y ve al directorio de Arduino. Este puede ser su directorio *sketchbook* (normalmente ``<Documents>/Arduino``), o el directorio de instalación de Arduino mismamente, es tu elección.

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

-  Inicialice los submodulos
   
   .. code:: bash

       cd %USERPROFILE%\Documents\hardware\esp8266com\esp8266
       git submodule update --init   
  
   Si tiene mensajes de error sobre archivos faltantes relacionados con ``SoftwareSerial`` durante el proceso de compilación, debe ser porque omitió este paso y es necesario.

-  Descarga las herramientas binarias:

   .. code:: bash

       cd esp8266/tools
       python get.py

-  Reinicia Arduino.

- Cuando actualice más tarde su biblioteca local, vaya al directorio esp8266 y haga un git pull:

  .. code:: bash

       cd hardware\esp8266com\esp8266
       git status
       git pull
