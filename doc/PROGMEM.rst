Guía para PROGMEM sobre ESP8266 y Arduino IDE
===========================================

Introducción
-----

PROGMEM es una característica Arduino AVR que ha sido portada a ESP8266 para asegurar la compatibilidad con las librerías existentes en Arduino, así como para ahorrar RAM. En ESP8266 al declarar una cadena como ``const char * xyz = "this is a string"`` colocará esta cadena en la RAM, no en la flash.  Es posible colocar una String en la flash, y después cargarla a la RAM cuando sea necesario.  En un AVR de 8bit AVR este proceso es muy simple. En el ESP8266 de 32bit hay condiciones que deben cumplirse para leer desde el flash.

En ESP8266, PROGMEM es una macro: 

.. code:: cpp

    #define PROGMEM   ICACHE_RODATA_ATTR

``ICACHE_RODATA_ATTR`` está definida por:

.. code:: cpp

    #define ICACHE_RODATA_ATTR  __attribute__((section(".irom.text")))

Coloca la variable en la sección .irom.text de la flash. Colocar cadenas en la flash requiere el uso de alguno de los siguientes métodos.  

| ### Declarar una cadena global almacenada en la flash.

.. code:: cpp

    static const char xyz[] PROGMEM = "Esta es una cadena almacenada en la flash";

Declarar una cadena flash en un bloque de código.
-----------------------------------------

Para esto puedes usar la macro PSTR. Que está completamente definida en `pgmspace.h <https://github.com/esp8266/Arduino/blob/master/cores/esp8266/pgmspace.h>`__

.. code:: cpp

    #define PGM_P       const char *
    #define PGM_VOID_P  const void *
    #define PSTR(s) (__extension__({static const char __c[] PROGMEM = (s); &__c[0];}))

En la práctica:

.. code:: cpp

    void myfunction(void) {
    PGM_P xyz = PSTR("Almacena esta cadena en la flash");
    const char * abc = PSTR("También almacena esta cadena en la flash");
    }

Los dos ejemplos anteriores almacenarán cadenas en la flash. Para recuperar y manipular cadenas de flash debe leerse desde la flash en palabras de 4 bytes. En el IDE Arduino para ESP8266 hay varias funciones que pueden ayudar a recuperar cadenas desde la flash que han sido almacenadas utilizando PROGMEM. Ambos ejemplos anteriores devuelven ``const char *``. Sin embargo el uso de estos punteros, sin corregir el alineamiento de 32bit provocará un error de segmentación y el ESP8266 se caerá. Debes leer desde la flash de forma alineada a 32-bits.

Funciones para leer desde PROGMEM
-----------------------------------

Está completamente definido en `pgmspace.h <https://github.com/esp8266/Arduino/blob/master/cores/esp8266/pgmspace.h>`__

.. code:: cpp

    int memcmp_P(const void* buf1, PGM_VOID_P buf2P, size_t size);
    void* memccpy_P(void* dest, PGM_VOID_P src, int c, size_t count);
    void* memmem_P(const void* buf, size_t bufSize, PGM_VOID_P findP, size_t findPSize);
    void* memcpy_P(void* dest, PGM_VOID_P src, size_t count);
    char* strncpy_P(char* dest, PGM_P src, size_t size);
    char* strcpy_P(dest, src)          
    char* strncat_P(char* dest, PGM_P src, size_t size);
    char* strcat_P(dest, src)         
    int strncmp_P(const char* str1, PGM_P str2P, size_t size);
    int strcmp_P(str1, str2P)        
    int strncasecmp_P(const char* str1, PGM_P str2P, size_t size);
    int strcasecmp_P(str1, str2P)        
    size_t strnlen_P(PGM_P s, size_t size);
    size_t strlen_P(strP)     
    char* strstr_P(const char* haystack, PGM_P needle);
    int printf_P(PGM_P formatP, ...);
    int sprintf_P(char *str, PGM_P formatP, ...);
    int snprintf_P(char *str, size_t strSize, PGM_P formatP, ...);
    int vsnprintf_P(char *str, size_t strSize, PGM_P formatP, va_list ap);

Hay muchas funciones pero en realidad son versiones ``_P`` de las funciones estándar C que han sido adaptadas para leer con el alineamiento de la flash de ESP8266 de 32bit. Todas ellas toman un ``PGM_P`` que es esencialmente un ``const char *``. Debajo de la capa se usan todas estas funciones, un proceso para garantizar que se lean 4 bytes y se devuelva el byte de la solicitud.

Esto funciona bien cuando se ha diseñado una función como la anterior, que está especializada para tratar con los punteros de PROGMEM, pero no hay verificación de tipos, excepto contra ``const char *``. Esto significa que es totalmente legítimo, en lo que respecta al compilador, pasar cualquier cadena ``const char *`` , lo que obviamente no es cierto y conducirá a un comportamiento indefinido. Esto hace que sea imposible crear cualquier función sobrecargada que pueda usar cadenas de flash cuando están definidas como `` PGM_P``. Si lo intenta, obtendrá un error de sobrecarga ambiguo como ``PGM_P`` == ``const char *`` .

Introduzca \_\_FlashStringHelper... Esta es una clase contenedora que permite que las cadenas flash se usen como una clase, esto significa que la verificación de tipos y la sobrecarga de funciones se pueden usar con cadenas flash. La mayoría de las personas estarán familiarizadas con la macro ``F()`` y posiblemente con la macro FPSTR(). Estos se definen en `WString.h <https://github.com/esp8266/Arduino/blob/master/cores/esp8266/WString.h#L37>`__:

.. code:: cpp

    #define FPSTR(pstr_pointer) (reinterpret_cast<const __FlashStringHelper *>(pstr_pointer))
    #define F(string_literal) (FPSTR(PSTR(string_literal)))

