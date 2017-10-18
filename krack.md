# Ataques de reinstalación de claves: Forzar la reutilización del Nonce en WPA2

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
este handshake, el cliente es llamado el solicitante, y el AP es
llamado el autenticador (usamos estos términos como sinónimos). El
PMK es derivado de una contraseña pre-compartida en una red personal,
y es negociada usando una fase de autenticacion 802.1x en una red empresarial (ver Figura 2).
El PTK es derivado del PMK, el (Nonce del Autenticador | Authenticator Nonce) (ANonce), (Nonce del solicitante | Supplicant Nonce) (SNonce), y
la dirección MAC de tanto el solicitante como el autenticador. Una vez
generada, la PTK es dividida en una  (Clave de Confirmación Clave | Key Confirmation Key) (KCK),
(Clave de Encriptación Clave | Key Encryption Key) (KEK), Y (Clave Temporal | Temporal Key) (TK). La KCK y KEK
son usadas para proteger los mensajes del handshake, mientras que la TK es usada para
proteger tramas de datos normales con un protocolo de confidencialidad de datis.
Si se usa WPA2, el 4-way handshake también transporta la actual
(Clave Temporal Grupal | Group Temporal Key) (GTK) para el solicitante.
```
+------------+------------------------+-------+.....+-----+.....+---------+-----------+----------+
| encabezado | contador de repeticion | nonce |     | RSC |     |   MIC   |    Datos de Clave    |
+------------+------------------------+-------+.....+-----+.....+---------+-----------+----------+
<---------------------------------------------------------------><--------><---------------------->
                   82 bytes                                       variable        cifrado
```
##### Figura 1: Diseño simplificado de una trama EAPOL.

Cada mensaje en el 4-way handshake se define utilizando tramas EAPOL.
Discutiremos brevemente el diseño y los campos más importantes
de estas tramas (ver Figura 1).
Primero, el encabezado define qué mensaje representa una trama EAPOL en particular en el handshake.
Usaremos la notación ***mensaje n*** y  ***MsgN*** para referirnos al mensaje n-th de un 4-way handshake
El campo del contador de repetición se usa para detectar tramas repetidas.
El autenticador incrementa siempre el contador de repetición despues de trasmitir una trama.
Cuando el solicitante responde a una trama EAPOL del autenticador,
usa el mismo contador de repetición de la trama EAPOL a la qué está respondiendo.
El campo nonce transporta los nonces aleatorios que el solicitante
y el autenticador generan para derivar una clave de sesión nueva.
Despues, en caso de que la trama EAPOL transmita una (Clave Grupal | Group Key),
el (Contador de Secuencia de Recepción | Receive Sequence Counter) (RSC) contiene 
el número del paquete inicial de esta clave.
La clave grupal se almacena en el campo Datos de Clave, que está cifrado usando
la KEK. Finalmente, la autenticidad de la trama es protegida usando la KCK con
una (Verificación de Integridad de Mensaje | Message Integriti Check) (MIC).

La figura 2 ilustra los mensajes que son intercambiados durante el 4-way
handshake. En est, usamos la siguiente notación:
```
MsgN(r, Nonce; GTK)
```
Esto representa el mensaje N de el 4-way handshake, teniendo un contador de
repetición de ****r****, y con el dado nonce (si está presente). Todos los
parametros despues del punto y coma se almacenan en el campo datos de clave,
y  por lo tanto son cifrados usando la KEK (recuerde la Figura 1).

El autenticador inicia el 4-way handshake enviando el mensaje 1. Este contiene
el ANonce, y es el unico mensaje EAPOL que no está protegido por un MIC.
Al recibir este mensaje, el solicitante genera el SNonce y derica la PTK
(es decir, la clave de sesión). El solicitante entonces envía el SNonce al
autenticador en el mensaje 2. Una vez que el autenticador se aprende el
SNonce, tambien deriva la PTK, y envía la clave de grupo (GTK) al solicitante.
Finalmente, para finalizar el handshake, el solicitante responde con el
mensaje 4 y después de eso instala la PTK y GTK. Después de recibir este
mensaje, el autenticador tambien instala la PTK y GTK (la GTK es instalada
cuando se inicia el AP). Resumiendo, los primeros dos mensajes son usados
para transportar nonces, y los dos ultimos son usados para transportar la
clave de grupo y protegerla de ataques de degradación (downgrade).

