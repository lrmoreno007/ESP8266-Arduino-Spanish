Actualizaciones OTA
===========


Introducción
------------

La actualización OTA (Over the Air - Por el aire) es el proceso de carga del firmware en el módulo ESP mediante una conexión Wi-Fi en lugar de un puerto serie. Dicha funcionalidad se volve extremadamente útil en caso de acceso físico limitado o nulo al módulo.

OTA puede usarse mediante:

-  `Arduino IDE <#arduino-ide>`__
-  `Buscador Web <#buscador-web>`__
-  `Servidor HTTP <#servidor-http>`__

La opción IDE de Arduino está destinada principalmente para la fase de desarrollo de software. Las otras dos opciones serían más útiles después de la implementación para proporcionar al módulo actualizaciones de la aplicación manualmente con un navegador web o automáticamente utilizando un servidor http.

En cualquier caso, la primera carga de firmware debe realizarse a través de un puerto serie. Si las rutinas OTA se implementan correctamente en un sketch, entonces todas las subidas siguientes se pueden realizar por el aire.

No hay seguridad impuesta en el proceso OTA de ser hackeado. Es responsabilidad del desarrollador garantizar que las actualizaciones solo se permitan desde fuentes legítimas/confiables. Una vez que se completa la actualización, el módulo se reinicia y se ejecuta el nuevo código. El desarrollador debe asegurarse de que la aplicación que se ejecuta en el módulo se cierre y reinicie de manera segura. Los capítulos a continuación proporcionan información adicional sobre la seguridad y protección del proceso OTA.

Seguridad
~~~~~~~~

El módulo debe exponerse de forma inalámbrica para actualizarse con un nuevo sketch. Eso plantea posibilidades de que el módulo sea hackeado y se cargue otro código. Para reducir la posibilidad de ser hackeado, considere proteger sus subidas con una contraseña, seleccionando cierto puerto OTA, etc.

Compruebe las funcionalidades proporcionadas con la librería `ArduinoOTA <https://github.com/esp8266/Arduino/tree/master/libraries/ArduinoOTA>`__ para mejorar la seguridad:

.. code:: cpp

    void setPort(uint16_t port);
    void setHostname(const char* hostname);
    void setPassword(const char* password);

Cierta funcionalidad de protección ya está incorporada y no requiere ninguna codificación adicional por parte del desarrollador. `ArduinoOTA <https://github.com/esp8266/Arduino/tree/master/libraries/ArduinoOTA>`__ y espota.py utiliza `Digest-MD5 <https://es.wikipedia.org/wiki/Digest_access_authentication>`__ para autentificar la subida. La integridad de los datos transferidos se verifica en el lado ESP utilizando un checksum `MD5 <https://es.wikipedia.org/wiki/MD5>`__.

Haga su propio análisis de riesgos y dependiendo de la aplicación, decida qué funciones de la librería implementa. Si es necesario, considere la implementación de otros medios de protección contra la piratería, por ejemplo: exponer el módulo para cargar solo según el programa específico, desencadenar OTA solo si el usuario presiona el botón dedicado "Actualizar" conectado al ESP, etc.

Protección
~~~~~~

El proceso OTA toma los recursos y el ancho de banda del ESP durante la carga. Luego se reinicia el módulo y se ejecuta un nuevo bsketch. Analiza y prueba cómo afecta la funcionalidad del sketch existente y nuevo.

Si el ESP se coloca en una ubicación remota y controla algún equipo, debe poner atención adicional sobre lo que sucede si el proceso de actualización interrumpe repentinamente la operación de este equipo. Por lo tanto, decida cómo poner este equipo en un estado seguro antes de iniciar la actualización. Por ejemplo, su módulo puede estar controlando el sistema de riego de un jardín en una secuencia. Si esta secuencia no se cierra correctamente y se deja abierta una válvula de agua, su jardín puede quedar inundado.

La siguientes funciones son proporcionadas en la librería `ArduinoOTA <https://github.com/esp8266/Arduino/tree/master/libraries/ArduinoOTA>`__, destinadas a manejar la funcionalidad de su aplicación durante etapas específicas de OTA o ante un error de OTA:

.. code:: cpp

    void onStart(OTA_CALLBACK(fn));
    void onEnd(OTA_CALLBACK(fn));
    void onProgress(OTA_CALLBACK_PROGRESS(fn));
    void onError(OTA_CALLBACK_ERROR (fn));

