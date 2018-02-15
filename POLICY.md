Este documento describe las reglas a seguir en este repositorio, destinado a todo aquel que contribuya en el seguidor de "issues" y en los "Pull request".

**Todos los "issue" y "pull request" deben realizarse en inglés, ya que es el idioma oficial de este proyecto. No se atenderán peticiones en ningún otro idioma.**
# Abriendo nuevos Issues
1.   El seguidor de Issues es precisamente eso: una herramienta para seguir los issues en el código del core y no un foro de discusión. Abrir un "issue" significa que se ha encontrado un problema en el código del core y debe ser abordado por un "Contributor".
2.   Cuando se abre un issue, se muestra una plantilla con campos para completar. La información requerida es importante. Si se ignora la plantilla, o no se facilita suficiente información, el issue puede ser cerrado debido a falta de información. Ejemplo:
      * Error usando WifiMulti y FS. ¿Por qué? (Sin información básica, sin configuraciones del IDE, no facilita el sketch)
3.   Preguntas del tipo "Como..." o "Podéis ayudarme con..." o "Puede el ESP hacer..." no serán atendidos. Este tipo de preguntas deben ser dirigidas a un foro de discusión, como esp8266.com o stackoverflow. Todos los issues de este tipo serán cerrados con una simple referencia a este documento (versión en inglés). Ejemplo:
    * ¿Como conecto al WiFi?.
    * ¿Como conecto dos ESP?.
    * ¿Puedo mandar http a una red pública?
    * Mi esquema/proyecto/código no funciona, ¡Ayuda!
4.   Issues con errores del usuario, errores del lenguaje de programación, faltos de conocimientos o experiencia con el uso semántico de las librerías del core, o similares, serán cerrados con una referencia al presente documento (versión en ingles). Ejemplos: 
    *    Mi sketch no funciona por un char[] que no es "null terminated".
    *    Intentando utilizar yield/delay, o librerías que utilizan yield/delay, en una llamada asíncrona. 
    *    Utilizar new/malloc sin antes borrar/liberar (pérdida de memoria).
5.   Issues sobre cuestiones ya aclaradas en la documentación serán cerradas de igual manera. Ejemplo:
    *    No puedo flasear por un fallo en espcomm.
6.   Los issues deben contener un sketch lo mas reducido posible. Issues con un sketch incompleto, o muy grandes, serán cerrados. Se debe poner el máximo esfuerzo por la persona que abre el issue para reducirlo al código mas relevante que sea capaz de reproducir el problema, para que se pueda hacer la investigación. Es obligatorio un ejemplo mínimo, completo y verificable.
7.   Los issues relativos a PRs aún no mezclados serán cerrados. Si hay issue con un PR, la explicación debe hacerse al PR mismo.
8.   Issues acompañados ya de una investigación que muestre la raíz del problema serán tratados con prioridad.
9.   Issues duplicados serán cerrados con una referencia al original.

# Triaje (Clasificación)
1.    Cualquier "Contributor" del proyecto puede participar en el proceso de triaje, si así lo desea.
2.    Un issue que necesita cerrarse por no cumplir estas reglas o por otra razón, puede ser cerrado por un "Contributor".
3.    Los issues aceptados deben ser marcados con la etiqueta apropiada, p.ej.: component: xyz
4.    Si se considera que un issue requiere conocimiento especializado (p.ej.: TLS, HTTP parser, integración SDK, etc), el/los "Contributor/s" relevante/s al código afectado pueden ser etiquetado, llamando de esta forma su atención.
5.    Los issues severos deben ser añadidos al siguiente hito (p.ej.: la próxima versión que sea lanzada), o al siguiente hito. Está bien rechazar problemas dependiendo de los recursos disponibles.
6.    Los issues que pueden afectar a la funcionalidad de muchos usuarios deben considerarse severos. 
7.    Los issues causados por el SDK o el chip no deben marcarse como severos, porque normalmente no se puede hacer mucho mas. Debe aplicarse el sentido común cuando se decide. Tales problemas deberían documentarse en un KID (documento de problemas conocidos), posiblemente en el Wiki, para que los usuarios lo puedan consultar. Ejemplo:
    * ARP issue
    * Extra channel change beacon announced by the SoftAP
    * Error de Wakeup ROM en el chip ESP