Tenga en cuenta que en una conexion existente, la PTK puede ser renovada
iniciando un nuevo 4-way handshake. Durante esta renovación, todos los mensajes
del 4-way handshake son encriptados por el protocolo de confidencialidad
de datos usando la PTK actual (Dependemos de esto en la Sección 3.4)

```
+-----------------------+                    +-----------------------+
| Solicitante (cliente) |                    |   Autenticador (AP)   |
+-----------------------+                    +-----------------------+
```
###### Fase de Asociación
```
  |                    Petición de Autenticación                   |
  |--------------------------------------------------------------->|
  |                    Respuesta de Autenticación                  |
  |<---------------------------------------------------------------|
  |                    Petición de (Re)Asociación                  |
  |--------------------------------------------------------------->|
  |                    Respuesta de (Re)Asociación                 |
  |<---------------------------------------------------------------|
  |                                                                |
  |<---------------- Autenticación 802.1x opcional --------------->|
  |                                                                |
```
###### 4-way Handshake
```
  |                          Msg1(r, Anonce)                       |
  |<---------------------------------------------------------------|
 +-------------------+                                             |
 | Derivación de PTK |                                             |
 +-------------------+                                             |
  |                          Msg2(r, SNonce)                       |
  |--------------------------------------------------------------->|
  |                                             +-------------------+
  |                                             | Derivación de PTK |
  |                                             +-------------------+
  |                           Msg3(r+1; GTK)                       |
  |<---------------------------------------------------------------|
  |                             Msg4(r+1)                          |
  |--------------------------------------------------------------->|
 +--------------------+                              +--------------+ 
 | Instalar PTK y GTK |                              | Instalar PTK |
 +--------------------+                              +--------------+
  |                                                                |
  |<--- Ahora se pueden intercambiar tramas de datos cifradas  --->|
  |                                                   +-------------+
  |                                                   | Renovar GTK |
  |                                                   +-------------+
```
###### Group Key Handshake
```
  |                   EncXptk {Grupo1(r+2; GTK)}                   |
  |<---------------------------------------------------------------|
  |                      EncYptk {Grupo2(r+2)}                     |
  |--------------------------------------------------------------->|
  |                                                                |
 +--------------+                                    +--------------+
 | Instalar GTK |                                    | Instalar GTK |
 +--------------+                                    +--------------+
```

##### Figura 2: Mensajes intercambiados cuando un solicitante (cliente) se conecta con un autenticador (AP), realiza el 4-way handshake, y ejecuta periódicamente el group key handshake

## 2.4 Protocolos de Integridad y Confidencialidad
La enmienda 802.11i define dos protocolos de confidencialidad de datos.
El primero se denomina Protocolo de Integridad de Clave (Temporal Key Integrity Protocol | TKIP). Sin embargo,
hoy en día TKIP está descontinuado debido a problemas de seguridad.
El segundo protocolo es comúnmente denominado (AES-)CCMP, y es actualmente
el protocolo de confidencialidad de datos más utilizado. En
2012, la enmienda 802.11ad añadio un nuevo protocolo de confidencialidad de datos
llamado el Protocolo Modo Galios/Contador (Galios/Counter Mode Protocol | GCMP). Esta
enmienda tambien añade soporte para comunicaciones de corto rango en la banda de 60 GHz,
la cual requiere un cifrado rápido tal como GCM. En este momento, 802.11ad está siendo
expandido bajo el nombre Wireless Gigabit (WiGig), y se espera que sea adoptado a una
alta velocidad en los proximos años. Finalmente, la enmienda 802.11ac extiende más GCMP
añadiendo soporte para claves de 256-bits.

