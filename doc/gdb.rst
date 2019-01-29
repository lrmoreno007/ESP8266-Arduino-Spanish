Utilizando GDB para depurar aplicaciones
===============================

Las aplicaciones ESP se pueden depurar usando GDB, el depurador de GNU, que se incluye con la instalación estándar de IDE. Esta nota solo discute los pasos específicos de ESP, así que por favor consulte la `documentación principal de GNU GDB
<//sourceware.org/gdb/download/onlinedocs/gdb/index.html>`__.

Tenga en cuenta que a partir de 2.5.0, la cadena de herramientas se movió desde el parche ESPRESSIF, versión de código cerrado de GDB a la versión principal de GNU.  Los formatos de depuración son diferentes, así que asegúrese de usar solo el último ejecutable GDB del toolchain de Arduino.

Nota CLI e IDE
----------------

Debido a que el IDE de Arduino no admite la depuración interactiva, las siguientes secciones describen la depuración utilizando la línea de comandos. Otros IDE que utilizan GDB en sus backends de depuración deberían funcionar de manera idéntica, pero es posible que deba editar sus opciones de archivos de configuración para habilitar la depuración de serie remota requerida y establecer las opciones estándar. ¡Felizmente, se aceptan PRs para actualizar este documento con IDEs adicionales!

Preparando tu aplicación para GDB
----------------------------------

Las aplicaciones deben cambiarse para habilitar el soporte de depuración GDB. Este cambio agregará 2-3KB de flash y alrededor de 700 bytes de uso de IRAM, pero no debería afectar el funcionamiento de la aplicación.

En su archivo principal ``sketch.ino``, agregue la siguiente línea en la parte superior de la aplicación:

.. code:: cpp

    #include <GDBStub.h>

Y en la función ``void setup()`` asegúrese de que el puerto serie esté inicializado y llame a ``gdbstub_init()``:

.. code:: cpp

    Serial.begin(115200);
    gdbstub_init();

Vuelva a compilar y cargar su aplicación, debería ejecutarse exactamente como antes.

Iniciando la sesión de depuración
------------------------

Una vez que su aplicación se está ejecutando, el proceso para añadir un depurador es bastante simple:

- Cierre el Monitor Serie de Arduino
- Localice el archivo Application.ino.elf
- Abra un Símbolo del sistema e inicie GDB
- Aplicar las configuraciones GDB
- Añadir el depurador
- A depurar!

Cierre el Monitor Serie de Arduino
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Debido a que GDB necesita el control total del puerto serie, deberá cerrar todas las ventanas del Monitor Serie de Arduino que pueda tener abiertas. De lo contrario, GDB informará de un error al intentar depurar.

Localice el archivo Application.ino.elf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Para que GDB pueda depurar su aplicación, necesita localizar la versión compilada en formato ELF (que incluye los símbolos de depuración necesarios). Bajo Linux, estos archivos se almacenan en ``/tmp/arduino_build_*`` y el siguiente comando ayudará a localizar el archivo correcto para su aplicación

.. code:: cpp

    find /tmp -name "*.elf" -print

Tenga en cuenta la ruta completa del archivo ELF que corresponde al nombre de su boceto, se necesitará más adelante una vez que se inicie GDB.

Abra un Símbolo del sistema e inicie GDB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Abra un terminal o Símbolo de sistema y navegue hasta el directorio adecuado de la cadena de herramientas ESP8266.

Linux

.. code:: cpp

    ~/.arduino15/packages/esp8266/hardware/xtensa-lx106-elf/bin/xtensa-lx106-elf-gdb

Windows (Usando versión del Gestor de Tarjetas)

.. code:: cpp

    %userprofile%\AppData\Local\Arduino15\packages\esp8266\tools\xtensa-lx106-elf-gcc\2.5.0-3-20ed2b9\bin\xtensa-lx106-elf-gdb.exe

Windows (Usando versión Git)

.. code:: cpp

    %userprofile%\Documents\Arduino\hardware\esp8266com\esp8266\tools\xtensa-lx106-elf\bin\xtensa-lx106-elf-gdb.exe

