Actualizaciones OTA
===========


Introducción
------------

La actualización OTA (Over the Air - Por el aire) es el proceso de carga del firmware en el módulo ESP mediante una conexión Wi-Fi en lugar de un puerto serie. Dicha funcionalidad se vuelve extremadamente útil en caso de acceso físico limitado o nulo al módulo.

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

1. Antes de continuar, asegurese de tener el siguiente software instalado:

   -  Arduino IDE 1.6.7 o posterior -
      https://www.arduino.cc/en/Main/Software
   -  Paquete de la plataforma esp8266/Arduino 2.0.0 o posterior - para instrucciones vaya a:
      https://github.com/lrmoreno007/ESP8266-Arduino-Spanish#instalando-mediante-el-gestor-de-tarjetas
   -  Python 2.7 - https://www.python.org/

      **Nota:** Los usuarios de Windows deben seleccionar "Agregar python.exe a la ruta" (ver más abajo, esta opción no está seleccionada de manera predeterminada).

      .. figure:: a-ota-python-configuration.png
         :alt: Configuración de la instalación de Python

2. Ahora prepare el sketch y la configuración para la primera subida a través del puerto serie.

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

4. Solo si el módulo está conectado a la red, después de un par de segundos, el puerto esp8266-ota aparecerá en el IDE de Arduino. Seleccione el puerto con la dirección IP que se muestra en la ventana del Monitor serie en el paso anterior:

   .. figure:: a-ota-ota-port-selection.png
      :alt: Selección del puerto OTA

   **Nota:** Si el puerto OTA no se muestra, salga del IDE de Arduino, ábralo nuevamente y verifique si el puerto está allí. Si no funciona, verifique la configuración de su firewall y del router. El puerto OTA se anuncia mediante el servicio mDNS. Para verificar si su PC puede ver el puerto, puede usar una aplicación como Bonjour Browser.

5. Ahora prepárate para tu primera carga OTA seleccionando el puerto OTA:

   .. figure:: a-ota-ota-upload-configuration.png
      :alt: Configuración para la subida OTA

   **Nota:** La entrada del menú ``Upload Speed:`` no importa en este punto ya que se refiere al puerto serie. Dejelo sin cambiar.

6. Si ha completado con éxito todos los pasos anteriores, puede cargar (Ctrl+U) el mismo (o cualquier otro) sketch sobre OTA:

   .. figure:: a-ota-ota-upload-complete.png
      :alt: Subida OTA completa

**Nota:** Para poder cargar su sketch una y otra vez utilizando OTA, debe insertar rutinas OTA en su interior. Por favor use BasicOTA.ino como ejemplo.

Protección con contraseña
^^^^^^^^^^^^^^^^^^^

Proteger sus cargas OTA con contraseña es realmente sencillo. Todo lo que necesita hacer es incluir la siguiente declaración en su código:

.. code:: cpp

    ArduinoOTA.setPassword((const char *)"123");

Donde ``123`` es una contraseña de ejemplo que debe reemplazar con la suya.

Antes de implementarlo en su sketch, es una buena idea verificar cómo funciona el sketch *BasicOTA.ino* disponible en *Archivo > Ejemplos > ArduinoOTA*. Adelante, abra *BasicOTA.ino*, descomente la declaración anterior y cargue el sketch. Para facilitar la resolución de problemas, no modifique el boceto de ejemplo, solo lo absolutamente necesario. Se incluye una contraseña OTA ``123`` simple y original. A continuación, intente cargar el sketch de nuevo (utilizando OTA). Una vez finalizada la compilación, una vez que la carga está a punto de comenzar, debería ver la solicitud de contraseña de la siguiente manera:

.. figure:: a-ota-upload-password-prompt.png
   :alt: Aviso de aontraseña para la subida OTA

Ingrese la contraseña y la carga debe iniciarse como de costumbre, con la única diferencia del mensaje ``Autentificando ... OK`` visible en el registro de subida.

.. figure:: a-ota-upload-password-authenticating-ok.png
   :alt: Authenticating...OK duante la subida OTA

No se le solicitará que vuelva a ingresar la misma contraseña la próxima vez. Arduino IDE lo recordará por ti. Verá una solicitud de contraseña solo después de volver a abrir el IDE o si la cambia en su sketch, cargue el sketch y luego intente cargarlo nuevamente.

Tenga en cuenta que es posible revelar la contraseña ingresada previamente en el IDE de Arduino, si el IDE no se ha cerrado desde la última carga. Esto se puede hacer habilitando *Mostrar salida detallada mientras: Subir* en *Archivo > Preferencias* y intentando subir el modulo.

.. figure:: a-ota-upload-password-passing-upload-ok.png
   :alt: Salida de subida detallada con contraseña en modo texto