Cuando se usa TKIP, la parte Clave Temporal (Temporal Key | TK) de la clave se sesión
(PTK) es dividida en una clave de cifrado de 128-bits, y dos claves de Verificación de
la Integridad del Mensaje (Message Integrity Check | MIC) de 64-bits. La primera clave
MIC se usa para la comunicación AP-a-cliente, mientras que la segunda se usa en dirección
contraria. Se usa RC4 para cifrar, con una unica clave por-paquete que es una mezcla
de la clave de cifrado de 128-bits, la dirección MAC del remitente, y un nonce incremental
de 48-bits. Este nonce es incrementado despues de de transmitir una trama, usado como contador
de repetición por el receptor, e inicializado a 1 cuando se instala la TK. La autenticidad
del mensaje es proporcionada por el algoritmo Michael. Desafortunadamente, Michael es trivial
de invertir: dados datos en texto plano y su valor MIC, se puede recuperar eficientemente la clave MIC.


El protocolo CCMP se basa en el cifrado AES que opera en modo CCMP (modo contador con CBC-MAC).
Es un algoritmo de Cifrado Autenticado con Datos Asociados (Authenticated Encryption with
Associated Data | AEAD), y seguro mientras no se repita ningún vector de inicialización (IV)
bajo una clave en particular. En CCMP el IV es la concatenación de la dirección MAC del remitente,
una nonce de 48-bits, y algunas banderas adicionales derivadas de la trama transmitida. El nonce
tambien es usado como un contador de repetición por el receptor, incrementado por uno antes de ser
enviada cada trama e inicializado a 1 cuando se instala la TK. Este se supone que debe garantizar
que los IVs no se repitan. Adicionalmente, esta implementación permite que la TK sea usada directamente
como la clave para ambas direcciones de la comunicación

El protocolo GCMP se basa en AES-GCM, lo que significa que usa el Modo Contador para el cifrado,
con el texto cifrado resultante siendo autenticado usando la función GHASH. Similar a CCMP, es un
cifrado AEAD, y es seguro mientras que el IV no se repita bajo una clave en particular. En GCMP,
el IV es la concatenación de la dirección MAC del remitente y un nonce de 48-bits. El nonce es
usado también como contador de repetición por el receptor, incrementado por uno antes de enviar
cada trama, e inicializado a 0 cuando se instala la TK. Esto normalmente asegura que cada IV es
usado solamente una vez. Como con CCMP, la TK es usada directamente como la clave para ambas
direcciones de la comunicación. Si el nonce es repetido alguna vez, es posible reconstruir la clave
de autenticacion utilizada por la función GHASH.

Para indicar que una trama está cifrada y autenticada usando un protocolo de confidencialidad
de datos, utilizamos la siguiente notación:
```
   n
Enc {.}
   k
```
Aquí ***n*** indica el nonce siendo usado (y, por lo tanto, tambien el contador de repetición).
El parametro ***k*** indica la clave, la cual es la PTK (clave de sesión) para tráfico de uni-difusión.
Para tráfico dirigido por grupo, es decir, tramas de difusión y multi-difusión, esta es la GTK
(clave de grupo). Finalmente, las dos notaciones
```
Datos(payload)
DatosDeGrupo(payload)
```
son usadas para representar una trama uni-difusión ordinaria o datos dirigidos por grupo,
respectivamente, con el payload dado.


## 2.5 El Group Key handshake
El autenticador renueva la clave de grupo periódicamente, y distribuye esta nuev clave de
grupo a todos los clientes usando el group key handshake. Este handshake fue provado ser seguro en [39],
y es ilustrado en la ultima fase de la Figura 2. El autenticador inicia el handshake enviando un mensaje
de grupo 1 a todos los clientes. El solicitante reconoce la recepción de la nueva clave de grupo
respondiendo con un mensaje de grupo 2. Dependiendo de la implementación, el autenticador instala
la nueva GTK bien sea, antes de enviar el mensaje de grupo 1, o despues de recibir una respuesta
de todos los clientes conectados (ver Sección 4).
Finalmente, el mensaje de grupo 1 tambien contiene el Contador de Repetición de Recepción actual de la
clave de grupo en el campo RSC (ver Figura 1).