Tenga en cuenta que el nombre correcto de GDB es "xtensa-lx106-elf-gdb". Si ejecuta accidentalmente "gdb", puede iniciar el propio GDB de su sistema operativo, que no sabrá cómo hablar con el ESP8266.

Aplicar las configuraciones GDB
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

En el prompt ``(gdb)``, ingrese las siguientes opciones para configurar GDB con el mapa de memoria ESP8266 y la configuración:

.. code:: cpp

    set remote hardware-breakpoint-limit 1
    set remote hardware-watchpoint-limit 1
    set remote interrupt-on-connect on
    set remote kill-packet off
    set remote symbol-lookup-packet off
    set remote verbose-resume-packet off
    mem 0x20000000 0x3fefffff ro cache
    mem 0x3ff00000 0x3fffffff rw
    mem 0x40000000 0x400fffff ro cache
    mem 0x40100000 0x4013ffff rw cache
    mem 0x40140000 0x5fffffff ro cache
    mem 0x60000000 0x60001fff rw
    set serial baud 115200

Ahora dile a GDB dónde se encuentra tu archivo ELF compilado:

.. code:: cpp

    file /tmp/arduino_build_257110/sketch_dec26a.ino.elf

Añadir el depurador
~~~~~~~~~~~~~~~~~~~

Una vez que GDB se haya configurado correctamente y haya cargado sus símbolos de depuración, conéctelo al ESP con el comando (reemplace ttyUSB0 o COM9 con el puerto serie de su ESP):

.. code:: cpp

    target remote /dev/ttyUSB0

o

.. code:: cpp

    target remote \\.\COM9

En este punto, GDB enviará una detención de la aplicación en el ESP8266 y podrá comenzar a configurar un punto de interrupción (``break loop``) o cualquier otra operación de depuración.

Ejemplo de sesión de depuración
-------------------------

Cree un nuevo boceto y pegue el siguiente código en él:

.. code:: cpp

    #include <GDBStub.h>
    
    void setup() {
      Serial.begin(115200);
      gdbstub_init();
      Serial.printf("Iniciando...\n");
    }
    
    void loop() {
      static uint32_t cnt = 0;
      Serial.printf("%d\n", cnt++);
      delay(100);
    }

Guárdelo, compilelo y carguelo en su ESP8266. En el Monitor Serie debería ver algo como:

.. code:: cpp

    Iniciando...
    1
    2
    3
    ....


Ahora cierre el Monitor Serie.

Abra un Símbolo de sistema y busque el fichero ELF:

.. code:: cpp

    earle@server:~$ find /tmp -name "*.elf" -print
    /tmp/arduino_build_257110/testgdb.ino.elf
    /tmp/arduino_build_531411/listfiles.ino.elf
    /tmp/arduino_build_156712/SDWebServer.ino.elf

En este ejemplo, se encuentran varios archivos ``elf``, pero solo nos importa el que acabamos de crear, ``testgdb.ino.elf``.

Abre el GDB específico para ESP8266 adecuado

.. code:: cpp

    earle@server:~$ ~/.arduino15/packages/esp8266/hardware/xtensa-lx106-elf/bin/xtensa-lx106-elf-gdb
    GNU gdb (GDB) 8.2.50.20180723-git
    Copyright (C) 2018 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    Type "show copying" and "show warranty" for details.
    This GDB was configured as "--host=x86_64-linux-gnu --target=xtensa-lx106-elf".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
        <http://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
        <http://www.gnu.org/software/gdb/documentation/>.

    For help, type "help".
    Type "apropos word" to search for commands related to "word".
    (gdb) 

Ahora estamos en el indicador de GDB, pero no se ha configurado nada para el ESP8266 y no se ha cargado información de depuración. Cortar y pegar las opciones de configuración:

.. code:: cpp
    (gdb) set remote hardware-breakpoint-limit 1
    (gdb) set remote hardware-watchpoint-limit 1
    (gdb) set remote interrupt-on-connect on
    (gdb) set remote kill-packet off
    (gdb) set remote symbol-lookup-packet off
    (gdb) set remote verbose-resume-packet off
    (gdb) mem 0x20000000 0x3fefffff ro cache
    (gdb) mem 0x3ff00000 0x3fffffff rw
    (gdb) mem 0x40000000 0x400fffff ro cache
    (gdb) mem 0x40100000 0x4013ffff rw cache
    (gdb) mem 0x40140000 0x5fffffff ro cache
    (gdb) mem 0x60000000 0x60001fff rw
    (gdb) set serial baud 115200
    (gdb) 

