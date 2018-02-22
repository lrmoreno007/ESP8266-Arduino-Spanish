Obtengo el error "espcomm_sync failed" cuando intento subir a mi ESP. ¿Como resuelvo este problema?
--------------------------------------------------------------------------------------------------

-  `Introducción <#introducción>`__
-  `Comprobaciones iniciales <#comprobaciones-iniciales>`__
-  `Comprobaciones avanzadas <#comprobaciones-avanzadas>`__
-  `Métodos de Reset <#métodos-de-reset>`__
-  `Ck <#ck>`__
-  `NodeMCU <#nodemcu>`__
-  `Estoy atascado <#estoy-atascado>`__
-  `Conclusión <#conclusión>`__

Introducción
~~~~~~~~~~~~

Este mensaje indica un problema al subir al módulo ESP mediante conexión por puerto serie. Existen varias posibles causas que dependen del tipo de módulo y de si tiene un convertidor serie independiente, que parámetros ha seleccionado para subir, etc. Como resultado no hay una solo respuesta a la causa raíz. Para solucionarlo necesitas completar una serie de pasos.

    Nota: Si estás comenzando con ESP, para reducir los problemas con la subida, selecciona una tarjeta ESP con convertidor USB a Serie integrado. Esto reducirá el número de factores dependientes del usuario o opciones de configuración que influyen en el proceso de subida.

Ejemplos de tarjetas con convertidor USB a Serie integrado, que harán que el desarrollo de tus primeros proyectos sea mas sencillo:

.. figure:: pictures/a01-example-boards-with-usb.png
   :alt: Ejemplos de tarjetas con convertidor USB a Serie integrado

   Ejemplos de tarjetas con convertidor USB a Serie integrado

Si estás utilizando un módulo Genérico ESP8266, con convertidor USB a Serie separado y conectado por ti mismo, asegúrate de que haces bien lo siguiente:
1. Se suministra al módulo suficiente energía
2. GPIO0, GPIO15 y CH_PD están conectados utilizando resistencias pull up / pull down. 
3. El módulo entra en modo bootloader.

Para detalles específicos consulta la sección [Modulo Generico ESP8266](../boards.md)


`Módulo Genérico ESP8266 <../boards.rst#generic-esp8266-modules>`__. Ejemplos de módulos sin convertidor USB a Serie en la tarjeta:

.. figure:: pictures/a01-example-boards-without-usb.png
   :alt: Ejemplos de módulos sin convertidor USB a Serie

   Ejemplos de módulos sin convertidor USB a Serie

Comprobaciones iniciales
~~~~~~~~~~~~~~

Para solucionar el error "espcomm_sync failed", por favor proceda paso por paso a través de la siguiente lista de tareas. Esta lista está organizada comenzando por lo mas común y simple a lo mas complejo.

1. Comienza leyendo el mensaje mostrado en la ventana de debug del IDE Arduino. En muchos casos provee información directa de donde está el problema.

.. figure:: pictures/a01-espcomm_open-failed.png
   :alt: error "espcomm_open failed"

   error "espcomm_open failed"

Por ejemplo, el mensaje anterior sugiere que Arduino IDE no puede abrir un puerto serie COM3. Compruebe si ha seleccionado el puerto al que está conectado su módulo.

.. figure:: pictures/a01-serial-port-selection.png
   :alt: Selección de puerto Serie

   Selección de puerto Serie

2. Si el módulo está conectado al puerto Serie pero no responde como un dispositivo válido, el mensaje será ligeramente diferente (ver debajo). Si tienes otros módulos conectados a tu PC, asegúrate de que estás subiendo el programa al ESP8266 y no p. ej. a un Arduino Uno.

.. figure:: pictures/a01-espcomm_sync-failed.png
   :alt: error "espcomm_sync failed"

   error "espcomm_sync failed"

3. Para hacer que tu PC se comunique con el ESP, selecciona el tipo de ESP exacto en el menú de tarjetas.
   Si la selección es incorrecta la subida puede fallar.

.. figure:: pictures/a01-board-selection.png
   :alt: Selección de tarjeta

   Selección de tarjeta