Ambos mensajes en el grop key handshake son definidos usando tramas EAPOL, y son representados como Grupo1
y Grupo2 en la Figura 2.
Tenga en cuenta que el mensaje de grupo 1 almacena la nueva clave de grupo en el campo Datos de Clave,
y por lo tanto es cifrada usando la KEK (recordar la Figura 1). Desde este punto una PTK es instalada,
la trama EAPOL completa es protegida usando un protocolo de confidencialidad de datos.
Finalmente, si un cliente transmite una trama de difusión o multi-difusión, la envía primero como
una trama uni-difusión al AP. El AP entonces encripta la trama usando la clave de grupo, y la transmite
a todos los clientes. Esto asegura que todos los clientes dentro del rango del AP reciban la trama.

# 3 ATACANDO EL 4-WAY HANDSHAKE
En esta sección mostramos que el mecánismo de declaración detrás del 4-way handshake es vulnerable a un ataque de reinstalación de claves.
Entonces demostramos cómo ejecutar este ataque en entornos de la vida real.

## 3.1 Mecánismo de Declaración del Solicitante
La enmienda 802.11i no contiene un mecánismo de declaración formal describiendo cómo el solicitante
debe implementar el 4-way handshake. En cambio, sólamente proporciona pseudo-código que describe cómo,
pero no cuando, ciertos mensajes del handshake deben ser procesados.
Afortunadamente, 802.11r extiende el 4-way handshake ligeramente, y lo hace proporcionando
una detallado mecánismode declaración del solicitante.
La Figura 2 contiene una versión simplificada de este mecánismo de declaración.

Cuando se conecta por primera vez a una red e inicia el 4-way handshake, el solicitante transita al
mecánismo PTK-INIT (ver Figura 3).
Aquí, inicializa la PMK (Pairwise Master Key). Cuando  recibe el mensaje 1 transita a la fase PTK-START.
Esto podría suceder cuando se conecta a una red por primera vez, o cuando la clave de sesión ha sido renovada
después de una previo handshake (completado). Al ingresar a PTK-START, el solicitante genera un SNonce
aleatorio, calcula la PTK Temporal (TPTK), y envía su SNonce al autenticador usando el mensaje 2. El autenticador
responderá con el mensaje 3, el cual es aceptado por el solicitante si el MIC y el contador de repetición son validos.
De ser así, se mueve al mecánismo PTK-NEGOTIATING, donde marca el TPTK como valido asignandoselo a la variable PTK,
y envía el mensaje 4 al autenticador.
Entonces transita de inmediato al mecánismo PTK-DONE donde la PTK y GTK están instaladas para ser usadas
por el protocolo de confidencialidad de datos usando la petición primitiva MLME-SETKEYS.
Finalmente, abre el puerto 802.1x de modo que el solicitante pueda recibir y enviar tramas de datos normales.
Tenga en cuenta que el mecánismo de declaración toma en cuenta de manera explicita retransmisiones de ya sea
el mensaje 1 o 3, lo cual ocurre si el autenticador no recibe el mensaje 2 o 4, respectivamente.
Estas retransmisiones usan un contador de repetición EAPOL incrementado.
Confirmamos que el mecánismo de declaración en 802.11r concuerda con el original, que fue "definido"
en 802.11i usando descipciones textuales esparcidas a lo largo de la enmienda. Más importante, verificamos
dos propiedades de las cuales abusamos en nuestro ataque de reinstalación de claves. Primero, 802.11i declara que
el AP retransmita el mensaje 1 o 3 si no recibio una respuesta. Por lo tanto, el cliente de manejar las retransmisiones
del mensaje 1 o 3, coincidiendo el mecánismo de declaración de 802.11r. Adicionalmente, 802.11i declara que el cliente
debe instalar la PTK despues de procesar y responder el mensaje 3. Esto una vez más coincide con el mecánismo de declaración
dado en 802.11r.