La imagen de arriba muestra que la contraseña es visible en el registro, ya que se pasa al script de subida *espota.py*.

Otro ejemplo a continuación muestra la situación cuando la contraseña se cambia entre subidas.

.. figure:: a-ota-upload-password-passing-again-upload-ok.png
   :alt: Salida detallada cuando se cambia la contraseña OTA entre subidas

En la imagen puede verse que al subir, el IDE de Arduino utilizó la contraseña ingresada previamente, por lo que la carga falló y eso fue claramente reportado por el IDE. Solo entonces el IDE solicitó una nueva contraseña. Se ingresó correctamente y el segundo intento de carga fue exitoso.

Solución de problemas
^^^^^^^^^^^^^^^

Si la actualización de OTA falla, el primer paso es verificar los mensajes de error que pueden aparecer en la ventana de carga del IDE de Arduino. Si esto no proporciona sugerencias útiles, intente cargar de nuevo mientras verifica lo que muestra ESP en el puerto serie. Serial Monitor de IDE no será útil en ese caso. Al intentar abrirlo, es probable que veas lo siguiente:

.. figure:: a-ota-network-terminal.png
   :alt: Arduino IDE network terminal window

Esta ventana es para Arduino Yún y aún no está implementada para esp8266/Arduino. Aparece porque el IDE está intentando abrir Serial Monitor utilizando el puerto de red que ha seleccionado para la carga OTA.

En su lugar, necesita un monitor serie externo. Si es un usuario de Windows, consulte `Termite <http://www.compuphase.com/software_termite.htm>`__. Este es un terminal RS232 práctico, elegante y simple que no impone el control de flujo RTS o DTR. Dicho control de flujo puede causar problemas si está utilizando líneas respectivas para alternar los pines GPIO0 y RESET en ESP para la carga.

Seleccione el puerto COM y la velocidad en baudios en el programa terminal externo como si estuviera usando Arduino Serial Monitor. Consulte la configuración típica de `Termite <http://www.compuphase.com/software_termite.htm>`__ a continuación:

.. figure:: termite-configuration.png
   :alt: Configuración Termite

Luego ejecute OTA desde el IDE y observe lo que se muestra en el terminal. El proceso `ArduinoOTA <#arduinoota>`__ exitoso usando el sketch BasicOTA.ino se ve a continuación (la dirección IP depende de la configuración de su red):

.. figure:: a-ota-external-serial-terminal-output.png
   :alt: Subida OTA satisfactoria - Salida en un Terminal Serie externo

Si la carga falla, es probable que vea los errores detectados por el cargador, la excepción y el seguimiento de la pila, o ambos.

En lugar del registro como en la pantalla anterior, puede ver lo siguiente:

.. figure:: a-ota-external-serial-terminal-output-failed.png
   :alt: Subida OTA fallida - Salida en un Terminal Serie externo

Si este es el caso, lo más probable es que el módulo ESP no se haya reiniciado después de la primera subida utilizando el puerto serie.

Las causas más comunes de fallo OTA son las siguientes:

- no hay suficiente memoria física en el chip (por ejemplo, ESP01 con 512K de memoria flash no es suficiente para OTA).
- demasiada memoria declarada para SPIFFS, por lo que el nuevo sketch no se ajustará entre el boceto existente y SPIFFS - vea `Proceso de actualización - vista de la memoria <#proceso-de-actualizacion-vista-de-la-memoria>`__.
- muy poca memoria declarada en Arduino IDE para su placa seleccionada (es decir, menor que el tamaño físico).
- no reiniciar el módulo ESP después de la primera subida utilizando el puerto serie.

Para obtener más información sobre el diseño de la memoria flash, consulte `Sistema de ficheros <../filesystem.rst>`__. Para obtener información general sobre dónde se almacena el nuevo boceto, cómo se copia y cómo se organiza la memoria para el propósito de OTA, consulte `Proceso de actualización - vista de la memoria <#proceso-de-actualizacion-vista-de-la-memoria>`__.

Buscador Web
-----------

Las actualizaciones descritas en este capítulo se realizan con un navegador web que puede ser útil en los siguientes escenarios típicos:

- después de la implementación de la aplicación si la carga directa desde Arduino IDE es inconveniente o no es posible.
- después de la implementación si el usuario no puede exponer el módulo para OTA desde un servidor de actualización externo.
- para proporcionar actualizaciones después de la implementación a una pequeña cantidad de módulos al configurar un servidor de actualización no es factible.

Requerimientos
~~~~~~~~~~~~

-  El ESP y el ordenador deben estar conectados a la misma red.

Descripción general de la implementación
~~~~~~~~~~~~~~~~~~~~~~~

