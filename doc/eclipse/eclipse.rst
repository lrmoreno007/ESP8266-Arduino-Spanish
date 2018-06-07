Utilizar Eclipse con Arduino ESP8266
==================================

Que descargar
~~~~~~~~~~~~~~~~

-  `IDE Arduino <https://www.arduino.cc/en/Main/Software>`__
-  `IDE Eclipse para desarrolladores C/C++ <http://www.eclipse.org/downloads/packages/eclipse-ide-cc-developers/oxygen3a>`__
-  `Java <http://www.java.com/>`__

Configurar Arduino
~~~~~~~~~~~~~

Ver el `documento <https://github.com/esp8266/Arduino#installing-with-boards-manager>`__

Configurar Eclipse
~~~~~~~~~~~~~

-  `Paso 1 <http://www.baeyens.it/eclipse/how_to.shtml#/c>`__
-  `Paso 2 <http://www.baeyens.it/eclipse/how_to.shtml#/e>`__
-  Ve a Window --> preferences --> Arduino
-  Añade en "Private hardware path" una nueva línea a la ruta de Arduino ESP8266 

Ejemplo de Ruta privada de hardware (Private hardware path) 
                             

::

    Windows: C:\Users\[username]\AppData\Roaming\Arduino15\packages\esp8266\hardware
    Linux: /home/[username]/.arduino15/packages/esp8266/hardware

Eclipse no compila
~~~~~~~~~~~~~~~~~~

Si Eclipse no encuentra la ruta al compilador añade a "platform.txt" tras:

::

    version=1.6.4

esto:

::

    runtime.tools.xtensa-lx106-elf-gcc.path={runtime.platform.path}/../../../tools/xtensa-lx106-elf-gcc/1.20.0-26-gb404fb9
    runtime.tools.esptool.path={runtime.platform.path}/../../../tools/esptool/0.4.4

Notas: 

- La ruta puede cambiar, comprueba la versión actual.
- Cada actualización del IDE Arduino eliminará la solución.
- Puede no necesitarse en el futuro si el Plugin Eclipse se actualiza.