## 3.2 El ataque de reinstalación de clave
Nuestro ataque de reinstalación de clave es fácil de detectar ahora: ya que el solicitante aún acepta retransmisiones del mensaje 3,
incluso cuando está en el mecánismo PTK-DONE, podemos forzar una reinstalación de la PTK.
Más precisamente, primero establecemos una posición de hombre en el medio (Man in the Middle) entre el solicitante y el autenticador.
Usamos está posición MitM para activar retransmisiones del mensaje 3 al prevenir que el mensaje 4 llegue al autenticador.
Como resultado, este retransmitirá el mensaje 3, lo cual causa que el solicitante reinstale una clave PTK ya en uso.
A su vez, esto restaura el nonce usado por el protocolo de confidencialidad de datos. Dependiendo del protocolo siendo usado,
esto le permite a un atacante repetir, decifrar, y/o falsificar paquetes.
En la Seccion 6.1 exploraremos en detalle cuales son los impactos prácticos de la reutilizacion del nonce para cada protocolo de confidencialidad.

En la práctica, surgen algunas complicaciones cuando se ejecuta el ataque.
Primero, no todos los clientes Wi-Fi implementan el mecánismo de declaración.
En particular, Windoes y iOS no aceptan retransmisiones del mensaje 3 (ver Tabla 1 columna 2).
Esto viola el estandar 802.11. Como resultado, estas implementaciones no son vulnerables a nuestro ataque de reinstalación de claves contra
el 4-way handshake. Desafortunadamente, desde una perspectiva de defensores, tanto iOS como Windows aún son vulnerables a nuestro ataque
de reinstalación de claves contra el group key handshake (ver Seccion 4).
Adicionalmente, ya que ambos Sistemas Operativos soportan 802.11r, es aún posible atacarlos indirectamente realizando un ataque de reinstalación
de clave contra el AP durante un FT handshake (ver Seccion 5).

Un segundo obstáculo menor es que debemos obtener una posición MitM entre el Cliente y el AP.
Esto no se puede hacer configurando un AP falso con una dirección MAC diferente,
y entonces redirigiendo los paquetes entre el AP real y el cliente.
Recordar de la Seccion 2.3 que la clave de sesión se basa en la dirección MAC del cliente y el AP,
significando que ambos generarian un clave de sesión diferente, lo que causaría que el ataque y el handshake fallen.
En su lugar, empleamos un ataque MitM basado en el canal, donde el AP es clonado en un diferente canal con la misma dirección MAC
del AP seleccionado. Esto asegura que el cliente y el AP generen la misma clave de sesión.

El tercer obstáculo es que ciertas implementaciones sólamente aceptan tramas protegidas usando el protocolo de confidencialidad de datos
una vez que una PTK ha sido instalada (ver Tabla 1 columna 3). Esto es problematico para nuestro ataque, ya que el autenticador
retransmitira el mensaje 3 sin cifra. Esto significa que el mensaje 3 retransmitido será ignorado por el solicitante.
Aunque esto parece frustrar nuestro ataque, encontramos una técnica para saltarse este problema (ver Sección 3.4).

En las siguientes dos Secciones, describiremos con detalle cómo ejecutar nuestro ataque de reinstalación de claves contra el 4-way
handshake bajo varias condiciones. Más precisamente, primero explicamos nuestro ataque cuando el cliente (victima) acepta retransmisiones
en texto plano del mensaje 3 (ver Tabla 1 columna 3). Entonces demostramos el ataque cuando la victima acepta sólamente retransmisiones cifradas
del mensaje 3 (ver Tabla 1 columna 4). La Tabla 1 columna 6 resume que dispositivos son vulnerables a alguna variante del ataque de reinstalacion
de claves contra el 4-way handshake. Recalcamos que el comportamiento de un dispositivo depende tanto del sistema operativo
como del NIC inalambrico (Controlador de Interfaz de Red o Adaptador o Tarjeta de Red, etc..) usado.
Por ejemplo, aunque linux acepta retransmisiones en texto plano del mensaje 3, los NICs Wi-Fi usados por varios dispositivos Android las rechazan.
Sin embargo, los teléfonos Android con un NIC diferente podrían de hecho aceptar retransmisiones en texto plano del mensaje 3.
