¿Como solucionar el error "Board generic (platform esp8266, package esp8266) is unknown"?
------------------------------------------------------------------------------------

Este error puede aparecer al cambiar entre los paquetes de instalación de ESP8266/Arduino `staging <https://github.com/esp8266/Arduino#staging-version->`__ y `stable <https://github.com/esp8266/Arduino#stable-version->`__, o tras actualizar de versión de paquete.

.. figure:: pictures/a04-board-is-unknown-error.png
   :alt: Board nodemcu2 (platform esp8266, package esp8266) is unknown error

   Board nodemcu2 (platform esp8266, package esp8266) is unknown error

Si te enfrentas a este problema, no podrás compilar ningún boceto para ningún tipo de módulo ESP8266.

Lee a continuación cuál es la causa raíz del error o salte directamente a la `solución <#cómo-arreglarlo>`__

La causa raiz
~~~~~~~~~~~~~~

Este problema se atribuye al Gestor de Tarjetas del IDE Arduino, no limpiando la instalación previa del paquete antes de aplicar uno nuevo. Como esto no se hace, es responsabilidad del usuario eliminar el paquete anterior antes de aplicar uno nuevo.

Para evitar que suceda, si está cambiando entre **staging** y **stable**, primero presione en el Gestor de Tarjetas el botón *Eliminar* para eliminar la instalación actualmente utilizada.

.. figure:: pictures/a04-remove-package-yes.png
   :alt: Si cambia entre staging y stable, elimine el paquete instalado actualmente

   Si cambia entre staging y stable, elimine el paquete instalado actualmente

No es necesario eliminar el paquete instalado si lo está cambiando a otra versión (sin cambiar entre staging y stable).

.. figure:: pictures/a04-remove-package-no.png
   :alt: No es necesario eliminar el paquete instalado si cambia entre versiones (actualiza)

   No es necesario eliminar el paquete instalado si cambia entre versiones (actualiza)

Dependiendo del módulo seleccionado, el mensaje de error es ligeramente diferente. Por ejemplo, si elige *Generic ESP8266 Module*, se verá de la siguiente manera:

::

    Board generic (platform esp8266, package esp8266) is unknown
    Error compiling for board Generic ESP8266 Module.

A continuación un mensaje de ejemplo para `WeMos <../boards.rst#wemos-d1-r2-mini>`__:

::

    Board d1_mini (platform esp8266, package esp8266) is unknown
    Error compiling for board WeMos D1 R2 & mini.

... y otro para `Adafruit Feather HUZZAH <../boards.rst#adafruit-feather-huzzah-esp8266>`__:

::

    Board huzzah (platform esp8266, package esp8266) is unknown
    Error compiling for board Adafruit HUZZAH ESP8266.

Si el problema ya ocurre, desinstalar y volver a instalar el paquete con *Boards Manager* normalmente no lo arreglará.

Desinstalar y reinstalar el IDE de Arduino no lo arreglará tampoco.

Bien, bien, bien. Podrás arreglarlo con Boards Manager. Para hacerlo, debe ir paso a paso con cuidado eliminar el paquete nuevo y luego el anterior. Una vez hecho esto, puede instalar nuevamente el nuevo paquete. ¿Mencioné que entre tanto necesita cambiar dos veces `JSON <https://github.com/esp8266/Arduino#installing-with-boards-manager>`__ en *Gestor de URLs Adicionales de Tarjetas*?

Afortunadamente hay una solución más rápida y efectiva. Vea abajo.

Cómo arreglarlo
~~~~~~~~~~~~~~

La resolución de problemas es tan simple como eliminar la carpeta con la instalación anterior de esp8266/Arduino.

El procedimiento es idéntico en Windows, Linux y Mac OS. La única diferencia es la ruta de la carpeta. Por ejemplo, en Mac, será ``/Users/$USER/Library/Arduino15/packages/esp8266/hardware/esp8266``. El siguiente ejemplo muestra la ruta para Windows.

1. Comprueba la ubicación de la carpeta de instalación yendo a *Archivo > Preferencias* (Ctrl+,). La ubicación de la carpeta se encuentra en la parte inferior de la ventana *Preferencias*.

.. figure:: pictures/a04-arduino-ide-preferences.png
   :alt: Comprueba las preferencias del IDE Arduino

   Comprueba las preferencias del IDE Arduino

2. Haz clic en el enlace para abrir la carpeta. Para Windows 7 se verá de la siguiente manera:

.. figure:: pictures/a04-contents-of-preferences-folder.png
   :alt: Contenido de la carpeta de preferencias del IDE Arduino

   Contenido de la carpeta de preferencias del IDE Arduino

3. Ve hasta el directorio ``Arduino15\packages\esp8266\hardware\esp8266``. En el interior, encontrarás dos carpetas con diferentes instalaciones del paquete ESP8266/Arduino.

.. figure:: pictures/a04-contents-of-package-folder.png
   :alt: Comprueba el contenido de la carpeta del paquete ESP8266/Arduino

   Comprueba el contenido de la carpeta del paquete ESP8266/Arduino

4. Elimina la carpeta anterior. Reinicia Arduino IDE, selecciona tu módulo ESP y el error desaparecerá.

Nota: Si no estás seguro de qué carpeta eliminar, elimina ambas. Reinicia Arduino IDE, ve a *Herramientas> "última placa seleccionada" > Gestor de Tarjetas* e instala el paquete ESP8266/Arduino nuevamente. Selecciona el módulo ESP8266 deseado y el problema debería resolverse.

Mas información
~~~~~~~~~~~~~~~~

Este problema ha sido informado con bastante frecuencia en la sección de `Issues <https://github.com/esp8266/Arduino/issues>`__ del repositorio ESP8266/Arduino. La solución más apreciada fue proporcionada por [@anhhuy0501](https://github.com/anhhuy0501) en `#1387 <https://github.com/esp8266/Arduino/issues/1387#issuecomment-204865028>`__.

Si estás interesado en más detalles, consulta `#2297 <https://github.com/esp8266/Arduino/issues/2297>`__, `#2156 <https://github.com/esp8266/Arduino/issues/2156>`__, `#2022 <https://github.com/esp8266/Arduino/issues/2022>`__, `#1802 <https://github.com/esp8266/Arduino/issues/1802>`__, `#1514 <https://github.com/esp8266/Arduino/issues/1514>`__, `#1387 <https://github.com/esp8266/Arduino/issues/1387>`__, `#1377 <https://github.com/esp8266/Arduino/issues/1377>`__, `#1251 <https://github.com/esp8266/Arduino/issues/1251>`__, `#1247 <https://github.com/esp8266/Arduino/issues/1247>`__, `#948 <https://github.com/esp8266/Arduino/issues/948>`__