8.    Los issues con solicitúd de características deben analizarse para determinar su viabilidad / conveniencia. Ejemplo:
    * Soporte para nueva tarjeta. Si la nueva tarjeta no es ampliamente usada, no existe página web del fabricante, etc, entonces no es conveniente soportarla. Si la nueva tarjeta es esencialmente un duplicado de otra, no es conveniente duplicar la existente.
9.    La solicitud de funcionalidades o cambios destinados a casos de uso muy específicos/limitados, especialmente si aumentan la complejidad del código, deben ser denegados, o debe requerirse ser rediseñado, generalizado o simplificado.
10.    La solicitud de funcionalidades no acompañadas de un PR:
    * pueden ser cerradas inmediatamente (denegadas)
    * pueden ser cerradas tras un periodo de tiempo predeterminado (dando tiempo a algún candidato a hacerse cargo)
    * podría ser considerado suficientemente interesante para elaborarlo, pero sin un proyecto u objetivo específico y por lo tanto acumulado en una lista de solicitudes de funcionalidades a largo plazo. Dichas solicitudes de funcionalidades en general no se seleccionarán para una fecha límite o lanzamiento.
11.    En algunos casos, se solicitará una retroalimentación por parte de quien creó el issue, ya sea información adicional para clarificar, pruebas adicionales o otros. Si no hay retroalimentación tras 30 dias, el issue debe ser cerrado por un "Contributor".

# Compatibilidad
1.    La compatibilidad con el sistema Arduino es la primera prioridad. La compatibilidad con PlatformIO y make también es mantenida, pero es la segunda prioridad.
2.    La solicitud de funcionalidades debe ser compatible con Arduino.
    *    APIs especificas de ESP deben ser añadidas con cuidado y debe ser marcado como tal (Ejemplo: ESP8266WiFi)
    *    APIs de librerías comunes deben mantener la compatibilidad (Ejemplo: Wire, SPI, Servo)
    *    Extensiones especificas de ESP para APIs compatibles son aceptadas, especialmente si son requeridas para un uso completo de ciertos periféricos, pero dichas funciones deben ser claramente marcadas como especificas de ESP.
3.    Cuando se hagan cambios que afecten a PlatformIO o make, las personas relevantes deben ser notificadas. Compruebe si se necesitan cambios en el sistema de compilación. Cuando un issue acerca de este sistema de compilación se reporta, redirigir al creador del issue al respectivo seguidor de issues.
4.    Las librerías del core se implementan como una capa contenedora sobre el SDK de Espressif. Debido a los requisitos y las limitaciones impuestas por el SDK, existen diferencias inherentes entre el comportamiento de este núcleo y el núcleo estándar de Arduino. (Ejemplo: usar delay()s largos no están permitidos aquí). La compatibilidad no puede ser mantenida en algunos casos y las diferencias deben ser claramente documentadas.

# Pull requests
1.    Todos los pull requests deben someterse a revisión por al menos un "Contributor" que no sea el creador.
2.    Todos los pull requests deben considerar actualizaciones a la documentación.
3.    Todos los pull requests deben considerar actualizaciones a test de regresión, donde sea posible.
4.    Los pull requests que abordan un problema pendiente, en particular un problema que se considera grave, deben tener prioridad.
5.    Si un PR se acepta, luego debería ser revisado y actualizado según los comentarios proporcionados, y entonces fusionarse.
6.  Los pull requests que no cumplan con lo anterior serán denegados y cerrados.

# Otros
Se debe mantener una tabla para relacionar mantenedores y componentes. Al hacer el triaje, esto es esencial para determinar si se debe consultar a alguien en particular sobre cambios específicos.

Se debe establecer una cadencia de versiones estables, por ejemplo, cada 6 meses.

Las pruebas de regresión deben revisarse y simplificarse con el proceso de publicación. Se deben realizar pruebas de regresión antes de fusionar un PR para reducir la sobrecarga de un lanzamiento.