Así ``FSPTR()`` toma un puntero PROGMEM a una cadena y lo arroja a esta clase ``__FlashStringHelper``. Por lo tanto, si ha definido una cadena como la anterior ``xyz``, puede usar ``FPSTR()`` para convertirla en ``__FlashStringHelper`` para pasarla a las funciones que la toman.

.. code:: cpp

    static const char xyz[] PROGMEM = "Esta es una cadena almacenada en la flash";
    Serial.println(FPSTR(xyz));

El ``F()`` combina ambos métodos para crear una forma fácil y rápida de almacenar una cadena alineada en flash, y devolver el tipo ``__FlashStringHelper``. Por ejemplo:

.. code:: cpp

    Serial.println(F("Esta es una cadena almacenada en la flash"));

Aunque estas dos funciones proporcionan una función similar, cumplen funciones diferentes. ``FPSTR()`` te permite definir una cadena flash global y luego usarla en cualquier función que tome ``__FlashStringHelper``. ``F()`` le permite definir estas cadenas de flash en un lugar determinado, pero no puede usarlas en ningún otro lado. La consecuencia de esto es que puede compartir cadenas comunes usando ``FPSTR()`` pero no ``F()``. ``__FlashStringHelper`` es lo que la clase String usa para sobrecargar su constructor:

.. code:: cpp

    String(const char *cstr = ""); // constructor from const char * 
    String(const String &str); // copy constructor
    String(const __FlashStringHelper *str); // constructor for flash strings 

Esto te permite escribir:

.. code:: cpp

    String mystring(F("Esta cadena está almacenada en la flash"));

¿Cómo escribo una función para usar \_\_FlashStringHelper? Sencillo: vuelva a colocar el puntero en un PGM\_P y use las funciones ``_P`` que se muestran arriba. Esto es un ejemplo de implementación para String para la función concat.

.. code:: cpp

    unsigned char String::concat(const __FlashStringHelper * str) {
        if (!str) return 0; // return if the pointer is void
        int length = strlen_P((PGM_P)str); // Lo echa a PGM_P, que es básicamente const char *, y mide usando la versión _P de strlen.
        if (length == 0) return 1;
        unsigned int newlen = len + length;
        if (!reserve(newlen)) return 0; // Crea un buffer de la longitud correcta
        strcpy_P(buffer + len, (PGM_P)str); //copia la cadena dentro usando strcpy_P
        len = newlen;
        return 1;
    }

¿Como declaro una cadena global flash y la uso?
--------------------------------------------------

.. code:: cpp

    static const char xyz[] PROGMEM = "Esta es una cadena almacenada en la flash. Longitud = %u";

    void setup() {
        Serial.begin(115200); Serial.println(); 
        Serial.println( FPSTR(xyz) ); // Para imprimir la cadena, debes convertirla a FlashStringHelper primero usando FPSTR(). 
        Serial.printf_P( xyz, strlen_P(xyz)); // Utiliza printf con la cadena PROGMEM
    }

Como uso cadenas alineadas flash?
----------------------------------

.. code:: cpp

    void setup() {
        Serial.begin(115200); Serial.println(); 
        Serial.println( F("Esto es una cadena alineada")); // 
        Serial.printf_P( PSTR("Esto es una cadena alineada utilizando printf %s"), "hola");
    }

¿Como declaro y uso datos en PROGMEM?
-----------------------------------------

.. code:: cpp

    const size_t len_xyz = 30;
    const uint8_t xyz[] PROGMEM = {
      0x53, 0x61, 0x79, 0x20, 0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x20, 
      0x74, 0x6f, 0x20, 0x4d, 0x79, 0x20, 0x4c, 0x69, 0x74, 0x74, 
      0x6c, 0x65, 0x20, 0x46, 0x72, 0x69, 0x65, 0x6e, 0x64, 0x00};

     void setup() {
         Serial.begin(115200); Serial.println(); 
         uint8_t * buf = new uint8_t[len_xyz];
         if (buf) {
          memcpy_P(buf, xyz, len_xyz);
          Serial.write(buf, len_xyz); // Da salida al buffer. 
         }
     }

¿Como declaro algunos datos en PROGMEM y recupero un byte de él?
---------------------------------------------------------------------

Declara el dato como se hizo anteriormente, entonces utiliza ``pgm_read_byte`` para coger el valor de vuelta.

.. code:: cpp

    const size_t len_xyz = 30;
    const uint8_t xyz[] PROGMEM = {
      0x53, 0x61, 0x79, 0x20, 0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x20,
      0x74, 0x6f, 0x20, 0x4d, 0x79, 0x20, 0x4c, 0x69, 0x74, 0x74,
      0x6c, 0x65, 0x20, 0x46, 0x72, 0x69, 0x65, 0x6e, 0x64, 0x00
    };

    void setup() {
      Serial.begin(115200); Serial.println();
      for (int i = 0; i < len_xyz; i++) {
        uint8_t byteval = pgm_read_byte(xyz + i);
        Serial.write(byteval); // Da salida al buffer.
      }
    }

En resumen
----------

Es fácil almacenar cadenas en la flash usando ``PROGMEM`` y ``PSTR``, pero debe crear funciones que utilicen específicamente los punteros que generan, ya que básicamente son ``const char *``. Por otro lado, ``FPSTR`` y ``F()`` te ofrecen una clase con la que puede hacer conversiones implícitas, muy útil cuando se sobrecarga funciones y se realizan conversiones de tipo implícitas. Vale la pena agregar que si desea almacenar un ``int``, ``float`` o un puntero, estos pueden ser almacenados y leídos directamente ya que tienen 4 bytes de tamaño y por lo tanto siempre estarán alineados.

Espero que esto ayude.
