Traducción al español de la documentación de ESP8266 Arduino 
==============================================================

Este repositorio contiene la traducción al español (no oficial) de la documentación del core Arduino para ESP8266 que puedes encontrar en https://github.com/esp8266/Arduino. La documentación que aún no se ha traducido permanece en inglés hasta su traducción.

Si encuentras algún problema en esta traducción siéntete libre de abrir un nuevo issue en este repositorio y si quieres colaborar puedes traducir los ficheros que aún quedan por traducir o colaborar con la traducción de los cambios, mediante un pull request. 

**ATENCIÓN!!! Todos los "issue" y "pull request" respecto al core Arduino para ESP8266 deben realizarse en el [repositorio original](https://github.com/esp8266/Arduino) y SIEMPRE EN INGLÉS, ya que es el idioma oficial del proyecto. No se atenderán peticiones en ningún otro idioma.**

## DOCUMENTACIÓN

Puedes encontrar la mayor parte de la documentación en español en http://esp8266-arduino-spanish.readthedocs.io/es/latest/index.html.

En este repositorio de Github encontrarás también el fichero POLICY.md donde se exponen las reglas de comunicación del proyecto ESP8266/Arduino y el presente documento que es una traducción del fichero README.md.

A partir de aquí la traducción de README.es del proyecto ESP8266/Arduino

-------------------------------------------
Arduino core para el chip ESP8266 WiFi
===========================================

Este proyecto da soporte para el chip ESP8266 en el entorno Arduino. Permite escribir sketches usando funciones y librerías de la familia Arduino, y las ejecuta directamente en ESP8266, sin necesidad de ningún microcontrolador externo.

ESP8266 Arduino core tiene librerías para comunicarse sobre WiFi usando TCP y UDP, establece HTTP, mDNS, SSDP, y servidores DNS, hace actualizaciones OTA, utiliza un sistema de ficheros en la memoria flash, trabaja con tarjetas SD, servos, perifericos SPI y I2C.

**Todos los "issue" y "pull request" deben realizarse en inglés, ya que es el idioma oficial del proyecto ESP8266/Arduino. No se atenderán peticiones en ningún otro idioma.**

# Contenido
- Opciones (tipos) de instalación:
  - [Utilizando el Gestor de Tarjetas](#instalando-mediante-el-gestor-de-tarjetas)
  - [Utilizando la version git](#usando-versión-git)
  - [Utilizando PlatformIO](#usando-platformio)
  - [Construyendolo con make](#construyendo-con-make)
- [Documentación](#documentación)
- [Problemas y soporte](#problemas-y-soporte)
- [Contribuyendo](#contribuyendo)  
- [Licencia y créditos](#licencia-y-créditos)   

### Instalando mediante el Gestor de Tarjetas

A partir de 1.6.4, Arduino permite la instalación de paquetes de plataformas de terceras personas utilizando el Gestor de Tarjetas. Hay paquetes disponibles para Windows, Mac OS y Linux (32 y 64 bit).

- Instalar la actual distribución del IDE Arduino versión 1.8 o superior. La versión actual se encuentra en [Arduino website](http://www.arduino.cc/en/main/software).
- Iniciar Arduino IDE y abrir la ventana Preferencias en el menú Archivo/Preferencias.
- Introducir ```http://arduino.esp8266.com/stable/package_esp8266com_index.json``` en la casilla *Gestor de URLs Adicionales de Tarjetas*. Puedes añadir múltiples URLs, separandolas con comas.
- Abre la ventana del Gestor de Tarjetas desde el menú "Herramientas/<Última tarjeta seleccionada>/Gestor de Tarjetas...", instala la plataforma *esp8266* (tras la instalación no olvides seleccionar tu tarjeta ESP8266 en el menú "Herramientas/<Última tarjeta seleccionada>).

#### Latest release [![Latest release](https://img.shields.io/github/release/esp8266/Arduino.svg)](https://github.com/esp8266/Arduino/releases/latest/)
URL para el Gestor de Tarjetas: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`

Documentación Inglés: [https://arduino-esp8266.readthedocs.io/en/2.4.0/](https://arduino-esp8266.readthedocs.io/en/2.4.0/)

Documentación Español: [http://esp8266-arduino-spanish.readthedocs.io/es/latest/index.html](http://esp8266-arduino-spanish.readthedocs.io/es/latest/index.html)

### Usando versión git
[![Linux build status](https://travis-ci.org/esp8266/Arduino.svg)](https://travis-ci.org/esp8266/Arduino)

- Instala el IDE Arduino 1.8.2 desde [Arduino website](http://www.arduino.cc/en/main/software).
- Ve al directorio de Arduino.
- Clona este repositorio en la carpeta hardware/esp8266com/esp8266 (o clónalo donde quieras y crea un enlace simbólico).
```bash
cd hardware
mkdir esp8266com
cd esp8266com
git clone https://github.com/esp8266/Arduino.git esp8266
```
- Descarga las herramientas binarias (necesitas Python 2.7).
```bash
cd esp8266/tools
python get.py
```
- Reinicia Arduino.

### Usando PlatformIO

[PlatformIO](http://platformio.org?utm_source=github&utm_medium=arduino-esp8266) es un ecosistema open source para desarrollo IoT con sistema multiplataforma, gestor de librerías y soporte completo para el desarrollo Espressif (ESP8266). Funciona con los sistemas operativos mas populares: macOS, Windows, Linux 32/64, Linux ARM (like Raspberry Pi, BeagleBone, CubieBoard).

- [Que es PlatformIO?](http://docs.platformio.org/en/latest/what-is-platformio.html?utm_source=github&utm_medium=arduino-esp8266)
- [PlatformIO IDE](http://platformio.org/platformio-ide?utm_source=github&utm_medium=arduino-esp8266)
- [PlatformIO Core](http://docs.platformio.org/en/latest/core.html?utm_source=github&utm_medium=arduino-esp8266) (Herramienta de linea de comandos)
- [Uso avanzado](http://docs.platformio.org/en/latest/platforms/espressif8266.html?utm_source=github&utm_medium=arduino-esp8266) - opciones personalizadas, uploading a SPIFFS, Over-the-Air (OTA), staging versión.
- [Integración con Cloud y Standalone IDEs](http://docs.platformio.org/en/latest/ide.html?utm_source=github&utm_medium=arduino-esp8266) - Cloud9, Codeanywhere, Eclipse Che (Codenvy), Atom, CLion, Eclipse, Emacs, NetBeans, Qt Creator, Sublime Text, VIM, Visual Studio y VSCode.
- [Ejemplos de proyectos](http://docs.platformio.org/en/latest/platforms/espressif8266.html?utm_source=github&utm_medium=arduino-esp8266#examples)

### Construyendo con make

[makeEspArduino](https://github.com/plerup/makeEspArduino) es un makefile generico para cualquier proyecto ESP8266 Arduino.
Su uso permite al Arduino IDE generar fácilmente compilaciones automatizadas y productivas.

### Documentación

Documentación para la última versión Inglés: https://arduino-esp8266.readthedocs.io/en/latest/
Documentación para la última versión Español: http://esp8266-arduino-spanish.readthedocs.io/es/latest/index.html

### Problemas y soporte ###

**Todos los "issue" y "pull request" deben realizarse en inglés, ya que es el idioma oficial del proyecto ESP8266/Arduino. No se atenderán peticiones en ningún otro idioma.**


[Foro de la comunidad ESP8266](http://www.esp8266.com/u/arduinoanswers) es una buena comunidad para preguntas y respuestas sobre Arduino para ESP8266.

Si encuentras útil este foro, por favor considera tu apoyo mediante una donación. <br />
[![Donate](https://img.shields.io/badge/paypal-donate-yellow.svg)](https://www.paypal.com/webscr?cmd=_s-xclick&hosted_button_id=4M56YCWV6PX66)

Si encuentras un problema "issue" que crees es un bug en el ESP8266 Arduino Core o sus librerías asociadas, eres bienvenido a enviarlo aquí en GitHub: https://github.com/esp8266/Arduino/issues.

Por favor, proporciona la mayor información posible:

- Versión de ESP8266 Arduino core que utilizas (puedes comprobarla en el Gestor de Tarjetas)
- El código de tu sketch; por favor enmarcalo dentro de un bloque de código, ver [Github markdown manual](https://help.github.com/articles/basic-writing-and-formatting-syntax/#quoting-code).
- Cuando encuentres un problema que se genera durante la ejecución, añade la salida del Serial. Enmarcaló dentro de un bloque de código, exactamente igual que el sketch.
- Para problemas que se generan durante la compilación, activa "Mostrar salida detallada mientas --> Compilación" en las preferencias del IDE, y añade esta salida (también dentro de un bloque de código).
- Modelo de tu tarjeta de desarrollo ESP8266.
- Opciones seleccionadas en el IDE (tarjeta seleccionada, tamaño flash, etc)

### Contribuyendo

**Todos los "issue" y "pull request" deben realizarse en inglés, ya que es el idioma oficial del proyecto ESP8266/Arduino. No se atenderán peticiones en ningún otro idioma.**

Para correcciones menores del código o la documentación, por favor da un paso adelante y envía un pull request.

Comprueba la lista de problemas que son fáciles de corregir — [Problemas fáciles para 2.5.0](https://github.com/esp8266/Arduino/issues?q=is%3Aopen+is%3Aissue+milestone%3A2.5.0+label%3A%22level%3A+easy%22). Trabajar en ellos es una buena forma de hacer que proyecto vaya hacia adelante.

Cambios mayores (reescribir partes del código existente desde cero, añadiendo nuevas funciones al "core", añadiendo nuevas librerias) deben (generalmente) ser discutidos abriendo un "Issue" primero.

Ramas "Branches" de caracteristicas con multitud de pequeñas peticiones "commits" (especialmente etiquetados como "oops", "fix typo", "forgot to add file", etc.) deben ser aplastadas antes de abrir un "pull request". A la vez, por favor absténgase de poner múltiples cambios no relacionados en un solo "pull request".

### Licencia y créditos ###

El IDE Arduino está desarrollado y mantenido por el equipo Arduino. El IDE está bajo licencia GPL.

El ESP8266 core incluye un toolchain xtensa gcc, que también está bajo licencia GPL.

Esptool escrito por Christian Klippel está bajo licencia GPLv2, actualmente mantenido por Ivan Grokhotkov: https://github.com/igrr/esptool-ck.

El SDK de Espressif incluido en esta versión está bajo licencia Espressif MIT.

Los ficheros del ESP8266 core están bajo licencia LGPL.

[Sistema de Archivos SPI Flash (SPIFFS)](https://github.com/pellepl/spiffs) escrito por Peter Andersson es utilizado en este proyecto. Está distribuido bajo licencia MIT.

[umm_malloc](https://github.com/rhempel/umm_malloc) librería de gestión de memoria está escrita por Ralph Hempel y es utilizada en este proyecto. Está distribuida bajo licencia MIT.

[axTLS](http://axtls.sourceforge.net/) librería escrita por Cameron Rich, construida desde https://github.com/igrr/axtls-8266, se utiliza en este proyecto. Está distribuida bajo [BSD license](https://github.com/igrr/axtls-8266/blob/master/LICENSE).