Basado en la tarjeta seleccionada, el IDE Arduino aplicará un "método de reset" específico para ponerse en modo bootloader. El método de reset es específico de cada tarjeta. Algunas tarjetas no tienen hardware integrado que soporte el reset desde el IDE Arduino. Si este es el caso, necesitas poner tu tarjeta en modo bootloader manualmente.

4. La subida también puede fallar debido a una alta velocidad. Si tienes un cable USB muy largo o de baja calidad, prueba a reducir la selección en *Upload Speed*.

.. figure:: pictures/a01-serial-speed-selection.png
   :alt: Selección de velocidad Serie

   Selección de velocidad Serie

Comprobaciones avanzadas
~~~~~~~~~~~~~~~

1. Si todavía sufres problemas, comprueba que el módulo está entrando en modo bootloader. Puedes comprobarlo conectando un convertidor USB a Serie secundario y comprobando el mensaje mostrado. Conecta RX y GND del convertidor a TX y GND del ESP como se muestra en el ejemplo siguiente (`obtener fzz fuente <pictures/a01-secondary-serial-hookup.fzz>`__).

.. figure:: pictures/a01-secondary-serial-hookup.png
   :alt: Conexión de convertidor USB a serie secundario

   Conexión de convertidor USB a serie secundario

Abre un terminal a 74880 baudios y observa que mensaje obtienes cuando el ESP es reseteado para programarse. El mensaje correcto es este:

``ets Jan  8 2013,rst cause:2, boot mode:(1,7)``

Si obtienes un mensaje similar pero diferentes valores, decodifícalo utilizando `Mensajes de arranque y modos <../boards.rst#boot-messages-and-modes>`__. Lo importante de la información está contenida en el primer dígito y tres bits más a la derecha del mensaje "boot mode" como se muestra a continuación.

.. figure:: pictures/a01-boot-mode-decoding.png
   :alt: Decodificando el mensaje del modo de arranque

   Decodificando el mensaje del modo de arranque

Por ejemplo el mensaje ``boot mode (3,3)`` indica que los pines GPIO2 y GPIO0 están establecidos HIGH y GPIO15 está establecido como LOW. Esta es la configuración para `operación normal <../boards.rst#minimal-hardware-setup-for-running-only>`__ del módulo (para ejecutar la aplicación de la flash), no para entrar en modo `bootloader <../boards.rst#minimal-hardware-setup-for-bootloading-only>`__ (programación de la flash).

    Nota: Si no haces este paso correctamente no podrás subir nada a tu módulo mediante el puerto serie.

2. Una vez que has confirmado que el módulo está en modo bootloader, pero todavía falla la subida. Si estás utilizando un convertidor externo de USB a serie, entonces verifica si funciona correctamente al ponerlo en bucle. Esto es bastante simple. Simplemente conecta TX y RX de tu convertidor como en la imagen de abajo. Luego abre el Monitor Serie y escribe algunos caracteres. Si todo está bien, entonces deberías ver lo que escribes inmediatamente impreso en el monitor. Para un ESP con USB a convertidor en serie a bordo, este control puede implicar la ruptura de algunas pistas de la PCB. No lo haría a menos que estuviera desesperado. En su lugar, prueba los pasos a continuación.

.. figure:: pictures/a01-usb-to-serial-loop-back.png
   :alt: Bucle en el convertidor USB a serie

   Bucle en el convertidor USB a serie

3. El siguiente paso a probar si no se ha hecho ya, es comprobar los mensajes de debug detallados. Ve a *Archivo > Preferencias*, activa *Mostrar salida detallada mientas: Subir* y prueba a subir el programa otra vez. El mensaje para una subida correcta debe parecerse al ejemplo siguiente:

``C:\Users\Krzysztof\AppData\Local\Arduino15\packages\esp8266\tools\esptool\0.4.8/esptool.exe -vv -cd ck -cb 115200 -cp COM3 -ca 0x00000 -cf C:\Users\KRZYSZ~1\AppData\Local\Temp\build7e44b372385012e74d64fb272d24b802.tmp/Blink.ino.bin    esptool v0.4.8 - (c) 2014 Ch. Klippel <ck@atelier-klippel.de>       setting board to ck       setting baudrate from 115200 to 115200       setting port from COM1 to COM3       setting address from 0x00000000 to 0x00000000       espcomm_upload_file       espcomm_upload_mem       setting serial port timeouts to 1000 ms   opening bootloader   resetting board   trying to connect       flush start       setting serial port timeouts to 1 ms       setting serial port timeouts to 1000 ms       flush complete       espcomm_send_command: sending command header       espcomm_send_command: sending command payload       read 0, requested 1   trying to connect       flush start       setting serial port timeouts to 1 ms       setting serial port timeouts to 1000 ms       flush complete       espcomm_send_command: sending command header       espcomm_send_command: sending command payload       espcomm_send_command: receiving 2 bytes of data       espcomm_send_command: receiving 2 bytes of data       espcomm_send_command: receiving 2 bytes of data       espcomm_send_command: receiving 2 bytes of data       espcomm_send_command: receiving 2 bytes of data       espcomm_send_command: receiving 2 bytes of data       espcomm_send_command: receiving 2 bytes of data       espcomm_send_command: receiving 2 bytes of data   Uploading 226368 bytes from to flash at 0x00000000       erasing flash       size: 037440 address: 000000       first_sector_index: 0       total_sector_count: 56       head_sector_count: 16       adjusted_sector_count: 40       erase_size: 028000       espcomm_send_command: sending command header       espcomm_send_command: sending command payload       setting serial port timeouts to 15000 ms       setting serial port timeouts to 1000 ms       espcomm_send_command: receiving 2 bytes of data       writing flash   ..............................................................................................................................................................................................................................   starting app without reboot       espcomm_send_command: sending command header       espcomm_send_command: sending command payload       espcomm_send_command: receiving 2 bytes of data   closing bootloader       flush start       setting serial port timeouts to 1 ms       setting serial port timeouts to 1000 ms       flush complete``

El registro de subida puede ser muy largo dependiendo del número de intentos hechos por esptool. Analízalo en busca de anomalías de configuración que tengas seleccionado en el IDE Arduino, como diferente puerto serie, método de reset, upload speed, etc. Resuelve las diferencias encontradas.

Métodos de Reset
~~~~~~~~~~~~~

Si has llegado a este punto y todavía obtienes ``espcomm_sync failed``, entonces es el momento de sacar la artillería pesada.

Conecta un osciloscopio o un analizador lógico a los pines GPIO0, RST y RXD del ESP para comprobar que está pasando.

Compara tus medidas con las formas de ondas recogidas en los siguientes circuitos. Están documentados dos métodos estandar de reset para la subida al ESP8266, que puedes seleccionar en el IDE Arduino `ck <#ck>`__ y `nodemcu <#nodemcu>`__.

Ck
^^

El siguiente circuito ha sido preparado para coleccionar ondas del método de reset ck (`get fzz source <pictures/a01-circuit-ck-reset.fzz>`__). Es mas simple que el método de reset `nodemcu <#nodemcu>`__ y por lo tanto, a menudo se utiliza para conectar los módulos Genéricos ESP en una placa de prueba. Compruébalo las medidas contra tu cableado con las formas de onda a continuación.

.. figure:: pictures/a01-circuit-ck-reset.png
   :alt: Ejemplo de circuito para comprobar método ck

   Ejemplo de circuito para comprobar método ck

Las siguientes formas de ondas muestran señales de voltaje en los pines GPIO0 y RST de la tarjeta ESP cuando se sube un firmware.

Observa la secuencia de señal del método reset ck al comienzo de la subida a continuación.

.. figure:: pictures/a01-reset-ck-closeup.png
   :alt: Método de reset: ck, observa el comienzo de la subida

   Método de reset: ck, observa el comienzo de la subida

La siguiente imagen muestra una subida completa del ejemplo `Blink.ino <https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/Blink/Blink.ino>`__ a 921600 baudios. Esta es una velocidad bastante alta, por lo que la carga solo tarda unos 8 segundos.