Requerimientos Básicos
~~~~~~~~~~~~~~~~~~

El tamaño del chip flash debe poder contener el boceto anterior (actualmente en ejecución) y el nuevo boceto (OTA) al mismo tiempo.

Tenga en cuenta que el sistema de archivos y la EEPROM, por ejemplo, también necesitan espacio (una vez), consulte el `Esquema de la memoria flash <../filesystem.rst#esquema-de-la-memoria-flash>`__.

.. code:: cpp

    ESP.getFreeSketchSpace();

puede usarse para comprobar el espacio libre para el nuevo sketch.

Para obtener una descripción general del diseño de la memoria, dónde se almacena el nuevo boceto y cómo se copia durante el proceso OTA, consulte `Proceso de actualización - Vista de la memoria <#proceso-de-actualizacion-vista-de-la-memoria>`__.

Los siguientes capítulos proporcionan más detalles y métodos específicos para hacer OTA..

Arduino IDE
-----------

La carga inalámbrica de módulos desde Arduino IDE está diseñada para los siguientes escenarios típicos: 

- durante el desarrollo del firmware como una alternativa más rápida a la carga en serie.
- para actualizar una pequeña cantidad de módulos.
- solo si los módulos están disponibles en la misma red que la computadora con Arduino IDE.

Requerimientos
~~~~~~~~~~~~

-  El ESP y el ordenador deben estar conectados a la misma red.

Ejemplo de Aplicación
~~~~~~~~~~~~~~~~~~~

Las siguientes instrucciones muestran la configuración de OTA en la placa NodeMCU 1.0 (módulo ESP-12E). Puedes usar cualquier otra placa asumiendo que cumple `Requerimientos Básicos <#requerimientos-basicos>`__ descritos anteriormente. Esta instrucción es válida para todos los sistemas operativos compatibles con Arduino IDE. Se han realizado capturas de pantalla en Windows 7 y es posible que vea pequeñas diferencias (como el nombre del puerto serie), si está usando Linux y MacOS.

1. Before you begin, please make sure that you have the following s/w
   installed:

   -  Arduino IDE 1.6.7 o posterior -
      https://www.arduino.cc/en/Main/Software
   -  Paquete de la plataforma esp8266/Arduino 2.0.0 o posterior - para instrucciones vaya a:
      https://github.com/lrmoreno007/ESP8266-Arduino-Spanish#instalando-mediante-el-gestor-de-tarjetas
   -  Python 2.7 - https://www.python.org/

      **Nota:** Los usuarios de Windows deben seleccionar "Agregar python.exe a la ruta" (ver más abajo, esta opción no está seleccionada de manera predeterminada).

      .. figure:: a-ota-python-configuration.png
         :alt: Configuración de la instalación de Python

2. Ahora prepare el sketch y la configuración para la carga a través del puerto serie.

   -  Inicie Arduino IDE y cargue el sketch BasicOTA.ino disponible en Archivo > Ejemplos > ArduinoOTA 
   |ota sketch selection|

   -  Actualice el SSID y la contraseña en el sketch, para que el módulo pueda unirse a su red Wi-Fi 
   |ota ssid pass entry|

   -  Configure los parámetros de carga como se muestra a continuación (es posible que deba ajustar la configuración si está utilizando un módulo diferente): 
   |ota serial upload config|

      **Nota:** Dependiendo de la versión del paquete de plataforma y la placa que tenga, puede ver ``Upload using:`` en el menú de arriba. Esta opción está inactiva y no importa lo que seleccione. Se dejó para compatibilidad con la implementación anterior de OTA y finalmente se eliminó en la versión 2.2.0 del paquete de plataforma.
      
3. Suba el sketch (Ctrl+U). Una vez hecho, abra el Monitor Serie (Ctrl+Shift+M) y compruebe si el módulo se ha unido a su red Wi-Fi:

   .. figure:: a-ota-upload-complete-and-joined-wifi.png
      :alt: Compruebe que el módulo ha entrado en la red



      **Nota:** El módulo ESP debe reiniciarse después de la carga por el puerto serie. De lo contrario, los siguientes pasos no funcionarán. El reinicio se puede hacer automáticamente después de abrir el monitor serie como se muestra en la captura de pantalla anterior. Depende de cómo tengas conectado DTR y RTS desde el convertidor USB-serie al ESP. Si el restablecimiento no se realiza automáticamente, hágalo presionando el botón de reset o reiniciando manualmente la alimentación. Para obtener más información sobre por qué debería hacerse esto, consulte `Preguntas frecuentes <../faq#he-observado-que-esprestart-no-funciona-cual-es-la-razón>`__ con respecto a ``ESP.restart()``.