Las actualizaciones con un navegador web se implementan utilizando la clase ``ESP8266HTTPUpdateServer`` junto con las clases ``ESP8266WebServer`` y ``ESP8266mDNS``. Se requiere el siguiente código para que funcione:

setup()

.. code:: cpp

        MDNS.begin(host);

        httpUpdater.setup(&httpServer);
        httpServer.begin();

        MDNS.addService("http", "tcp", 80);

loop()

.. code:: cpp

        httpServer.handleClient();

Ejemplo de aplicación
~~~~~~~~~~~~~~~~~~~

La implementación de ejemplo proporcionada a continuación se ha realizado utilizando:

-  Sketch de ejemplo WebUpdater.ino disponible en la librería ``ESP8266HTTPUpdateServer`` library.
-  NodeMCU 1.0 (Módulo ESP-12E).

Puede utilizar otro módulo si cumple con los requisitos descritos anteriormente. `Requerimientos Básicos <#requerimientos-basicos>`__.

1. Before you begin, please make sure that you have the following software installed:

   - Arduino IDE y la versión 2.0.0-rc1 (del 17 de noviembre de 2015) de la plataforma del paquete como se describe en https://github.com/lrmoreno007/ESP8266-Arduino-Spanish#instalando-mediante-el-gestor-de-tarjetas
   -  Software de host en función del sistema operativo que utilice:

      1. Avahi http://avahi.org/ para Linux
      2. Bonjour http://www.apple.com/support/bonjour/ para Windows
      3. Mac OSX y iOS - ya soportado interiormente, no se requiere ningún software extra

2. Prepare el boceto y la configuración para la primera subida mediante puerto serie.

   -  Inicie Arduino IDE y cargue el sketch WebUpdater.ino disponible en Archivo > Ejemplos > ESP8266HTTPUpdateServer.
   -  Actualice su SSID y contraseña en el sketch, para que el módulo pueda unirse a su red Wi-Fi.
   -  Abra Archivo > Preferencias, busque “Mostrar salida detallada mientras:” y active la opción “Compilación”.

      .. figure:: ota-web-show-verbose-compilation.png
         :alt: Preferencias - activando salida detallada durante la compilación

      **Nota:** Esta configuración será necesaria en el paso 5 a continuación. Puedes desmarcar esta configuración después.

3. Suba el sketch (Ctrl+U). Una vez hecho esto, abra el Monitor Serie (Ctrl+Shift+M) y verifique si aparece el siguiente mensaje, que contiene la url para la actualización OTA.

   .. figure:: ota-web-serial-monitor-ready.png
      :alt: Serial Monitor - Tras la subida inicial mediante serial

   **Nota:** Dicho mensaje se mostrará solo después de que el módulo se una con éxito a la red y esté listo para una carga OTA. Recuerde lo hablado acerca de reiniciar el módulo después de la primera subida mediante serial como se explica en el capítulo `Arduino IDE <#arduino-ide>`__, paso 3.

4. Ahora abra el navegador web e ingrese la url proporcionada por el Monitor Serie, es decir, ``http://esp8266-webupdate.local/update``. Una vez ingresado, el navegador debe mostrar un formulario como el que se encuentra en su módulo. El formulario te invita a elegir un archivo para actualizar.

   .. figure:: ota-web-browser-form.png
      :alt: Formulario de actualización OTA en el buscador Web

   **Nota:** Si mediante ``http://esp8266-webupdate.local/update`` no funciona, intente reemplazar ``esp8266-webupdate`` con la dirección IP del módulo. Por ejemplo, si la IP de su módulo es ``192.168.1.100``, entonces la url debería ser ``http://192.168.1.100/update``. Esta solución es útil en caso de que el software host instalado en el paso 1 no funcione. Si todavía nada funciona y no hay pistas en el Monitor Serie, intente diagnosticar el problema abriendo la URL proporcionada en Google Chrome, presionando F12 y verificando el contenido de las pestañas "Consola" y "Red". Chrome proporciona un registro avanzado en estas pestañas.

5. Para obtener el archivo, navegue al directorio utilizado por Arduino IDE para almacenar los resultados de la compilación. Puede verificar la ruta de acceso a este archivo en el registro de compilación que se muestra en la ventana de depuración del IDE como se indica a continuación.

   .. figure:: ota-web-path-to-binary.png
      :alt: Compilación completa - Dirección del fichero binario

