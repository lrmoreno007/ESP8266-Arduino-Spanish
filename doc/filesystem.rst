Sistema de ficheros
==========

Esquema de la memoria flash
------------

Aunque el sistema de archivos está almacenado en el mismo chip flash que el sketch, la programación de un nuevo boceto no modificará el contenido del sistema de archivos. Esto permite utilizar el sistema de archivos para almacenar datos del sketch, archivos de configuración o contenido para el servidor web.

El siguiente diagrama ilustra el esquema o plantilla utilizado por el entorno Arduino:

::

    |--------------|----------|--------------------|---|---|---|---|---|
    ^              ^          ^                    ^       ^
    Sketch    Actualiz. OTA   Sistema de ficheros  EEPROM  Config. WiFi (SDK)

El tamaño del sistema de ficheros depende del tamaño del chip flash. Dependiendo de que tarjeta se ha seleccionado en el IDE, tendrás las siguientes opciones de tamaño flash:

+---------------------------------+--------------------------+---------------------------+
| Tarjeta                         | tañano chip flash, bytes | Tamaño sistem. fich, bytes|
+=================================+==========================+===========================+
| Generic module                  | 512k                     | 64k, 128k                 |
+---------------------------------+--------------------------+---------------------------+
| Generic module                  | 1M                       | 64k, 128k, 256k, 512k     |
+---------------------------------+--------------------------+---------------------------+
| Generic module                  | 2M                       | 1M                        |
+---------------------------------+--------------------------+---------------------------+
| Generic module                  | 4M                       | 3M                        |
+---------------------------------+--------------------------+---------------------------+
| Adafruit HUZZAH                 | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+
| ESPresso Lite 1.0               | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+
| ESPresso Lite 2.0               | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+
| NodeMCU 0.9                     | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+
| NodeMCU 1.0                     | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+
| Olimex MOD-WIFI-ESP8266(-DEV)   | 2M                       | 1M                        |
+---------------------------------+--------------------------+---------------------------+
| SparkFun Thing                  | 512k                     | 64k                       |
+---------------------------------+--------------------------+---------------------------+
| SweetPea ESP-210                | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+
| WeMos D1 & D1 mini              | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+
| ESPDuino                        | 4M                       | 1M, 3M                    |
+---------------------------------+--------------------------+---------------------------+

**Nota** para utilizar funciones del sistema de ficheros en el sketch, añade la siguiente línea include al sketch:

.. code:: cpp

    #include "FS.h"

Limitaciones del sistema de ficheros
-----------------------

La implementación del sistema de archivos para ESP8266 tuvo que acomodarse a las restricciones del chip, entre las cuales está su RAM limitada. `SPIFFS <https://github.com/pellepl/spiffs>`__ fue seleccionado porque está diseñado para sistemas pequeños, pero tiene como coste algunas simplificaciones y limitaciones.

Primero, por detrás, SPIFFS no soporta directorios, solo almacena una lista "plana" de ficheros. Pero en contra de un sistema de ficheros tradicional, el caracter "slash" ``'/'`` está permitido en los nombres de ficheros, por lo que las funciones que se ocupan de listar directorios (por ejemplo, ``openDir("/website")`` ) básicamente solo filtra los nombres de archivo y conserva los que comienzan con el prefijo solicitado ( ``/website/`` ). En términos prácticos, eso hace poca diferencia sin embargo.

Segundo, existe una limitación a 32 caracteres en total en los nombres de ficheros. Un caracter ``'\0'`` está reservado para la cadena C de terminación, lo que nos deja 31 caracteres para utilizar.

Combinado, significa que se recomienda mantener los nombres de archivo cortos y no usar directorios profundamente anidados, como la ruta completa de cada archivo (incluido directorios, caracteres ``'/'`` , nombre base, punto y extensión) tiene que ser
31 caracteres como máximo. Por ejemplo, el nombre de archivo ``/website/images/bird_thumbnail.jpg`` tiene 34 caracteres y causará problemas si se utiliza, por ejemplo en ``exists()`` o en caso de que otro archivo comience con los mismos primeros 31 caracteres.

**Peligro**: Ese límite se alcanza fácilmente y si se ignora, los problemas podrían pasar desapercibidos porque no aparecerá ningún mensaje de error en la compilación ni en el tiempo de ejecución.