4. Only if module is connected to network, after a couple of seconds, the esp8266-ota port will show up in Arduino IDE. Select port with IP address shown in the Serial Monitor window in previous step:

   .. figure:: a-ota-ota-port-selection.png
      :alt: Selection of OTA port

   **Note:** If OTA port does not show up, exit Arduino IDE, open it again and check if port is there. If it does not help, check your firewall and router settings. OTA port is advertised using mDNS service. To check if port is visible by your PC, you can use application like Bonjour Browser.

5. Now get ready for your first OTA upload by selecting the OTA port:

   .. figure:: a-ota-ota-upload-configuration.png
      :alt: Configuration of OTA upload

   **Note:** The menu entry ``Upload Speed:`` does not matter at this point as it concerns the serial port. Just left it unchanged.

6. If you have successfully completed all the above steps, you can upload (Ctrl+U) the same (or any other) sketch over OTA:

   .. figure:: a-ota-ota-upload-complete.png
      :alt: OTA upload complete

**Note:** To be able to upload your sketch over and over again using OTA, you need to embed OTA routines inside. Please use BasicOTA.ino as an example.

Password Protection
^^^^^^^^^^^^^^^^^^^

Protecting your OTA uploads with password is really straightforward. All you need to do, is to include the following statement in your code:

.. code:: cpp

    ArduinoOTA.setPassword((const char *)"123");

Where ``123`` is a sample password that you should replace with your own.

Before implementing it in your sketch, it is a good idea to check how it works using *BasicOTA.ino* sketch available under *File > Examples > ArduinoOTA*. Go ahead, open *BasicOTA.ino*, uncomment the above statement that is already there, and upload the sketch. To make troubleshooting easier, do not modify example sketch besides what is absolutely required. This is including original simple ``123`` OTA password. Then attempt to upload sketch again (using OTA). After compilation is complete, once upload is about to begin, you should see prompt for password as follows:

.. figure:: a-ota-upload-password-prompt.png
   :alt: Password prompt for OTA upload

Enter the password and upload should be initiated as usual with the only difference being ``Authenticating...OK`` message visible in upload log.

.. figure:: a-ota-upload-password-authenticating-ok.png
   :alt: Authenticating...OK during OTA upload

You will not be prompted for a reentering the same password next time. Arduino IDE will remember it for you. You will see prompt for password only after reopening IDE, or if you change it in your sketch, upload the sketch and then try to upload it again.

Please note, it is possible to reveal password entered previously in Arduino IDE, if IDE has not been closed since last upload. This can be done by enabling *Show verbose output during: upload* in *File > Preferences* and attempting to upload the module.

.. figure:: a-ota-upload-password-passing-upload-ok.png
   :alt: Verbose upload output with password passing in plain text

The picture above shows that the password is visible in log, as it is passed to *espota.py* upload script.

Another example below shows situation when password is changed between uploads.

.. figure:: a-ota-upload-password-passing-again-upload-ok.png
   :alt: Verbose output when OTA password has been changed between uploads

When uploading, Arduino IDE used previously entered password, so the upload failed and that has been clearly reported by IDE. Only then IDE prompted for a new password. That was entered correctly and second attempt to upload has been successful.

Troubleshooting
^^^^^^^^^^^^^^^

If OTA update fails, first step is to check for error messages that may be shown in upload window of Arduino IDE. If this is not providing any useful hints, try to upload again while checking what is shown by ESP on serial port. Serial Monitor from IDE will not be useful in that case. When attempting to open it, you will likely see the following:

.. figure:: a-ota-network-terminal.png
   :alt: Arduino IDE network terminal window

This window is for Arduino Yún and not yet implemented for esp8266/Arduino. It shows up because IDE is attempting to open Serial Monitor using network port you have selected for OTA upload.

Instead you need an external serial monitor. If you are a Windows user check out `Termite <http://www.compuphase.com/software_termite.htm>`__. This is handy, slick and simple RS232 terminal that does not impose RTS or DTR flow control. Such flow control may cause issues if you are using respective lines to toggle GPIO0 and RESET pins on ESP for upload.