6. Ahora presione “Choose File” en el navegador web, vaya al directorio identificado en el paso 5 anterior, busque el archivo "WebUpdater.cpp.bin" y cárguelo. Si la carga se realiza correctamente, verá "OK" en el navegador web como se muestra a continuación.

   .. figure:: ota-web-browser-form-ok.png
      :alt: Actualización OTA completa

   Se reiniciará el módulo que debería estar visible en el Monitor Serie:

   .. figure:: ota-web-serial-monitor-reboot.png
      :alt: Monitor Serie - tras actualización OTA

   Justo después de reiniciar, debería ver exactamente el mismo mensaje ``HTTPUpdateServer ready! Open http://esp8266-webupdate.local/update in your browser`` como en el paso 3. Esto se debe a que el módulo se ha cargado nuevamente con el mismo código: primero utilizando el puerto serie y luego usando OTA.

Una vez que se sienta cómodo con este procedimiento, siga adelante y modifique el boceto de WebUpdater.ino para imprimir algunos mensajes adicionales, compile, localice un nuevo archivo binario y cárguelo utilizando el navegador web para ver los cambios introducidos en el Monitor Serie.

También puede agregar rutinas OTA a su propio sketch siguiendo las pautas en `Descripción general de la implementación <#descripcion-general-de-la-implementacion>`__. Si esto se hace correctamente, siempre debe poder cargar un nuevo boceto sobre el anterior utilizando un navegador web.

En caso de que la actualización de OTA falle después de introducir modificaciones en su boceto, siempre puede recuperar el módulo cargándolo en un puerto serie. Luego, diagnostique el problema con el boceto utilizando el Monitor Serie. Una vez que se solucione el problema intente OTA otra vez.

Servidor HTTP
-----------

La clase ``ESPhttpUpdate`` puede buscar actualizaciones y descargar un archivo binario desde el servidor web HTTP. Es posible descargar actualizaciones de cada dirección IP o de dominio en la red o en Internet.

Requerimientos
~~~~~~~~~~~~

-  Servidor Web

Código Arduino
~~~~~~~~~~~~

Actualizador Sencillo
^^^^^^^^^^^^^^

El actualizador sencillo descarga el archivo cada vez que se llama a la función.

.. code:: cpp

    ESPhttpUpdate.update("192.168.0.2", 80, "/arduino.bin");

Actualizador Avanzado
^^^^^^^^^^^^^^^^

Es posible apuntar la función de actualización a un script en el servidor. Si se proporciona un argumento de cadena de versión, se enviará al servidor. El script del lado del servidor puede usar esto para verificar si se debe realizar una actualización.

El script del lado del servidor puede responder de la siguiente manera: 

- código de respuesta 200, y enviar la imagen del firmware
- o código de respuesta 304 para notificar a ESP que no se requiere ninguna actualización.

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

Manejo de solicitudes del servidor
~~~~~~~~~~~~~~~~~~~~~~~

Actualizador Sencillo
^^^^^^^^^^^^^^

Para el actualizador sencillo, el servidor solo necesita entregar el archivo binario para su actualización.

Advanced updater
^^^^^^^^^^^^^^^^

Para la administración avanzada de actualizaciones, un script debe ejecutarse en el lado del servidor, por ejemplo, un script PHP. En cada solicitud de actualización, el ESP envía alguna información en los encabezados HTTP al servidor.

Ejemplo de datos de encabezado:

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

Con esta información, el script ahora puede verificar si se necesita una actualización. También es posible entregar diferentes binarios basados en la dirección MAC, por ejemplo.

Ejemplo de guión:

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

Interfaz de transmisión
----------------

POR HACER descripción del Interfaz de Transmisión

El Interfaz de Transmisión es la base para todos los demás modos de actualización como OTA, http Server / client.

Clase Updater
-------------

Updater está en el Core y se ocupa de escribir el firmware en la memoria flash, verificar su integridad y decirle al cargador de arranque que cargue el nuevo firmware en el siguiente arranque.

**Nota:** El comando del cargador de arranque se almacenará en los primeros 128 bytes de la memoria RTC del usuario, y luego será recuperado por eboot en el arranque. Eso significa que los datos de usuario presentes allí se perderán `(discusión en #5330) <https://github.com/esp8266/Arduino/pull/5330#issuecomment-437803456>`__.

Proceso de actualización - Vista de la memoria
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- El nuevo sketch se almacenará en el espacio entre el sketch anterior y el spiff.
- En el siguiente reinicio, el gestor de arranque "eboot" verifica los comandos.
- El nuevo sketch ahora se copia "sobre" el anterior.
- Se inicia el nuevo sketch.

.. figure:: update_memory_copy.png
   :alt: Diseño de memoria para actualizaciones OTA

.. |ota sketch selection| image:: a-ota-sketch-selection.png
.. |ota ssid pass entry| image:: a-ota-ssid-pass-entry.png
.. |ota serial upload config| image:: a-ota-serial-upload-configuration.png