.. figure:: pictures/a01-reset-ck-complete.png
   :alt: Método reset: ck, subida completa

   Método reset: ck, subida completa

Observa que esptool no es capaz de inicializar la subida al primer intento, entonces reintenta el procedimiento de reset. El caso de un solo intento se muestra como forma de onda a continuación.

.. figure:: pictures/a01-reset-ck-complete-1-retry.png
   :alt: Método reset: ck, subida completa

   Método reset: ck, subida completa

Cada intento se muestra en la ventana de debug de la siguiente manera:

::

    resetting board
    trying to connect
        flush start
        setting serial port timeouts to 1 ms
        setting serial port timeouts to 1000 ms
        flush complete
        espcomm_send_command: sending command header
        espcomm_send_command: sending command payload
        read 0, requested 1

El circuito ck tiene una limitación importante cuando se trata de trabajar con el IDE Arduino. Después de abrir el Monitor Serie (Ctrl-Shift-M), tanto las líneas RTS como las líneas DTR se reducen (pulled down) permanentemente. Como la línea RTS está conectada a la entrada RST de ESP, el módulo se mantiene en estado de reinicio / no se puede ejecutar. Por lo tanto, después de cargar el módulo, debes desconectar ambas líneas o utilizar un programa de terminal en serie diferente que no esté tirando de las líneas RTS y DTR. De lo contrario, el módulo se atascará esperando a que se libere la señal RST y no verá nada en el monitor serie.