Select COM port and baud rate on external terminal program as if you were using Arduino Serial Monitor. Please see typical settings for `Termite <http://www.compuphase.com/software_termite.htm>`__ below:

.. figure:: termite-configuration.png
   :alt: Termite settings


Then run OTA from IDE and look what is displayed on terminal. Successful `ArduinoOTA <#arduinoota>`__ process using BasicOTA.ino sketch looks like below (IP address depends on your network configuration):

.. figure:: a-ota-external-serial-terminal-output.png
   :alt: OTA upload successful - output on an external serial terminal

If upload fails you will likely see errors caught by the uploader, exception and the stack trace, or both.

Instead of the log as on the above screen you may see the following:

.. figure:: a-ota-external-serial-terminal-output-failed.png
   :alt: OTA upload failed - output on an external serial terminal

If this is the case, then most likely ESP module has not been reset after initial upload using serial port.

The most common causes of OTA failure are as follows:

- not enough physical memory on the chip (e.g. ESP01 with 512K flash memory is not enough for OTA).
- too much memory declared for SPIFFS so new sketch will not fit between existing sketch and SPIFFS – see `Update process - memory view <#update-process-memory-view>`__.
- too little memory declared in Arduino IDE for your selected board (i.e. less than physical size).
- not resetting the ESP module after initial upload using serial port.

For more details regarding flash memory layout please check `File system <../filesystem.rst>`__. For overview where new sketch is stored, how it is copied and how memory is organized for the purpose of OTA see `Update process - memory view <#update-process-memory-view>`__.

Buscador Web
-----------

Updates described in this chapter are done with a web browser that can be useful in the following typical scenarios:

-  after application deployment if loading directly from Arduino IDE is
   inconvenient or not possible,
-  after deployment if user is unable to expose module for OTA from
   external update server,
-  to provide updates after deployment to small quantity of modules when
   setting an update server is not practicable.

Requirements
~~~~~~~~~~~~

-  The ESP and the computer must be connected to the same network.

Implementation Overview
~~~~~~~~~~~~~~~~~~~~~~~

Updates with a web browser are implemented using ``ESP8266HTTPUpdateServer`` class together with ``ESP8266WebServer`` and ``ESP8266mDNS`` classes. The following code is required to get it work:

setup()

.. code:: cpp

        MDNS.begin(host);

        httpUpdater.setup(&httpServer);
        httpServer.begin();

        MDNS.addService("http", "tcp", 80);

loop()

.. code:: cpp

        httpServer.handleClient();

Application Example
~~~~~~~~~~~~~~~~~~~

The sample implementation provided below has been done using:

-  example sketch WebUpdater.ino available in
   ``ESP8266HTTPUpdateServer`` library,
-  NodeMCU 1.0 (ESP-12E Module).

You can use another module if it meets previously described `requirements <#basic-requirements>`__.

1. Before you begin, please make sure that you have the following
   software installed:

   -  Arduino IDE and 2.0.0-rc1 (of Nov 17, 2015) version of platform
      package as described under
      https://github.com/esp8266/Arduino#installing-with-boards-manager
   -  Host software depending on O/S you use:

      1. Avahi http://avahi.org/ for Linux
      2. Bonjour http://www.apple.com/support/bonjour/ for Windows
      3. Mac OSX and iOS - support is already built in / no any extra
         s/w is required

2. Prepare the sketch and configuration for initial upload with a serial
   port.

   -  Start Arduino IDE and load sketch WebUpdater.ino available under
      File > Examples > ESP8266HTTPUpdateServer.
   -  Update SSID and password in the sketch, so the module can join
      your Wi-Fi network.
   -  Open File > Preferences, look for “Show verbose output during:”
      and check out “compilation” option.

      .. figure:: ota-web-show-verbose-compilation.png
         :alt: Preferences - enabling verbose output during compilation

      **Note:** This setting will be required in step 5 below. You can
      uncheck this setting afterwards.

3. Upload sketch (Ctrl+U). Once done, open Serial Monitor (Ctrl+Shift+M)
   and check if you see the following message displayed, that contains
   url for OTA update.

   .. figure:: ota-web-serial-monitor-ready.png
      :alt: Serial Monitor - after first load using serial

   **Note:** Such message will be shown only after module successfully
   joins network and is ready for an OTA upload. Please remember about
   resetting the module once after serial upload as discussed in chapter
   `Arduino IDE <#arduino-ide>`__, step 3.