Para mas detalles sobre la implementación interna de SPIFFS, ver el `fichero readme SPIFFS <https://github.com/esp8266/Arduino/blob/master/cores/esp8266/spiffs/README.md>`__.

Subiendo ficheros al sistema de archivos
------------------------------

*ESP8266FS* es una herramienta que se integra en el IDE de Arduino. Añadiendo una nueva casilla al menú *Herramientas* para subir el contenido del directorio "data" del sketch al sistema de ficheros flash del ESP8266.

-  Descarga la herramienta: https://github.com/esp8266/arduino-esp8266fs-plugin/releases/download/0.3.0/ESP8266FS-0.3.0.zip.
-  En el directorio de sketchs de Arduino, crea el directorio ``tools`` si no existe todavía.
-  Descomprime la herramienta en el directorio ``tools`` (la ruta debe quedar ``<home_dir>/Arduino/tools/ESP8266FS/tool/esp8266fs.jar``).
-  Reinicia el IDE de Arduino.
-  Abre el sketch (o crea uno nuevo y sálvalo)
-  Ve al directorio del sketch (selecciona Programa > Mostrar carpeta de programa)
-  Crea un directorio llamado ``data`` y algún fichero que quieras tener en el sistema de ficheros.
-  Asegúrate de tener tu tarjeta seleccionada, el puerto (COM, tty, etc) y cierra el Monitor Serie.
-  Selecciona Herramientas > ESP8266 Sketch Data Upload. Debería comenzar la subida de los ficheros a sistema de ficheros flash del ESP8266. Cuando acabe, la barra de estado del IDE mostrará el mensaje ``SPIFFS Image Uploaded``.

Objeto sistema de ficheros *SPIFFS*
---------------------------

begin
~~~~~

.. code:: cpp

    SPIFFS.begin()

Este método monta el sistema de ficheros SPIFFS. Debe ser llamado antes de usar cualquier otro API del sistema de ficheros. Devuelve *true* si el sistema de archivos se ha montado satisfactoriamente, *false* en caso contrario.

end
~~~

.. code:: cpp

    SPIFFS.end()

Este método desmonta el sistema de ficheros SPIFFS. Utiliza este método antes de realizar una actualización OTA del SPIFFS.

format
~~~~~~

.. code:: cpp

    SPIFFS.format()

Formatea el sistema de ficheros. Se puede llamar antes o después de llamar ``begin``. Devuelve *verdadero* si el formateo tuvo éxito.

open
~~~~

.. code:: cpp

    SPIFFS.open(path, mode)

Abre un fichero. ``path`` debe ser un camino absoluto comenzando con un slash (p.ej. ``/dir/filename.txt``). ``mode`` es una palabra que especifica el modo de acceso. Puede ser una de las siguientes: "r", "w", "a", "r+", "w+", "a+". El significado de estos modos es el mismo que para la función ``fopen`` en C.

::

       r      Abre un fichero de texto para leerlo. La secuencia se coloca en el comienzo del archivo.

       r+     Abre un fichero para lectura y escritura. La secuencia se coloca en el comienzo del archivo.

       w      Trunca en fichero con una longitud cero o crea un fichero de texto para escritura. 
              La secuencia se coloca en el comienzo del archivo.

       w+     Abre para lectura y escritura. El fichero se crea si no existe, de lo contrario se trunca.
              La secuencia se coloca en el comienzo del archivo.

       a      Abre el fichero para añadir (escribiendo al final del fichero). El fichero se crea si no existe.
              La secuencia se coloca al final del archivo.

       a+     Abre el fichero para añadir (escribiendo al final del fichero). El fichero se crea si no existe.
              La posición inicial para lectura es al comienzo del fichero, pero la salida es siempre añadida 
              al final del fichero

Devuelve el objeto *File*. Para comprobar si el archivo se abrió con éxito, utilice un operador booleano.

.. code:: cpp

    File f = SPIFFS.open("/f.txt", "w");
    if (!f) {
        Serial.println("No se pudo abrir el fichero");
    }

exists
~~~~~~

.. code:: cpp

    SPIFFS.exists(path)

Devuelve *true* si existe el archivo con la ruta indicada, *false* en caso contrario.