Puedes probar el add-on para el IDE Arduino `Serial Monitor for ESP8266 <https://github.com/esp8266/Arduino/issues/1360>`__ desarrollado por el usuario [@mytrain](https://github.com/mytrain) y discutido en `#1360 <https://github.com/esp8266/Arduino/issues/1360>`__.

Si prefieres un programa de terminal externo, entonces para usuarios Windows recomendamos la herramienta libre y práctica: `Termite <http://www.compuphase.com/software_termite.htm>`__.

NodeMCU
^^^^^^^

El método de reset llamado NodeMCU por la tarjeta `NodeMCU <https://github.com/nodemcu/nodemcu-devkit>`__ la cual lo introdujo por primera vez. Supera las limitaciones con el manejo de las líneas RTS y DTR discutidas anteriormente para el método de reset `ck <#ck>`__.

A continuación se muestra un ejemplo de circuito para medir la forma de onda (`get fzz source <pictures/a01-circuit-nodemcu-reset.fzz>`__).

.. figure:: pictures/a01-circuit-nodemcu-reset.png
   :alt: Circuito de ejemplo para comprobar el método de reset nodemcu

   Circuito de ejemplo para comprobar el método de reset nodemcu

Observa las señales de voltaje en los pines GPIO0 y RST al comienzo de la subida del firmware a continuación.

.. figure:: pictures/a01-reset-nodemcu-closeup.png
   :alt: Metodo de reset: nodemcu, observa al comienzo de la subida

   Método de reset: nodemcu, observa al comienzo de la subida

Observa que la secuencia de reset es mas o menos unas 10 veces mas corta comparada con el método de reset `ck <#ck>`__ (sobre 25ms contra 250ms).

La siguiente imagen muestra una subida completa del ejemplo `Blink.ino <https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/Blink/Blink.ino>`__ a 921600 baudios. Salvo la diferencia de la secuencia de la señal de reset, la subida completa es similar a `ck <#ck>`__.

.. figure:: pictures/a01-reset-nodemcu-complete.png
   :alt: Método de reset: nodemcu, subida completa

   Método de reset: nodemcu, subida completa

A continuación se muestra la forma de onda para otra subida del ejemplo `Blink.ino <https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/Blink/Blink.ino>`__ a 921600 baudios, pero con dos reintentos de reset.

.. figure:: pictures/a01-reset-nodemcu-complete-2-retries.png
   :alt: Método de reset: nodemcu, reintentos de reset

   Método de reset: nodemcu, reintentos de reset

Si estás interesado en como está implementado el método de reset nodemcu, comprueba el circuito a continuación. Como se dijo este circuito no une a GND las líneas RTS y DTR una vez que abre el Monitor Serie en el IDE Arduino.

.. figure:: pictures/a01-nodemcu-reset-implementation.png
   :alt: Implementación del reset nodemcu

   Implementación del reset nodemcu

Se compone de dos transistores y resistencias que puede encontrar a la derecha en la placa NodeMCU. A la izquierda, puede ver el circuito completo y la tabla de la verdad con cómo las señales RTS y DTR dela interfaz serie se traducen a RST y GPIO0 en el ESP. Para obtener más información, consulte el repositorio `nodemcu <https://github.com/nodemcu/nodemcu-devkit>`__ en GitHub.

Estoy atascado
~~~~~~~~~
 
Es de esperar que en este punto hayas podido resolver el problema ``espcomm_sync failed`` y ahora disfrutes de cargas rápidas y confiables de tu módulo ESP.

Si aún no lo resolvistes, revisa una vez más todos los pasos discutidos en la siguiente lista de verificación.

**Comprobaciones iniciales**

* [ ] ¿Está tu módulo conectado al puerto serie y visible en el IDE?

* [ ] ¿El dispositivo conectado está respondiendo al IDE? ¿Cuál es el mensaje exacto en la ventana de depuración?

* [ ] ¿Has seleccionado el tipo correcto de módulo ESP en el menú *Placa*? ¿Cuál es la selección?

* [ ] ¿Has intentado reducir la velocidad de carga? ¿Qué velocidades has probado?

**Comprobaciones avanzadas**

* [ ] ¿Qué mensaje informa ESP a 74880 baudios al entrar en modo bootloader?

* [ ] ¿Has comprobado tu convertidor de USB a serie haciendo un bucle? ¿Cual es el resultado?

* [ ] ¿Tu registro de subida detallado es consistente con la configuración en el IDE? ¿Cuál es el registro?

**Método de reset**

* [ ] ¿Qué método de reset utilizas?

* [ ] ¿Cuál es tu circuito de conexión? ¿Coincide con alguno de los circuitos descritos?

* [ ] ¿Cuál es tu forma de onda de reset de la placa? ¿Concuerda con la forma de onda descrito?

* [ ] ¿Cuál es tu forma de onda de subida completa? ¿Concuerda con la forma de onda descrita?

**Software**

* [ ] ¿Utiliza la última versión estable de `ESP8266/Arduino <https://github.com/esp8266/Arduino>`__? ¿Cual es?

* [ ] ¿Cuál es el nombre y la versión de su IDE y O/S?

Si está atascado en cierto paso, publique esta lista rellena en el `foro de la comunidad ESP8266 <http://www.esp8266.com/>`__ pidiendo ayuda.

Conclusión
~~~~~~~~~~

Con la variedad de módulos y placas ESP8266 disponibles, así como posibles métodos de conexión, la resolución de problemas de subida puede tomar varios pasos.

Si eres un principiante, entonces utiliza tarjetas con fuente de alimentación y convertidor USB a serial integrados. Verifica cuidadosamente el mensaje en la ventana de debug y actúa en consecuencia. Selecciona tu tipo de tarjeta exacto en el IDE e intenta ajustar la velocidad de subida. Verifica si la placa está ingresando al modo bootloader. Verifica el funcionamiento de tu convertidor de USB a serie con un bucle. Analiza el registro de subida detallado en busca de inconsistencias con la configuración del IDE.

Verifica que tu diagrama de conexión y forma de onda tenga coherencia con el método de reinicio seleccionado.

Si se queda atascado, pregunte en la `comunidad <http://www.esp8266.com/>`__ con un resumen de todas las verificaciones completadas.

---------------

.. figure:: pictures/a01-test-stand.jpg
   :alt: Banco de pruebas realizado durante la comprobación del método de reinicio ck

   Banco de pruebas realizado durante la comprobación del método de reinicio ck

Ningún módulo ESP ha sido dañado durante la preparación de esta FAQ.

`FAQ :back: <readme.rst>`__