4. Now open web browser and enter the url provided on Serial Monitor,
   i.e. ``http://esp8266-webupdate.local/update``. Once entered, browser
   should display a form like below that has been served by your module.
   The form invites you to choose a file for update.

   .. figure:: ota-web-browser-form.png
      :alt: OTA update form in web browser

   **Note:** If entering ``http://esp8266-webupdate.local/update`` does
   not work, try replacing ``esp8266-webupdate`` with module’s IP
   address. For example, if your module IP is ``192.168.1.100`` then url
   should be ``http://192.168.1.100/update``. This workaround is useful
   in case the host software installed in step 1 does not work. If still
   nothing works and there are no clues on the Serial Monitor, try to
   diagnose issue by opening provided url in Google Chrome, pressing F12
   and checking contents of “Console” and “Network” tabs. Chrome
   provides some advanced logging on these tabs.

5. To obtain the file, navigate to directory used by Arduino IDE to
   store results of compilation. You can check the path to this file in
   compilation log shown in IDE debug window as marked below.

   .. figure:: ota-web-path-to-binary.png
      :alt: Compilation complete - path to binary file

6. Now press “Choose File” in web browser, go to directory identified in
   step 5 above, find the file “WebUpdater.cpp.bin” and upload it. If
   upload is successful, you will see “OK” on web browser like below.

   .. figure:: ota-web-browser-form-ok.png
      :alt: OTA update complete

   Module will reboot that should be visible on Serial Monitor:

   .. figure:: ota-web-serial-monitor-reboot.png
      :alt: Serial Monitor - after OTA update

   Just after reboot you should see exactly the same message
   ``HTTPUpdateServer ready! Open http:// esp8266-webupdate.local /update in your browser``
   like in step 3. This is because module has been loaded again with the
   same code – first using serial port, and then using OTA.

Once you are comfortable with this procedure, go ahead and modify WebUpdater.ino sketch to print some additional messages, compile it, locate new binary file and upload it using web browser to see entered changes on a Serial Monitor.

You can also add OTA routines to your own sketch following guidelines in `Implementation Overview <#implementation-overview>`__ above. If this is done correctly, you should be always able to upload new sketch over the previous one using a web browser.

In case OTA update fails dead after entering modifications in your sketch, you can always recover module by loading it over a serial port. Then diagnose the issue with sketch using Serial Monitor. Once the issue is fixed try OTA again.

Servidor HTTP
-----------

``ESPhttpUpdate`` class can check for updates and download a binary file from HTTP web server. It is possible to download updates from every IP or domain address on the network or Internet.

Requirements
~~~~~~~~~~~~

-  web server

Arduino code
~~~~~~~~~~~~

Simple updater
^^^^^^^^^^^^^^

Simple updater downloads the file every time the function is called.

.. code:: cpp

    ESPhttpUpdate.update("192.168.0.2", 80, "/arduino.bin");

Advanced updater
^^^^^^^^^^^^^^^^

Its possible to point update function to a script at the server. If version string argument is given, it will be sent to the server. Server side script can use this to check if update should be performed.

Server side script can respond as follows: - response code 200, and send the firmware image, - or response code 304 to notify ESP that no update is required.

.. code:: cpp

    t_httpUpdate_return ret = ESPhttpUpdate.update("192.168.0.2", 80, "/esp/update/arduino.php", "optional current version string here");
    switch(ret) {
        case HTTP_UPDATE_FAILED:
            Serial.println("[update] Update failed.");
            break;
        case HTTP_UPDATE_NO_UPDATES:
            Serial.println("[update] Update no Update.");
            break;
        case HTTP_UPDATE_OK:
            Serial.println("[update] Update ok."); // may not called we reboot the ESP
            break;
    }

Server request handling
~~~~~~~~~~~~~~~~~~~~~~~

Simple updater
^^^^^^^^^^^^^^

For the simple updater the server only needs to deliver the binary file for update.

Advanced updater
^^^^^^^^^^^^^^^^

For advanced update management a script needs to run at the server side, for example a PHP script. At every update request the ESP sends some information in HTTP headers to the server.

Example header data:

::

        [HTTP_USER_AGENT] => ESP8266-http-Update
        [HTTP_X_ESP8266_STA_MAC] => 18:FE:AA:AA:AA:AA
        [HTTP_X_ESP8266_AP_MAC] => 1A:FE:AA:AA:AA:AA
        [HTTP_X_ESP8266_FREE_SPACE] => 671744
        [HTTP_X_ESP8266_SKETCH_SIZE] => 373940
        [HTTP_X_ESP8266_SKETCH_MD5] => a56f8ef78a0bebd812f62067daf1408a
        [HTTP_X_ESP8266_CHIP_SIZE] => 4194304
        [HTTP_X_ESP8266_SDK_VERSION] => 1.3.0
        [HTTP_X_ESP8266_VERSION] => DOOR-7-g14f53a19

With this information the script now can check if an update is needed. It is also possible to deliver different binaries based on the MAC address for example.

Script example:

.. code:: php

    <?PHP

    header('Content-type: text/plain; charset=utf8', true);

    function check_header($name, $value = false) {
        if(!isset($_SERVER[$name])) {
            return false;
        }
        if($value && $_SERVER[$name] != $value) {
            return false;
        }
        return true;
    }

    function sendFile($path) {
        header($_SERVER["SERVER_PROTOCOL"].' 200 OK', true, 200);
        header('Content-Type: application/octet-stream', true);
        header('Content-Disposition: attachment; filename='.basename($path));
        header('Content-Length: '.filesize($path), true);
        header('x-MD5: '.md5_file($path), true);
        readfile($path);
    }

    if(!check_header('HTTP_USER_AGENT', 'ESP8266-http-Update')) {
        header($_SERVER["SERVER_PROTOCOL"].' 403 Forbidden', true, 403);
        echo "only for ESP8266 updater!\n";
        exit();
    }

    if(
        !check_header('HTTP_X_ESP8266_STA_MAC') ||
        !check_header('HTTP_X_ESP8266_AP_MAC') ||
        !check_header('HTTP_X_ESP8266_FREE_SPACE') ||
        !check_header('HTTP_X_ESP8266_SKETCH_SIZE') ||
        !check_header('HTTP_X_ESP8266_SKETCH_MD5') ||
        !check_header('HTTP_X_ESP8266_CHIP_SIZE') ||
        !check_header('HTTP_X_ESP8266_SDK_VERSION')
    ) {
        header($_SERVER["SERVER_PROTOCOL"].' 403 Forbidden', true, 403);
        echo "only for ESP8266 updater! (header)\n";
        exit();
    }

    $db = array(
        "18:FE:AA:AA:AA:AA" => "DOOR-7-g14f53a19",
        "18:FE:AA:AA:AA:BB" => "TEMP-1.0.0"
    );

    if(!isset($db[$_SERVER['HTTP_X_ESP8266_STA_MAC']])) {
        header($_SERVER["SERVER_PROTOCOL"].' 500 ESP MAC not configured for updates', true, 500);
    }

    $localBinary = "./bin/".$db[$_SERVER['HTTP_X_ESP8266_STA_MAC']].".bin";

    // Check if version has been set and does not match, if not, check if
    // MD5 hash between local binary and ESP8266 binary do not match if not.
    // then no update has been found.
    if((!check_header('HTTP_X_ESP8266_SDK_VERSION') && $db[$_SERVER['HTTP_X_ESP8266_STA_MAC']] != $_SERVER['HTTP_X_ESP8266_VERSION'])
        || $_SERVER["HTTP_X_ESP8266_SKETCH_MD5"] != md5_file($localBinary)) {
        sendFile($localBinary);
    } else {
        header($_SERVER["SERVER_PROTOCOL"].' 304 Not Modified', true, 304);
    }

    header($_SERVER["SERVER_PROTOCOL"].' 500 no version for ESP MAC', true, 500);

Stream Interface
----------------

TODO describe Stream Interface

The Stream Interface is the base for all other update modes like OTA, http Server / client.

Updater class
-------------

Updater is in the Core and deals with writing the firmware to the flash, checking its integrity and telling the bootloader to load the new firmware on the next boot.

Proceso de actualización - Vista de la memoria
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  The new sketch will be stored in the space between the old sketch and
   the spiff.
-  on the next reboot the "eboot" bootloader check for commands.
-  the new sketch is now copied "over" the old one.
-  the new sketch is started.

.. figure:: update_memory_copy.png
   :alt: Memory layout for OTA updates

.. |ota sketch selection| image:: a-ota-sketch-selection.png
.. |ota ssid pass entry| image:: a-ota-ssid-pass-entry.png
.. |ota serial upload config| image:: a-ota-serial-upload-configuration.png