Y dile a GDB dónde se encuentra el archivo ELF de información de depuración:

.. code:: cpp

    (gdb) file /tmp/arduino_build_257110/testgdb.ino.elf
    Reading symbols from /tmp/arduino_build_257110/testgdb.ino.elf...done.

Ahora, conéctate al ESP8266 en ejecución:

.. code:: cpp

    (gdb)     target remote /dev/ttyUSB0
    Remote debugging using /dev/ttyUSB0
    0x40000f68 in ?? ()
    (gdb)

No se preocupe de que GDB no sepa qué hay en nuestra dirección actual, ingresamos el código en un lugar aleatorio y podríamos estar en una interrupción, en la ROM o en cualquier otro lugar. Lo importante es que ahora estamos conectados y ahora sucederán dos cosas: podemos depurar y la salida Serie de la aplicación se mostrará en la consola GDB.

Continúa la aplicación en ejecución para ver la salida en Serie:

.. code:: cpp

    (gdb) cont
    Continuing.
    74
    75
    76
    77
    ...

La aplicación vuelve a funcionar y podemos detenerla en cualquier momento usando ``Ctrl-C``:

.. code:: cpp 
    113
    ^C
    Program received signal SIGINT, Interrupt.
    0x40000f68 in ?? ()
    (gdb) 

En este punto, podemos establecer un punto de interrupción en el ``loop()`` principal y reiniciar para ingresar nuestro propio código:

.. code:: cpp

    (gdb) break loop
    Breakpoint 1 at 0x40202e33: file /home/earle/Arduino/sketch_dec26a/sketch_dec26a.ino, line 10.
    (gdb) cont
    Continuing.
    Note: automatically using hardware breakpoints for read-only addresses.
    bcn_timout,ap_probe_send_start
    
    Breakpoint 1, loop () at /home/earle/Arduino/sketch_dec26a/sketch_dec26a.ino:10
    10	void loop()
    (gdb) 

Examinemos la variable local:

.. code:: cpp
    (gdb) next
    loop () at /home/earle/Arduino/sketch_dec26a/sketch_dec26a.ino:13
    13      Serial.printf("%d\n", cnt++);
    (gdb) print cnt
    $1 = 114
    (gdb) 

Y cambiémosla:

.. code:: cpp

    $2 = 114
    (gdb) set cnt = 2000
    (gdb) print cnt
    $3 = 2000
    (gdb) 

Y reinicie la aplicación y vea que nuestros cambios surten efecto:

.. code:: cpp

    (gdb) cont
    Continuing.
    2000
    Breakpoint 1, loop () at /home/earle/Arduino/sketch_dec26a/sketch_dec26a.ino:10
    10	void loop() {
    (gdb) cont
    Continuing.
    2001
    Breakpoint 1, loop () at /home/earle/Arduino/sketch_dec26a/sketch_dec26a.ino:10
    10	void loop() {
    (gdb) 

Parece que dejamos el punto de interrupción en ``loop()``, deshagámonos de él e intentemos nuevamente:

.. code:: cpp

    (gdb) delete
    Delete all breakpoints? (y or n) y
    (gdb) cont
    Continuing.
    2002
    2003
    2004
    2005
    2006
    ....

En este punto, podemos salir de GDB con ``quit`` o hacer más depuración.

Limitaciones de depuración del Hardware ESP8266
--------------------------------------

El ESP8266 solo admite un único punto de interrupción de hardware y un solo punto de vigilancia de datos de hardware. Esto significa que solo se permite un punto de interrupción en el código de usuario en cualquier momento. Considere usar el comando ``thb`` (punto de interrupción temporal de hardware) en GDB mientras realiza la depuración en lugar del comando más común ``break``, ya que ``thb`` eliminará el punto de interrupción una vez que se alcance automáticamente y le ahorrará algunos problemas .
