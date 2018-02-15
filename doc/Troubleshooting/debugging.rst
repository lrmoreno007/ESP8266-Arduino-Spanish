Depuración
=========

Introducción
------------

Desde 2.1.0-rc1 el core incluye una función de depuración que es controlable desde el menú IDE.

Los nuevos puntos de menú administran los mensajes de depuración en tiempo real.

Requerimientos
~~~~~~~~~~~~

Para realizar la depuración se requiere una conexión serie (Serial o Serial1).

El interfaz Serial necesita ser inicializado en ``setup()``.

Establece la velocidad en baudios de Serial tan alta como te lo permita tu Hardware.

Sketch mínimo para realizar la depuración:

.. code:: cpp

    void setup() {
        Serial.begin(115200);
    }

    void loop() {
    }

Uso
~~~~~

1. Selecciona el interfaz Serie para los mensajes de depuración: |Debug-Port|

2. Selecciona que tipo/nivel deseas de mensajes de depuración: |Debug-Level| 

3. Comprueba si el interfaz Serial está inicializado en ``setup()`` (ver
   `Requerimientos <#requerimientos>`__) .

4. Sube el sketch.

5. Comprueba la salida serie.

Información
------------

Funciona con cada sketch que active el interfaz serie que debe ser seleccionado como Debug Port.

El interfaz serie puede usarse normalmente también en el sketch.

La salida de depuración es adicional y no desactiva ningún interfaz del sketch.

Para desarrolladores
~~~~~~~~~~~~~~

Para el manejo de la depuración utiliza defines.

La definición se realiza por líneas de comandos.

Debug Port - Puerto de depuración
^^^^^^^^^^

El puerto tiene el define ``DEBUG_ESP_PORT`` posibles valores: 
- Desactivado: no existe el define.
- Serial: Serial
- Serial1: Serial1

Debug Level - Nivel de depuración
^^^^^^^^^^^

Todos los defines para los diferentes niveles comienzan con ``DEBUG_ESP_``

Puede encontrar una lista completa en el fichero `boards.txt <https://github.com/esp8266/Arduino/blob/master/boards.txt#L180>`__

Ejemplo para sus propios mensajes de depuración
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Los mensajes de depuración serán mostrados solo cuando se establezca el Debug Port en el menú del IDE.

.. code:: cpp

    #ifdef DEBUG_ESP_PORT
    #define DEBUG_MSG(...) DEBUG_ESP_PORT.printf( __VA_ARGS__ )
    #else
    #define DEBUG_MSG(...) 
    #endif

    void setup() {
        Serial.begin(115200);
        
        delay(3000);
        DEBUG_MSG("Iniciando...\n");
    }

    void loop() {
        DEBUG_MSG("loop %d\n", millis());
        delay(1000);
    }

.. |Debug-Port| image:: debug_port.png
.. |Debug-Level| image:: debug_level.png
