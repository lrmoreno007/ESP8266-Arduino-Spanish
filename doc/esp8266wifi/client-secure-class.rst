Clase: Client Secure
-------------------

Los métodos y propiedades que se describen en esta sección son específicos de ESP8266. No están cubiertos en la documentación de la `librería WiFi de Arduino <https://www.arduino.cc/en/Reference/WiFi>`__. Antes de que estén complétamente documentados, consulte la información a continuación.

Criptografía soportada
~~~~~~~~~~~~~~~~

En el fondo, se usa la librería `axtls <http://axtls.sourceforge.net>`_. La librería solo admite certificados de RSA y no hay nuevos certificados de curva elíptica. TLSv1.2 es compatible desde SDK 2.4.0-rc1.

Los siguientes cifrados y compendios son compatibles con las `especificaciones <http://axtls.sourceforge.net/specifications.htm>`_:

* Cifrado simétrico
    * AES128-SHA
    * AES256-SHA
    * AES128-SHA256
    * AES256-SHA256
* Cifrado asimétrico
    * RSA 512/1024/2048/4096 bit encriptado/desencriptado.
    * RSA firma/verificación
* Compendios
    * SHA1
    * MD5
    * SHA256/384/512
    * HMAC-SHA1
    * HMAC-MD5
    * HMAC-SHA256

loadCertificate
~~~~~~~~~~~~~~~

Carga el certificado de cliente desde el sistema de archivos.

.. code:: cpp

    loadCertificate(file) 

*Declaración*

.. code:: cpp

    #include <FS.h>
    #include <ESP8266WiFi.h>
    #include <WiFiClientSecure.h>

    const char* certyficateFile = "/client.cer";

*setup() o loop()*

.. code:: cpp

    if (!SPIFFS.begin()) 
    {
      Serial.println("Fallo al montar el sistema de ficheros");
      return;
    }

    Serial.printf("Abriendo %s", certyficateFile);
    File crtFile = SPIFFS.open(certyficateFile, "r");
    if (!crtFile)
    {
      Serial.println(" Falló!");
    }

    WiFiClientSecure client;

    Serial.print("Cargando %s", certyficateFile);
    if (!client.loadCertificate(crtFile))
    {
      Serial.println(" Falló!");
    }

    // proceder con la conexión del cliente al host


setCertificate
~~~~~~~~~~~~~~

Carga el certificado de cliente desde un array C.

.. code:: cpp

    setCertificate (array, size) 

Para un ejemplo práctico, compruebe `este interesante blog <https://nofurtherquestions.wordpress.com/2016/03/14/making-an-esp8266-web-accessible/>`__.

Otras llamadas a funciones
~~~~~~~~~~~~~~~~~~~~

.. code:: cpp

    bool  verify (const char *fingerprint, const char *domain_name) 
    void  setPrivateKey (const uint8_t *pk, size_t size) 
    bool  loadCertificate (Stream &stream, size_t size) 
    bool  loadPrivateKey (Stream &stream, size_t size) 
    template<typename TFile >  bool  loadPrivateKey (TFile &file)

La documentación para las funciones anteriores aún no se ha realizado.

Consulte la sección separada con `ejemplos <client-secure-examples.rst>`__ dedicados específicamente a la clase Client Secure.
