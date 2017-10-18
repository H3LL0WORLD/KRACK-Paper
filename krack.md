# Ataques de reinstalación de claves: Forzar la reutilización de Nonce en WPA2

- Mathy Vanhoef
- imec-DistriNet, KU Leuven
- Mathy.Vanhoef@cs.kuleuven.be

## ABSTRACTO
Presentamos el ataque de reinstalación de clave. Este ataque abusa de fallas
en el diseño o la implementación de protocolos criptográficos para reinstalar una
clave que ya está en uso. Esto restablece los parámetros asociados a la clave como
nonces de transmición y los contadores de repetición recibidos. Varios tipos de
handshakes de Wi-Fi criptográficos se ven afectados por el ataque.

Todas las redes Wi-Fi protegidas usan el 4-way handshake para generar
una nueva clave de sesión. Hasta ahora, este handshake de 14 años
se ha mantenido libre de ataques, e incluso se ha demostrado que es seguro.
Sin embargo, mostramos que el 4-way handshake es vulnerable a un ataque de
reinstalación de clave. Aquí, el atacante engaña a una víctima para que reinstale una clave ya en uso.
Esto se logra manipulando y repitiendo mensajes del handshake.
Al reinstalar la clave, los parámetros asociados tales como el número
de paquete de transmisión incremental (nonce) y el número de paquetes recibidos
(contador de repetición) son restablecidos a su valor inicial.
Nuestro ataque de reinstalación de clave también rompe los handshake PeerKey,
clave de grupo ([Group key](https://en.wikipedia.org/wiki/IEEE_802.11i-2004#The_group_key_handshake)) y 
Transición rápida del servicio de conjunto básico ([Fast Basic Set Service Transition](https://en.wikipedia.org/wiki/IEEE_802.11r-2008)).
El impacto depende del handshake siendo atacado, y el protocolo de confidencialidad de datos usado.
Simplificando, contra AES-CCMP un atacante puede repetir y descifrar (pero no falsificar) paquetes.
Esto hace posible secuestrar transmisiones TCP e inyectar datos maliciosos en ellas.
Contra WPA-TKIP y GCMP el impacto es catastrófico: los paquetes se pueden repetir, descifrar y falsificar.
Porque GCMP usa la misma clave de autenticación en ambas direcciones de la comunicación, se ve especialmente afectada.
Finalmente, confirmamos nuestros hallazgos en la práctica y descubrimos que todos los dispositivos Wi-Fi
son vulnerables a alguna variante de nuestros ataques.
Cabe destacar que nuestro ataque es excepcionalmente devastador contra Android 6.0:
obliga al cliente a usar una clave de cifrado predecible de sólo ceros.

## PALABRAS CLAVE
Protocolos de seguridad; seguridad de redes; ataques; reinstalación de clave;
WPA2; reutilización del nonce; handshake(apretón de manos); número de paquete; vector de inicialización

## 1. INTRODUCCIÓN
Todas las redes Wi-Fi seguras están protegidas con alguna versión de
Wi-Fi de Acceso protegido (WPA/2). Además, hoy en día incluso los hotspots públicos
pueden usar el cifrado autenticado gracias al programa Hotspot 2.0.
Todas estas tecnologías dependen del 4-way handshake definido en la enmienda 802.11i de 802.11.
En este trabajo, presentamos fallas en el diseño del 4-way handshake, y en handshakes relacionados.
Porque apuntamos a estos handshakes, tanto los productos WPA como los WPA2-certificados se ven afectados por nuestros ataques.

El 4-way handshake proveé autenticación mutua, y acuerdo de clave se sesión.
Junto con (AES)-CCMP un protocolo de integridad y confidencialidad de datos.
Desde su primera introducción en 2003, bajo el nombre WPA, esta parte fundamental
de la enmienda 802.11i ha permanecido libre de ataques.
En efecto, las unicas vulnerabilidades conocidas actualmente están en (WPA-)TKIP.
Este protocolo de confidencialidad de datos fue diseñado como una solución a corto término para el roto protocolo WEP.
En otras palabras, TKIP nunca fue pensado para ser una solución a largo plazo segura.
Ademas, mientras varios ataques contra redes Wi-Fi protegidas fueron descubiertos durante los últimos años,
estos no explotan fallas en el protocolo 802.11i, en cambio, explotan fallas en WPS (Wi-Fi Protected Setup),
controladores defectuosos, generadores de números aleatorios defectuosos, claves compartidas predecibles,
autenticación empresarial insegura, y demás.
Que no haya sido encontrada tal mayor vulnerabilidad en CCMP y el 4-way handshake, no es sorprendente.
Despues de todo, ambos han sido formalmente provados como seguros.
Con esto en mente, uno podría razonablemente asumir que el diseño del 4-way handshake es en efecto seguro.

A pesar de su historia y pruebas de seguridad, demostramos que
el 4-way handshake es vulnerable a los ataques de reinstalación de claves.
Además, descubrimos debilidades similares en otros handshakes de Wi-Fi.
Es decir, también atacamos el handshake de PeerKey, el handshake del group key y el handshake del Fast BSS Transition (FT).

La idea detrás de nuestros ataques es bastante trivial en retrospectiva, y puede
ser resumida de la siguiente manera. Cuando un cliente se une a una red, ejecuta
el 4-way handshake para negociar una nueva clave de sesión.
Se instalará esta clave después de recibir el 3er mensaje del handshake.
Una vez que la clave es instalada, será usada para cifrar tramas de datos normales
usando un protocolo de confidencialidad de datos
Sin embargo, como los mensajes se pueden perder o caer, el Punto de acceso (AP)
retransmitirá el mensaje 3 si no recibió una respuesta apropiada como reconocimiento.
Como resultado, el cliente puede recibir el mensaje 3 varias veces.
Cada vez que reciba este mensaje, volverá a instalar la misma clave de sesión;
restableciendo así el número de paquetes de transmisión incremental (nonce) y
el contador de repetición recibido utilizado por el protocolo de confidencialidad de datos.
Mostramos que un atacante puede forzar estos restablecimientos de nonce recolectando
y reproduciendo retransmisiones del mensaje 3.
Al forzar la reutilización de nonce de esta manera, el protocolo de confidencialidad de datos puede ser atacado,
por ejemplo, los paquetes se pueden reproducir, descifrar y/o falsificar.
La mismo técnica se usa para atacar el handshake del group key, PeerKey y Fast BSS Transition.

Cuando se ataca el handshake del Fast BSS Transition,
El impacto preciso depende del protocolo de confidencialidad de datos usado.
Si se usa CCMP, pueden descifrarse paquetes arbitrarios.
A su vez, esto se puede usar para descifrar paquetes TCP SYN y secuestrar
conexiones TCP. Por ejemplo, un atacante puede inyectar contenido malicioso en conexiones HTTP no cifradas.
Si se usa TKIP o GCMP, un atacante puede descifrar e inyectar paquetes arbitrarios.
Aunque GCMP es una adición relativamente nueva a Wi-Fi. Se espera que sea adoptado a un ritmo eleveado en los proximos años.
Finalmente, cuando el handshake del group key es atacado, un atacante puede repetir tramas direccionadas por grupo,
es decir, tramas de difusión y multidifusión.

Nuestro ataque es especialmente devastador contra la versión 2.4 y 2.5 de wpa_supplicant,
un cliente Wi-Fi comúnmente utilizado en Linux.
Aquí el el cliente instalará una clave de cifrado de totalmente ceros en lugar de reinstalar
la verdadera clave. Esta vulnerabilidad parece ser causada por una observación
en el estándar 802.11 que sugiere limpiar partes de la clave de sesión
de la memoria una vez que se ha instalado.
Ya que Android usa un wpa_supplicant modificado, Android 6.0 y Android Wear 2.0 también contienen esta vulnerabilidad.
Como resultado, actualmente el 31.2% de los dispositivos Android son vulnerables
a esta excepcionalmente devastadora variante de nuestro ataque.

Curiosamente, nuestros ataques no violan las propiedades de seguridad
probadas en análisis formales del 4-way handshake y el group key handshake.
En particular, estas pruebas indican que la clave de sesión negociada
permanece privada y que la identidad tanto del cliente como del punto acceso (AP) se confirma.
Nuestros ataques no filtran la clave de sesión.
Además, aunque las tramas de datos normales se pueden falsificar si
se usa TKIP o GCMP, un atacante no puede falsificar mensajes EAPOL
y por lo tanto no puede hacerse pasar por el cliente o el AP durante (despues) del handshake.
En cambio, el problema es que las pruebas no planean la instalación de la clave.
Dicho de otra manera, sus modelos no indican cuándo se debe instalar la clave negociada.
En la práctica, esto significa que la mismo clave se puede instalar varias veces,
restableciendo así los nonces y contadores de repetición utilizados por el protocolo de confidencialidad de datos.

Para resumir, nuestras principales contribuciones son:
- Presentamos los ataques de reinstalación de claves.
Aquí, un atacante fuerza la reinstalacion de una clave en uso, restaurando así cualquier nonce y/o contador de repetición asociado.
- Mostramos que los handshake 4-way, PeerKey, group key y Fast BSS Transition son vulnerables a ataques de reinstalación de claves.
- Diseñamos técnicas de ataque para llevar a cabo nuestros ataques en la práctica.
Esto demuestra que todas las implementaciones son vulnerables a alguna variante de nuestro ataque.
- Evaluamos el impacto práctico de la reutilización del nonce para todos los protocolo de confidencialidad de datos 802.11.

El resto de este paper se estructura de la siguiente manera.
La sección 2 presenta aspectos relevantes del estandar 802.11.
Nuestro ataque de reinstalación de claves es ilustrado contra los handshake 4-way y PeerKey en la sección 3,
contra el handshake Group Key en la sección 4, y contra el handshake Fast BSS Transition en la sección 5.
En la sección 6 evaluamos el impacto de nuetros ataques, las contramedidas actuales,
explicamos donde fallaron las pruebas y discutimos las lecciones aprendidas.
Finalmente presentamos trabajo relacionado en la sección 7 y concluimos en la sección 8.

## 2. BACKGROUND
En esta sección presentamos la enmienda 802.11i,
diversos mensajes y handshakes usados cuando se conecta a una red Wi-Fi,
y los protocolos de integridad y confidencialidad de datos de 802.11.

## 2.1 La Enmienda 802.11i
Despues de que investigadores mostraran que el (Privacidad Cableada Equivalente | Wired Equivalent Privacy) (WEP)
estaba fundamentalmente roto, el IEEE ofrecio una solución más robusta en la enmienda 802.11i de 802.11.
Esta enmienda define el 4-way handshake (ver Sección 2.3), y el protocolo de integridad y confidencialidad
de datos denominados (WPA-)TKIP y (AES-)CCMP (ver Sección 2.4).
Mientras que la enmienda 802.11i estaba en desarrollo la Wi-Fi Alliance
comenzó a certificar dispositivos basados en la version borrador D3.0 de 802.11i.
Este programa de certificación fue llamado Wi-Fi Acceso Protegido (WPA).
Una vez la version final D9.0 de 802.11i fue ratificada,
la certificación WPA2 fue creada basada en esta version oficialmente ratificada.
Ya que WPA y WPA2 estan basadas en 802.11i, son técnicamente indenticas.
La principal diferencia es que WPA2 exige compatibilidad para lo más seguro (CCMP), y opcionalmente permite TKIP,
mientras que lo contrario es cierto para WPA.
La funcionalidad requerida por WPA y WPA, y usada por todas las redes Wi-Fi protegidas, es el 4-way handshake.
Incluso las redes empresariales dependen del 4-way handshake.
Por lo tanto, todas las redes Wi-Fi protegidas son afectadas por nuestros ataques.
El 4-way handshake, GroupKey handshake, y el protocolo CCMP, han sido formalmente analizados y demostrado ser seguros.

## 2.2 Autenticación y Asociación
Cuando un cliente quiere conectarse a una red Wi-Fi, comienza con (mutua) autenticación y asociación con el AP.
En la figura 2 se ilustra esto en la fase de asociacion del handshake.
Sin embargo, cuando se conecta por primera vez a una red, ninguna autenticación real toma lugar en esta fase.
En cambio, se usa un (Sistema Abierto | Open System) de autentiación, el cual permite que cualquier cliente se autentique.
La autenticación real será realizada durante el 4-way handshake.
La autenticación real solamente sucede en esta fase cuando hay roaming
entre dos AP de la misma red utilizando el handshake Fast BSS Transition (ver Sección 3).
Despues de la autenticación (abierta), el cliente se asocia con la red,
esto se hace enviando una petición de asociación al AP.
Este mensaje contiene los conjuntos de pares y grupos de cifrado que el cliente desea usar.
El AP responde con una respuesta de asociación, informando al cliente si la asociación fue exitosa o no.

## 2.3 El 4-way Handshake
El 4-way handshake proporciona autenticación mutua basada en un
secreto compartido llamado (Clave Maestra Pareja | Pairwise Master Key) (PMK), y negocia
una nueva clave de sesión llamada (Clave Transitoria Pareja | Pairwise Transient Key) (PTK). Durante
este handshake, el cliente es llamado el suplicante, y el AP es
llamado el authnticador (usamos estos términos como sinónimos). El
PMK es derivado de una contraseña pre-compartida en una red personal,
y es negociada usando una fase de autenticacion 802.1x en una red empresarial (ver Figura 2).
El PTK es derivado del PMK, el (Nonce del Autenticador | Authenticator Nonce) (ANonce), (Nonce del Suplicante | Supplicant Nonce) (SNonce), y
la dirección MAC de tanto el suplicante como el autenticador. Una vez
generada, la PTK es dividida en una  (Clave de Confirmación Clave | Key Confirmation Key) (KCK),
(Clave de Encriptación Clave | Key Encryption Key) (KEK), Y (Clave Temporal | Temporal Key) (TK). La KCK y KEK
son usadas para proteger los mensajes del handshake, mientras que la TK es usada para
proteger tramas de datos normales con un protocolo de confidencialidad de datis.
Si se usa WPA2, el 4-way handshake también transporta la actual
(Clave Temporal Grupal | Group Temporal Key) (GTK) para el suplicante.
```
+------------+------------------------+-------+.....+-----+.....+---------+-----------+----------+
| encabezado | contador de repeticion | nonce |     | RSC |     |   MIC   |    Datos de Clave    |
+------------+------------------------+-------+.....+-----+.....+---------+-----------+----------+
<---------------------------------------------------><--------><---------------->
                   82 bytes                           variable  encrypted
```
##### Figura 1: Diseño simplificado de una trama EAPOL.

Cada mensaje en el 4-way handshake se define utilizando tramas EAPOL.
Discutiremos brevemente el diseño y los campos más importantes
de estas tramas (ver Figura 1).
Primero, el encabezado define qué mensaje representa una trama EAPOL en particular en el handshake.
Usaremos el mensaje de notación n y un MsgN para referirnos al mensaje n-th de un 4-way handshake

El campo del contador de repetición se usa para detectar tramas repetidas.
El autenticador incrementa siempre el contador de repetición despues de trasmitir una trama.
Cuando el suplicante responde a una trama EAPOL del autenticador,
este usa el mismo contador de repetición como al del que la trama EAPOL le  está respondiendo.
El campo nonce transporta los nones aleatorios que el suplicante
y el autenticador generan para derivar una nueva clave de sesión.
Despues, en caso de que la trama EAPOL transmita una (Clave Grupal | Group Key),
el (Contador de Secuencia de Recepción | Receive Sequence Counter) (RSC) contiene 
el número del paquete inicial de esta clave.
La clave grupal se almacenan en el campo Datos de Clave, que está encriptado usando la KEK.
Finalmente, la autenticidad de la trama es protegida usando la KCK con una (Verificación de Integridad de Mensajes | Message Integriti Check) (MIC).