openDir
~~~~~~~

.. code:: cpp

    SPIFFS.openDir(path)

Abre un directorio en la ruta absoluta indicada. Devuelve un objeto *Dir*.

remove
~~~~~~

.. code:: cpp

    SPIFFS.remove(path)

Elimina el fichero de la ruta absoluta indicada. Devuelve *true* si el fichero se borró satisfactoriamente.

rename
~~~~~~

.. code:: cpp

    SPIFFS.rename(pathFrom, pathTo)

Renombra el fichero ``pathFrom`` a ``pathTo``. La ruta debe ser absoluta. Devuelve *true* si el fichero se renombra satisfactoriamente.

info
~~~~

.. code:: cpp

    FSInfo fs_info;
    SPIFFS.info(fs_info);

Rellena la `estructura FSInfo <#filesystem-information-structure>`__ con información sobre el sistema de ficheros. Devuelve ``true`` si tiene éxito, ``false`` en caso contrario.

Estructura de información del sistema de archivos
--------------------------------

.. code:: cpp

    struct FSInfo {
        size_t totalBytes;
        size_t usedBytes;
        size_t blockSize;
        size_t pageSize;
        size_t maxOpenFiles;
        size_t maxPathLength;
    };

Esta es la estructura que se rellena al usar el método FS::info .

- ``totalBytes``    — Tamaño total de datos útiles en el sistema de archivos.

- ``usedBytes``     — Número de bytes usado por los ficheros.

- ``blockSize``     — Tamaño del bloque SPIFFS.

- ``pageSize``      — Tamaño de la página lógica SPIFFS.

- ``maxOpenFiles``  — Número máximo de archivos que pueden estar abiertos simultáneamente.

- ``maxPathLength`` — Longitud máxima del nombre de archivo (incluido un byte cero de terminación).

Objeto directorio *Dir*
----------------------

El propósito del objeto *Dir* es iterar sobre los ficheros dentro del directorio. Provee tres métodos: ``next()``, ``fileName()``, y ``openFile(mode)``.

El siguiente ejemplo muestra como debe utilizarse:

.. code:: cpp

    Dir dir = SPIFFS.openDir("/data");
    while (dir.next()) {
        Serial.print(dir.fileName());
        File f = dir.openFile("r");
        Serial.println(f.size());
    }

``dir.next()`` devuelve *true* mientras haya ficheros en el directorio para iterar. Debe llamarse antes de llamar a las funciones ``fileName`` y ``openFile`` .

El método ``openFile`` toma el argumento *mode* que tiene el mismo significado que para la función ``SPIFFS.open`` .

Objeto fichero *File*
-----------

Las funciones ``SPIFFS.open`` y ``dir.openFile`` devuelven un objeto *File*. Este objeto soporta todas las funciones de *Stream*, para que puedas usar ``readBytes``, ``findUntil``, ``parseInt``, ``println`` y todos los otros métodos *Stream*.

Hay algunas funciones que son específicas del objeto *File*.

seek
~~~~

.. code:: cpp

    file.seek(offset, mode)

Esta función se comporta como la función C ``fseek`` . Dependiendo de un valor de ``mode``, se mueve a la posición actual en un fichero de la siguiente manera:

-  Si ``mode`` es ``SeekSet``, la posición se establece a ``offset`` bytes desde el comienzo del fichero.
-  Si ``mode`` es ``SeekCur``, la posición actual se mueve a ``offset`` bytes.
-  Si ``mode`` es ``SeekEnd``, la posición se establece a ``offset`` bytes desde el final del fichero.

Devuelve *true* si la posición se estableció satisfactoriamente.

position
~~~~~~~~

.. code:: cpp

    file.position()

Devuelve la posición actual dentro del fichero, en bytes.

size
~~~~

.. code:: cpp

    file.size()

Devuelve el tamaño del fichero, en bytes.

name
~~~~

.. code:: cpp

    String name = file.name();

Devuelve el nombre del fichero, como ``const char*``. Conviértelo a *String* para almacenarlo

close
~~~~~

.. code:: cpp

    file.close()

Cierra el fichero. Ninguna otra operación debe realizarse sobre el objeto *File* después de llamar a la función ``close`` .
