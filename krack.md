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

### 2.1 La Enmienda 802.11i
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

### 2.2 Autenticación y Asociación
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

### 2.3 El 4-way Handshake
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
del 4-way handshake son cifrados por el protocolo de confidencialidad
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

### 2.4 Protocolos de Integridad y Confidencialidad
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


### 2.5 El Group Key handshake
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
una trama uni-difusión al AP. El AP entonces cifra la trama usando la clave de grupo, y la transmite
a todos los clientes. Esto asegura que todos los clientes dentro del rango del AP reciban la trama.

## 3. ATACANDO EL 4-WAY HANDSHAKE
En esta sección mostramos que el mecánismo de declaración detrás del 4-way handshake es vulnerable a un ataque de reinstalación de claves.
Entonces demostramos cómo ejecutar este ataque en entornos de la vida real.

### 3.1 Mecánismo de Declaración del Solicitante
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

#### Tabla 1: Comportamiento de clientes
Columnas:
1. Clientes
2. Muestra si la retransmisión del mensaje 3 es aceptada.
3. Si la retransmision de un mensaje EAPOL en texto plano es aceptada si una PTK es instalada.
4. Si los mensajes EAPOL en texto plano son aceptados si se envian inmediatamente después del 1er mensaje 3.
5. Si es afectado por el ataque de la Sección 3.4.
6. y 7. Indican si el cliente es vulnerable a un ataque de reinstalación de clave contra el 4-way o group key handshake.

Implementación | Re. Msg3 | Pt. EAPOL | Quick Pt. | Quick Ct. | 4-way | Group
-------------- | -------- | --------- | --------- | --------- | ----- | -----
OS X 10.9.5 | :heavy_check_mark: | :heavy_multiplication_x: | :heavy_multiplication_x: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
macOS Sierra 10.12 | :heavy_check_mark: | :heavy_multiplication_x: | :heavy_multiplication_x: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
iOS 10.3.1 c | :heavy_multiplication_x: | N/A | N/A | N/A | :heavy_multiplication_x: | :heavy_check_mark:
wpa_supplicant v2.3 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
wpa_supplicant v2.4-5 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:a | :heavy_check_mark:a | :heavy_check_mark:
wpa_supplicant v2.6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:b | :heavy_check_mark:b | :heavy_check_mark:
Android 6.0.1 | :heavy_check_mark: | :heavy_multiplication_x: | :heavy_check_mark: | :heavy_check_mark:a | :heavy_check_mark:a | :heavy_check_mark:
OpenBSD 6.1 (rum) | :heavy_check_mark: | :heavy_multiplication_x: | :heavy_multiplication_x: | :heavy_multiplication_x: | :heavy_multiplication_x: | :heavy_check_mark:
OpenBSD 6.1 (iwn) | :heavy_check_mark: | :heavy_multiplication_x: | :heavy_multiplication_x: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:
Windows 7 c | :heavy_multiplication_x: | N/A | N/A | N/A | :heavy_multiplication_x: | :heavy_check_mark:
Windows 10 c | :heavy_multiplication_x: | N/A | N/A | N/A | :heavy_multiplication_x: | :heavy_check_mark:
MediaTek | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:

- A: Debido a un bug, una TK de sólo ceros será instalada. ver Sección 6.3.
- B: Sólamente la clave de grupo es reinstalada en el 4-way handshake.
- C: Ciertas pruebas son irrelevantes (no aplicables) porque la implementación no acepta retransmisiones del mensaje 3.


### 3.2 El ataque de reinstalación de clave
Nuestro ataque de reinstalación de clave es fácil de detectar ahora: ya que el solicitante aún acepta retransmisiones del mensaje 3,
incluso cuando está en el mecánismo PTK-DONE, podemos forzar una reinstalación de la PTK.
Más precisamente, primero establecemos una posición de hombre en el medio (Man in the Middle) entre el solicitante y el autenticador.
Usamos está posición MitM para activar retransmisiones del mensaje 3 al prevenir que el mensaje 4 llegue al autenticador.
Como resultado, este retransmitirá el mensaje 3, lo cual causa que el solicitante reinstale una clave PTK ya en uso.
A su vez, esto restaura el nonce usado por el protocolo de confidencialidad de datos. Dependiendo del protocolo siendo usado,
esto le permite a un atacante repetir, descifrar, y/o falsificar paquetes.
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

### 3.3 Retransmision en texto plano del mensaje 3
Si la victima acepta retransmisiones en texto plano del mensaje 3 aún después de instalar la clave de sesión, nuestro ataque de reinstalación
es sencillo. Primero, el atacante usa un un ataque MitM basado en el canal para poder manipular los mensaje del handshake.
Entonces evita que el mensaje 4 llegue al autenticador. Esto es ilustrado en la etapa 1 de la Figura 4, inmediatamente después de enviar el
mensaje 4, la victima instalará la PTK y GTK. En este punto la victima tambien abre el puerto 802.1x y comienza a transmir tramas de datos
normales (recordar Sección 2.3). 
Observe que la primer trama usa un valor nonce de 1 en el protocolo de confidencialidad de datos. Entonces en la tercera etapa del ataque,
el autenticador retransmite el mensaje 3 porque no recibió el mensaje 4. El atacante retransmite el mensaje 3 a la victima, ocasionando que reinstale
la PTK y GTK. Como resultado, restaura el nonce y el contador de repetición usado por el protocolo de confidencialidad de datos. Tenga en cuenta
que el atacante no puede repetir un mensaje 3 viejo, porque su contador de repetición EAPOL ya no está actualizado.
Ignoramos la etapa 4 del ataque por ahora. Finalmente, cuando la victima transmita su siguiente trama de datos, el protocolo de confidencialidad
de datos reutiliza los nonces. Tenga en cuenta que un atacante puede esperar una cantidad de tiempo arbitraria antes de retransmitir el mensaje 3
a la victima. Por lo tanto, podemos controlar la cantidad de nonces que serán reusados. Ademas un atacante puede volver a realizar el ataque
desautenticando al cliente, después de lo cual se reconectará a la red y ejecutará un nuevo 4-way handshake.

La Figura 4 también muestra que nuestro ataque de reinstalación de clave ocurre espontáneamente si el mensaje 4 se pierde debido al ruido de fondo.
En otras palabras, los clientes que aceptan retransmisiones en texto plano del mensaje 3, podrían estar actualmente reutilizando nonces sin que
siquiera un atacante estuviera presente. Inspirado por esta observación, un atacante también podría selectivamente interferir el mensaje 4,
resultando en un ataque sigiloso que es indistinguible de la interferencia de fondo aleatoria.

Ahora volvemos a la etapa 4 del ataque. La meta de está etapa es completar el handshake en el lado del autenticador. Lo cual no es trivial ya que
la victima ya instalo la PTK, lo que significa que el mensaje está cifrado. Y como el autenticado aún no ha instalado la PTK, esto normalmente
rechazaría este mensaje 4 cifrado. Sin embargo, una cuidadosa inspección del estandar 802.11 revela que el autenticador podría aceptar
***cualquier*** contador de repetición usado en el 4-way handshake, no sólamente el último:
> "Al recibir el mensaje 4, el Autenticador verifica que el valor del campo Contador de Repetición de Clave sea uno usado en este 4-way handshake"

En la práctica, encontramos que vaios APs en efecto aceptan un contador de repetición antiguo. Más precisamente, algunos APs aceptan contadores
de repetición usados en un mensaje al cliente, pero no han sido usados aún en una respuesta del cliente (ver columna 2 en Tabla 2 en la página 8).
Estos APs aceptarán el antiguo mensaje 4 sin cifrar, que tiene el contador de repetición `r + 1` en la Figura 4.
Como resultado, estos APs instalarán la PTK, y comenzarán a enviar tramas uni-difusión cifradas al cliente.

A pesar que la Figura 4 sólamente ilustra la reutilización del nonce en tramas envíadas por el cliente, nuestro ataque tambien nos permite
repetir tramas. Primero, después de que el cliente reinstala la GTK en la etapa 3 del ataque, tramas de difusión y multi-difusión que envíe el AP
después de retransmitir el mensaje 3 puedens ser repetidas. Esto se debe a que los contadores de repetición son restaurados también cuando se
reinstala una clave. Segundo, si podemos hacer que el AP instale la PTK, también podemo repteir tramas uni-difusión enviadas por el AP al cliente.

Confirmamos que el ataque mostrado en la Figura 4 funciona contra la implementación MediaTek del cliente Wi-Fi, y contra ciertas versiones de 
wpa_supplicant (ver Sección 6.3). Cómo atacamos otras implementaciones se explica en la siguiente sección.

### 3.4 Retransmisión Cifrada del mensaje 3
Ahora describimos cómo podemos atacar clientes que, una vez instalada la PTK, aceptan sólamente retransmisiones cifradas del mensaje 3.
Para lograr esto, explotamos una condición de carrera inherente entre la entidad ejecutando el 4-way handshake, y la entidad implementando el
protoclo de confidencialidad de datos.

Como calentamiento, primero atacamos la implementación del solicitante de Android. Aquí encontramos que Android acepta retransmisiones del mensaje 3
en texto plano cuando son enviados inmediatamente despué del 3er mensaje (ver columna 4 de la Tabla 1).
La Figura 5 muestra por qué sucede esto y cómo se puede explotar. Tenga en cuenta que el AP no está dibujado en esta figura: sus acciones están
claras desde el contexto. En nuestro ataque, primero dejamos que el cliente y el AP intercambien el mensaje 1 y 2.
Sin embargo, no transmitimos el 1er mensaje 3 al cliente. En cambio esperamos a que el AP transmita un segundo mensaje 3.
En la etapa 2 del ataque; enviamos ambos mensajes 3 inmediatamente uno tras otro al cliente.
El NIC inalambrico, el cual implementa  el protocolo de confidencialidad de datos, no tiene una PTK instalada, debido a eso redirige ambos
mensajes a la cola de recepción de paquetes de la CPU principal. La CPU principal, que implementa el 4-way handshake, responde al primer mensaje
3 y le ordena al NIC inalambrico instalar la PTK. En la etapa 4 del ataque, la CPU principal del cliente toma el segundo mensaje 3 de su cola de
recepción. Aunque nota que la trama no fue cifrada, Android y Linux permiten tramas EAPOL sin cifrar como una excepción, y por lo tanto la
CPU principal procesará el paquete 3 retransmitido. Ya que el NIC apenas ha instalado la PTK, la respuesta será cifrada con un valor nonce de 1.
Después de esto lo ordena al NIC inalambrico que reinstale la PTK. Al hacer esto, el NIC restaura el nonce y el contador de repetición asociados
a la PTK, lo que significa que la siguiente trama transmitida reutilizará el nonce 1.

Ahora mostramos cómo atacar OpenBSD, OS X y macOS (ver la Tabla 1 columna 5). Estos dispositivos sólamente aceptan retransmisiones cifradas del
mensaje 3. Similar al ataque en Android, abusamos condiciones de carrera entre el NIC inalambrico y la CPU principal.
Sin embargo, ahora atacamos una ejecución del 4-way handshake que renueva las PTK (rekeys).
Recuerde de la Sección 2.3 que todos los mensajes transmitidos durante una nueva operación se cifran usando el protocolo de confidencialidad
de datos.

La Figura 6 ilustra los detalles del ataque.Tengua en cuenta que el AP no está dibujado en está figura: sus acciones son limpiadas del contexto.
Una vez más el atacante usa una posición MitM basado en canal. Entonces deja que la victima y el atacante ejecuten el inicial 4-way handshake, y
espera hasta que un segundo 4-way handshake se inicie para renovar la PTK. A pesar de que el atacante sólo ve tramas cifradas, los mensajes en
un 4-way handshake pueden ser detectados por su largo unico y su destino. En este punto, el ataque es analogo al caso de Android.
Es decir, en la etapa 2 del ataque, el atacante no redirige el primer mensaje 3 inmediatamente. En cambio, espera hasta que el AP retransmita
el mensaje 3, y entonces redirige ambos mensajes uno tras otro a la victima (ver Figura 6 etapa 2). El NIC inalambrico descifrará ambos mensajes
usando la PTK actual, y los redirigira a la cola de recpción de paquetes de la CPU principal. En la tercera etapa del ataque, la CPU principal
de la victima procesa el primer mensaje 3, lo responde, y le ordena al NIC que instale la nueva PTK. En la cuarta etapa, la CPI principal toma
el segundo mensaje 3 de la lista de recepción. Y ya que una PTK está instalada, OpenBSD, OS X y macOS (aquí llamados CPU principal) requeriran
que el mensaje este cifrado. Sin embargo, no comprueban con que clave fue cifrado el mensaje. Como resultado, a pesar de que el mensaje fue
decifrado con la PTK antigua, la CPU principal lo procesará. El mensaje 4 enviado como respuesta está cifrado ahora con la nueva PTK usando
un valor nonce de 1. Después de esto la CPU principal le ordena al NIC que reinstale la nueva PTK, de este modo restaurando el nonce y el contador
de repetición. Finalmente, la siguiente trama de datos que la victima transmita será cifrado de nuevo usando la nueva PTK con un nonce de 1.
Nosotros confirmamos este ataque contra OpenBSD 6.1, OS X 10.9.5, y macOS Sierra 10.12.

OpenBSD solo es vulnerable si el cifrado es descargado a la NIC inalámbrica. Por ejemplo, el controlador `iwn` y los dispositivos asociados
admiten el cifrado de hardware, por lo tanto, son vulnerables. Sin embargo, el controlador `rum` ejecuta el cifrado de software en la misma
entidad que el 4-way handshake, y no es vulnerable (ver Tabla 1, columna 5).

Esta técnica de ataque requiere que esperemos hasta que se produzca una renovación de la clave de sesión.
Varios AP hacen esto cada hora, algunos ejemplos son [24, 26]. En la práctica, los clientes también pueden solicitar una renovación enviando
una trama EAPOL al AP con los bits Request y Pairwise establecidos.
Coincidencialmente, los routers Broadcom no verifican la autenticidad (MIC) de esta trama, lo que significa que un adversario puede forzar a
los AP Broadcom a iniciar un handshake de renovación de clave. Con todo combinado, podemos suponer que eventualmente ocurrirá una renovación,
lo que significa que un atacante puede llevar a cabo el ataque de reinstalación clave.

### 3.5 Atacando el Handshake PeerKey

El handshake PeerKey está relacionado con el 4-way handshake, y se usa cuando dos clientes quieren comunicarse entre ellos directamente de
manera segura. Consta de dos fases. En la primera fase, se realiza un handshake de clave maestra (MasterKey | SMK) de Enlace de Estación a
Estación (Station to Station Link | STSL). Se negocia un secreto maestro compartido entre ambos clientes. En la segunda fase, una clave de sesión nueva se deriva de esta clave maestra usando el handshake STSL de Clave Transitoria (STSL Transient Key | STK).
Aunque este protocolo no parece ser ampliamente compatible, constituye un buen caso de prueba para medir que tan aplicable es nuestra técnica
de reinstalación de clave.

Como era de esperarse, el handshake SMK no se ve afectado por nuestro ataque de reinstalación de claves.
Después de todo, la clave maestra negociada en este handshake no es utilizada por un protocolo de confidencialidad de datos,
lo que significa que no hay nonces o contadores de repetición que restaurar. Sin embargo, el handshake STK se basa en el 4-way handshake,
y sí instala una clave para su uso mediante un protocolo de confidencialidad de datos. Como resultado, se puede atacar de la misma manera
que el 4-way handshake.
El ataque resultante se probó contra wpa_supplicant. Para llevar a cabo la prueba, modificamos otra instancia de wpa_supplicant para enviar
un segundo mensaje 3 (retransmitido). Esto confirmó que un wpa_supplicant no modificado reinstalará la clave STK al recibir un
mensaje 3 retransmitido del handshake STK.
Sin embargo, no encontramos otros dispositivos que soporten PeerKey. Como resultado, el impacto de nuestro ataque contra el handshake PeerKey es bastante bajo.

## 4. ROMPIENDO EL HANDSHAKE DE CLAVE GRUPAL
En esta sección aplicamos nuestra técnica de reinstalación de clave contra el handshake de clave grupal. Mostramos que todos los cliente Wi-Fi
son vulnerables, permitiendo que un atacante repita tramas de difusión y multidifusión.

### 4.1 Detalles del handshake de clave grupal
Las redes renuevan la clave grupal periodicamente, para asegurar de que sólo los clientes autorizados recientemente posean esta clave.
En el caso más defensivo la clave grupal es renovada cada vez que un cliente se desconecta de la red. La nueva clave grupal se distribuye
usando un handshake de clave grupal, y este handshake ha sido formalmente provado como seguro en [39]. Como se muestra en la Figura 2,
el handshake es iniciado por el AP cuando envía un mensaje de grupo 1 a todos los clientes. El AP retransmite este mensaje si no recibio una
respuesta adecuada. Tenga en cuenta que el contador de repetición EAPOL de estos mensajes retransmitidos se incrementa siempre por 1. En nuestro
ataque, la meta es coleccionar un mensaje de grupo 1 retransmitido, bloquearlo para que no llegue al cliente, y redirigirlo al cliente en un
momento posterior.
Esto hará que el cliente reinicie el contador de repetición de la clave grupal instalada.

El primer pre-requisito de nuestro ataque, es que el cliente reiniciará el contador de repetición cuando instale una clave grupal ya-en-uso.
Ya que el cliente también usa el primitivo MLME-SETKEYS.request para instalar la clave grupal, este debería ser el caso. Nosotros confirmamos
que en la práctica todos los clientes Wi-Fi en efecto reiniciaron el contador de repetición de una clave grupal ya-en-uso (ver Tabla 1, columna 7).
Por lo tanto, todos los clientes Wi-Fi son vulnerables a nuestros ataques posteriores.

El segundo pre-requisito es que debemos poder reunir un mensaje grupal 1 que el cliente (todavía) acepte, y que contenga una clave grupal que ya
este en uso por el AP. El cómo lograr esto depende de cuando el AP comienza a usar la clave grupal. En particular, el AP podría comenzar a usar la
clave grupal inmediatamente despues de enviar el primer mensaje grupal 1, o podría retrasar la instalación de la clave grupal hasta que el cliente
responda usando un mensaje grupal 2. La Tabla 2, columna 3, resume este comportamiento para varios APs. Tenga en cuenta que de acuerdo al estandar,
la nueva clave grupal debería ser instalada después de que todas las estaciones respondan con un mensaje grupal 2, es decir, la GTK debe ser
instalada de una forma retrasada [1, 12-53]. Cuando el AP instala la clave inmediatamente, nuestro ataque de reinstalación de clave es sencillo.
Sin embargo, si un AP instala la clave grupal de una manera retrasada, nuestro ataque se vulve más intricante, Discutiremos ambos en casos en más
detalles en la Sección 4.2 y 4.3, respectivamente.

Recordar de la Sección 2.3 que sólamente el AP transmitirá tramas de difusión y multi-difusión reales (es decir, tramas de grupo) las cuales
estan cifradas usando la clave grupal. Ya que nuestro ataque de reinstalación de claves se dirige al cliente, esto significa que no podemos forzar
la reutilización del nonce en el cifrado.
Sin embargo, el cliente el contador de repetición cuando reinstala la clave grupal, lo cual puede ser abusado para repetir tramas hacia los clientes.

La mayoria de APs renuevan la clave grupal cada hora. Algunas redes incluso renuevan la clave cada vez que un cliente abandona la red.
Adicionalmente, los clientes pueden activar un handshake de clave grupal enviando una trama EAPOL cifrada con las banderas Request y Group.
De nuevo, los routers Broadcom no verifican la autenticidad del mensaje, lo que significa que un atacante puede falsificarlo para activar una
actualización de la clave grupal. Con todo combinado, podemos asumir que la mayoria de redes eventualmente ejecutarán una actualización de
clave grupal, la cual podemos atacar posteriormente.

## 4.2 Atacando la Instalación de Clave Inmediata
La figura 7 ilustra nuestro ataque de reinstalación de clave cuando el AP instala inmediatamente la clave grupal despues de enviar el mensaje
grupal 1 a todos los clientes. Tenga en cuenta que los mensajes de handshakes de clave grupal se cifran usando el algoritmo de confidencialidad
de datos con la PTK actual. Al recibir el mensaje grupal 1, el cliente instala la nueva GTK y responde con el mensaje grupal 2. El atacante impide
que este mensaje llegue el AP. Por lo cual, el AP retransmitirá un nuevo mensaje grupal 1 en la etapa 2 del ataque. Ahora esperamos hasta
que una trama de difusión sea transmitida, y la redirigimos hacia la victima. Despues de esto, redirigimos el mensaje grupal 1 de la etapa 2
a la victima. Como resultado, la victima reinstalará GTK, y de este modo reiniciará su contador de repetición asociado. Esto nos permite repetir
la trama de difusión (ver Etapa 5). El cliente acepta está esta trama ya que su contador de repetición fue reiniciado.

Es esencial que la trama de difusión que repetimos se envíe antes de la retransmisión del mensaje grupal 1. Esto se debe a que el mensaje grupal
1 contiene el valor actual del contador de repetición de la clave grupal (recordar la Sección 2.5). Por lo tanto, si se envía después de la
trama de difusión, contendría el contador de repetición actualizado, y por lo tanto, no se puede abusar para reiniciar el contador de
repetición de la víctima.

Confirmamos este ataque en la práctica para APs que instalan la clave grupal inmediatamente despues de enviarl el mensaje grupal 1
(ver Tabla 2, columna 3). Basado en nuestros experimentos, todos los clientes Wi-Fi son vulnerables a este ataque cuando estan conectados a un
AP que se comporta de esta manera.

## 4.3 Atacando la Instalación de Clave Retrasada
Atacar el handshake de clave grupal cuando el AP instala la GTK de un manera retrasada es más tedioso. Tenga en cuenta que el ataque anterior
fallaría ya que la trama de difusión transmitida en la etapa 3 de la Figura 7 estaría cifrada todavía por la clave grupal antigua. Esto es
problematico porque el mensaje grupal 1 (re)instala la nueva clave grupal, y debido a eso, no puede ser usada para restaurar el contador de
repetición de la antigua clave grupal.

Una forma de aborder este problema está ilustrada en la Figura 8. Las primeras 2 etapas de este ataque son similares a las anteriores.
Es decir, el AP genera una nueva clave grupal, la transporta a la victima, y el atacante impide que el mensaje grupal 2 llege al AP.
Esto hace que el AP retransmita el mensaje grupal 1 usando un contador de repetición EAPOL incrementado de `r + 1`.
Sin embargo, en la etapa 3 del ataque, redirigimos el antiguo mensaje grupal 2 con el contador de repetición `r` al AP.
Interesantemente, el AP debería aceptar este mensaje aunque este no use el último valor contador de repetición.
> Al recibir el mensaje [grupal] 2, el AP verifica que el valor del campo del contador de repetición de clave coincida con uno
que haya sido utilizado en el handshake de clave grupal.

El estandar no requiere que el contador de repetición coincida con el ***último*** que el AP ha usado. En cambio, debe coincidir con uno
usado en el handshake de clave grupal, es decir, uno usado en cualquiera de los mensajes grupales 1 (re)transmitidos. En práctica descubrimos
que en efecto varias implementaciones aceptan este antiguo aún-no-recibido contador de repetición (ver Tabla 2, columna 2). Como resultado,
el AP instala la nueva clave grupal. Desde este punto, el ataque continua de una manera similar al anterior. Es decir, esperamos hasta que
una trama de difusión sea transmitida, realizamos la reinstalación de la clave grupal en la etapa 5 del ataque, entonces repetimos la trama
de difusión en la etapa 6.

Nuevamente es esencial que la trama de difusión que queremos repetir sea enviada antes de la retransmisión del mensaje grupal 1.
De otra manera incluiria el contador de repetición actualizado de la clave grupal.

Probamos este ataque contra APs que instalan la GTK de una manera retrasada, y que aceptan contadores de repetición que han enviado en un
mensaje al cliente, pero aún no lo recibieron en una respuesta (recordar Tabla 2, columna 2). Tenga en cuenta que ya sabemos que todos los
clientes Wi-Fi restauran el contador de repetición cuando reinstalan una GTK, y por lo tanto son todos vulnerables.
Finalmente, un AP OpenBSD no es vulnerable ya que instala la GTK de una manera retrasada, y sólamente acepta el último contador de repetición.

## 5. ATACANDO EL HANDSHAKE FT 802.11R
En esta sección presentamos el handshake de Transición BSS Rápida (Fast BSS Transition | FT), y mostramos que implementaciónes de este son
afectadas por nuestro ataque de reinstalación de clave.

## 5.1 El handshake de Transición BSS Rápida (FT)
La enmienda 802.11r añadio el handshake Transición de Conjunto de Servicio Básico Rápida (Fast Basic Service Set (BSS) Transision (FT)).
Su meta es reducir el tiempo de transmisión cuando un cliente se mueve de un AP, a otro en la misma red protegida
(es decir, del mismo Conjunto de Servicio Básico). Tradicionalmente, esto requiere un handshake que incluya un nuevo 802.1x y 4-way handshake
(recordar Figura 2). Sin embargo, ya que el FT handshake confía en las claves maestras derivadas durante una conexión previa con la red, un nuevo
handshake 802.1x no es requerido. Adicionalmente, esto añade la etapa del 4-way handshake en las tramas de autenticación y reasociación.

Un handshake FT normal se muestra en la etapa 1 de la Figura 9. Observe que a diferencia del 4-way handshake, el FT handshake es iniciado por el
solicitante. Los primeros dos mensajes son una Petición de Autenticación (AuthReq), y una Respuesta de Autenticación (AuthResp).
Son funcionalmente equivalentes al Mensaje 1 y 2 del 4-way handshake, respectivamente, y llevan nonces generados aleatoriamene que serán
usados para derivar una nueva clave de sesión.
Después de esto, el cliente envía una Petición de Reasociación (ReassoReq), y el AP responde con una Respuesta de Reasociación (ReassoResp).
Son funcionalmente similares al Mensaje 3 y 4 del 4-way handshake, finalizan el FT handshake y transportan la GTK al cliente.

Sólo los dos mensajes de reasociación son autenticados usando un MIC (ver Figura 9). Adicionalmente, ninguno de los mensajes en el FT handshake
contiene un contador de repetición. En cambio, El FT handshake depende del ANonce y SNonce aleatorio para brindar protección de repetición entre
diferentes invocaciónes del handshake.

De acuerdo al estandar, la PTK debe ser instalada después de que la respuesta de autenticación sea envíada o recibida. Esto se ilustra con
las cajas grises en la etapa 1 de la Figura 9. Adicionalmente, el puerto lógico de 802.1x sólamente se abre después de enviar o recibir la
petición de reasociación.

Esto se asegura de que, aunque la PTK ya este instalada mientras el handshake este todavia en progreso, el AP y el cliente sólo aceptan y
transmiten tramas una vez el handshake se ha completado. Combinado, esto implica que el handshake FT, tal y como se define en la enmienda 802.1r,
no es vulnerable a un ataque de reinstalación de clave. Sin embargo, por medio de experimientos e inspecciones al código, encontramos que 
la mayoria de implementaciones en realidad instalán la PTK, así como la GTK, después de enviar o recibir la repuesta de autenticación.
Este comportamiento se ilustra con las cajas negras en la etapa 1 de la Figura. Como resultado, en la práctica la mayoria de las implementaciones
del FT handshake ***son*** vulnerables al ataque de reinstalación de clave.

### 5.2 Un Ataque de Reinstalación de Clave contra el AP
Debido a que el AP instala la PTK en respuesta a una petición de reasociación, nuestro objetivo será repetir esta trama. Recalcamos que, en la
prácticam los APs deberían aceptar peticiones de reasociación. Esto se debe a que la respuesta de reasociación del AP podría perderse debido
al ruido de fondo, haciendo que el cliente envíe una nueva petición.

La Figura 9 muestra el ataque de reinstalación de clave resultante contra el handshake FT. Tenga en cuenta que no requerimos una posición de
hombre-en-el-medio. En cambio, poder escuchar a escondidas e inyectar tramas es suficiente. En la primera etapa del ataque, permitimos que el
cliente y el AP ejecuten un FT handshake normal. Entonces esperamos a que el AP haya transmitido una o más tramas de datos cifradas.
En este punto, le repetimos la petición de reasociación al AP. Ya que no contiene un contador de repetición, y contiene un MIC valido, el AP
aceptará y procesará la trama repetida. Como resultado, el AP reinstalará la PTK en la etapa 3 del ataque, de este modo restaura el nonce
y contador de repetición asociados. Finalmente, la siguiente trama de enviada por el AP será cifrada usando un nonce ya-en-uso. Similar a
nuestros previos ataques de reinstalación de claves, esto tambien le permite a un atacante repetir tramas de datos antiguas enviadas por el
cliente al AP. Recalcamos que nuestro ataque es particularmente devastador contral el FT handshake ya que sus mensajes no contienen un contador
de repetición. Esto le permite a un atacante repetir una petición de autenticación de manera continua, restaurando cada vez el nonce y el
contador de repetición usados por el AP.

Probamos este ataque conta todos nuestros tres APs que soportan 802.11r. El primero es la implementación de código abierto hostapd, la segunda es
la implementación para routers dómesticos MediaTek ejecutandose en un Linksys RE700, y el último es un AP Aerohive profesional.
Los tres fueron vulnerables al ataque de reinstalación de clave anterior.

Tenga en cuenta que si la respuesta de reasociación se pierde debido al ruido de fondo, el cliente retransmitirá la petición de reautenticación
espontaneamente, provocando que el AP reinstale la clave. Es decir, sin que un at.acante este presente, los AP ya podrían estar reusando nonces.

Tenga en cuenta que los mensajaes en el handshake nunca se someten a protección (adicional) usando un protocolo de confidencialidad de datos.
En particular, Management Frame Protection (MFP) no protege tramas de autenticación y reasociación. Por lo tanto, los ataques de reinstalación
de clave contra el FT handshake son triviales incluso si MFP esta activado.

### 5.3 Abusando de las Solicitudes de Transición BSS
Un handshake FT sólo se realiza cuando una estación se translada de un AP a otro. Esto limita cuando puede tomar lugar un ataque. Sin embargo,
podemos forzar a un cliente a realizar un FT handshake de la siguiente manera. Primero, suponga que un cliente está conectado a un AP de una
red que soporta 802.1r. Entonces, si ningun otro AP de la red está en el rango del cliente, clonamos un AP real de está red al lado del cliente
usando un ataque de aguro gusano (wormhole). Esto le hace pensar al cliente que otro AP de la red seleccionada esta cerca. Finalmente, enviamos
una Petición de Gestión de Transición BSS al cliente. Esta trama se usa para equilibrar la carga y le ordena al cliente trasladarse a otro AP.
Es una trama de gestión no autenticada, y por lo tanto puede ser falsificada por un atacante. En consecuencia, el cliente acepta esta trama, y
se traslada al AP (wormholed) usando un FT handshake.

Probamos esto contra clientes que soportan 802.11r. Esto confirmó que wpa_supplicant, iOS y Windows aceptan la petición de transicion, y se
trasladan a otro AP usando un FT handshake.

## 6. EVALUACIÓN Y DISCUSIÓN
En esta sección evaluamos el impacto de la reutilización del nonce para los protocolos de confidencialidad de datos 802.11, presentamos ejemplos
de escenarios de ataque, discutimos la implementación de vulnerabilidades especificas, explicamos por qué las pruebas de seguridad pasaron por
alto nuestros ataques, y presentamos contramedidas

### 6.1 Impacto de la Reutilización del Nonce en 802.11
El impacto preciso de la reutilización del nonce causado por nuestros ataques depende del protocolo de confidencialidad de datos que se utilice.
Recuerde que puede ser TKIP, CCMP o GCMP. Los tres protocolos usan un cifrado de flujo para cifrar las tramas.
Por lo tanto, la reutilización de un nonce siempre implica la reutilización del flujo de claves.
Esto se puede usar para descifrar paquetes. Observamos que en nuestro ataque el contador de repetición de la víctima también se restablece.
Por lo tanto, los tres protocolos también son vulnerables a los ataques de repetición.

Cuando se usa TKIP, también podemos recuperar la clave MIC de la siguiente manera.
Primero, abusamos de la reutilización del nonce para descifrar un paquete TKIP completo, incluido su campo MIC.
Luego atacamos el débil algoritmo de Michael: con la trama en texto plano y su valor MIC descifrado, podemos recuperar la clave MIC.
Debido a que TKIP utiliza una clave MIC diferente para cada dirección de la comunicación (recordar la Sección 2.4),
esto nos permite falsificar tramas en una dirección específica. El origen de esta dirección es el dispositivo al que se dirige el ataque de
reinstalación de clave. La Tabla 3 resume esto debajo de las filas que mencionan TKIP.

Cuando se usa CCMP, los ataques prácticos están restringidos a la repetición y el descifrado de paquetes.
Aunque hay algunos trabajos que analizan los ataques de falsificación de mensajes cuando se repiten nonces,
los ataques son teóricos y no se pueden usar para falsificar mensajes arbitrarios.

Cuando se usa GCMP, el impacto es catastrófico. Primero, es posible repetir y descifrar paquetes.
Además, se puede recuperar la clave de autenticación, que en GCMP es usada para proteger ambas direcciones de la comunicación
(recordar la Sección 2.4). Por lo tanto, a diferencia de TKIP, un atacante puede falsificar paquetes en ambas direcciones.

Ya que se espera que GCMP se adopte a un ritmo elevado en los próximos años bajo el nombre de WiGig, esta es una situación preocupante.

En general, un atacante siempre puede repetir, descifrar o falsificar paquetes en una dirección específica de la comunicación.
La dirección concreta depende del handshake que se está atacando. Por ejemplo, ya que con el 4-way handshake atacamos al cliente,
puede usarse para: (1) repetir tramas de uni-difusión y de difusión/multidifusión hacia el cliente; (2) descifrar tramas enviadas por el
cliente al AP; y (3) falsificar tramas del cliente hacia el AP.
Sin embargo, contra el FT handshake atacamos al AP en lugar del cliente, lo que significa que podemos repetir, descifrar y/o falsificar
paquetes en sentido contrario. La Tabla 3 resume esto en el apéndice, teniendo en cuenta el handshake atacado.

Por último, en varios casos podemos falsificar mensajes del cliente hacia el AP (ver Tabla 3).
Curiosamente, el AP generalmente no es el destino final de una trama, en su lugar, reenviará la trama a su destino real.
Esto significa que podemos falsificar paquetes hacia cualquier dispositivo conectado a la red. Dependiendo del AP, incluso es posible enviar
un trama que se refleje de nuevo al cliente.

### 6.2 Ejemplos de escenarios de ataque
Entre otras cosas, nuestros ataques de reinstalación de clave permiten a un atacante descifrar un paquete TCP,
conocer el número de secuencia y secuestrar el stream TCP para inyectar datos arbitrarios.
Esto permite uno de los ataques más comunes a través de redes Wi-Fi: inyectar datos maliciosos en una conexión HTTP no cifrada.

La capacidad de repetir tramas de difusión y multidifusión, es decir, tramas de grupo, también es una clara violación de seguridad.
Para ilustrar cómo esto podría afectar a sistemas reales, considere el protocolo de tiempo de red (Network Time Protocol | NTP) que funciona
en modo de difusión. En este modo, el cliente primero pasa por un proceso de inicialización y luego sincroniza su reloj escuchando paquetes NTP de difusión autenticados.

Malhotra y Goldberg han demostrado que si se repiten estas tramas de transmisión, las víctimas se quedan atrapadas en un momento determinado
para siempre. Usando nuestro ataque de clave grupal, podemos reproducir estas tramas incluso si se envían a través de una red Wi-Fi protegida.
Tenga en cuenta que manipular el tiempo de esta manera quebranta la seguridad, por ejemplo, los certificados TLS, DNSSEC, autenticación
Kerberos y bitcoin. Otro ejemplo es el protocolo de automatización del hogar xAP y xPL.
En general, utilizan paquetes UDP de difusión para enviar comandos a los dispositivos. Suponemos que nuestro ataque clave de reinstalación
nos permite repetir estos comandos. Todos estos ejemplos combinados, ilustran que no debe subestimarse el impacto de repetir tramas de
transmisión o multidifusión.

### 6.3 Vulnerabilidad de clave de cifrado de sólo ceros
Nuestro ataque de reinstalación de clave contra el 4-way handshake descubrió un comportamiento especial en wpa_supplicant.
En primer lugar, la versión 2.3 y versiones anteriores son vulnerables a nuestros ataques sin efectos secundarios inesperados.
Sin embargo, descubrimos que las versiones 2.4 y 2.5 instalan una clave de cifrado (TK) de sólo ceros cuando reciben un mensaje retransmitido.
Esta vulnerabilidad parece ser causada por una observación en el estándar 802.11 que indirectamente sugiere borrar la TK de la memoria una vez
que ha sido instalada. La versión 2.6 corrigió este error instalando únicamente la TK cuando recibía el mensaje 3 por primera vez.
Sin embargo, cuando se parcheó este error, solo se consideró un escenario benigno donde el mensaje 3 se retransmitiá porque el mensaje 4 se
habia perdido debido al ruido de fondo. No consideraron que un atacante activo puede abusar de este error para forzar la instalación de una 
clave de sólo ceros. Como resultado, el parche no se trató como crítico de seguridad y no se transfirió a versiones anteriores.
Independientemente de este error, todas las versiones de wpa_supplicant vuelven a instalar la clave grupal cuando reciben un mensaje
retransmitido, y también son vulnerables al ataque de clave grupal de la Sección 4.

Ya que Android usa internamente una versión ligeramente modificada de wpa_supplicant, también se ve afectado por estos ataques.
En particular, inspeccionamos el repositorio oficial de código fuente de wpa_supplicant de Android, y descubrimos que todas las versiones de
Android 6.0 contienen la vulnerabilidad de clave de cifrado de sólo ceros. Android Wear 2.0 también es vulnerable a este ataque.
Aunque los fabricantes terceros pueden usar una versión diferente de wpa_supplicant en sus compilaciones de Android,
esta es una fuerte indicación de que la mayoría de las versiones de Android 6.0 son vulnerables.
En otras palabras, el 31.2% de los teléfonos inteligentes con Android son vulnerables a la vulnerabilidad de la clave de cifrado de sólo ceros.
Finalmente, también confirmamos empíricamente que Chromium es vulnerable a la vulnerabilidad de la clave de cifrado de sólo ceros.

### 6.4 Limitaciones de las pruebas de seguridad
Curiosamente, nuestros ataques no violan las propiedades dre seguridad probadas en el análisis formal del 4-way handshake ni de la clave grupal.

Primero, He et al. demostró que el 4-way handshake proporciona confidencialidad de clave y autenticación de sesión.
La confidencialidad de clave establece que solo el autenticador y el solicitante poseerán la PTK.
Ya que no recuperamos la PTK, esto todavía se cumple de forma adecuada.
La autenticación de sesión se comprobó utilizando la noción estándar de conversaciones coincidentes.
Intuitivamente, esto indica que un protocolo es seguro si la única forma en que un atacante puede lograr que una parte complete el protocolo 
es transmitiendo fielmente los mensajes. Nuestros ataques, incluida la posición MitM basada en el canal que empleamos,
no violan esta propiedad: solo podemos hacer que los puntos finales completen el handshake enviando mensajes (retransmitidos).

En segundo lugar, He et al. comprobó el orden de las claves y el secreto clave para el handshake de clave grupal.
El orden de las claves garantiza que los suplicantes no instalen una GTK antigua.
Esto sigue siendo cierto en nuestro ataque, ya que reinstalamos la clave grupal actual.
Además, no conocemos la clave grupal, por lo tanto, el secreto clave tampoco es violado por nuestros ataques.

## 7. TRABAJO RELACIONADO
En esta sección exploramos la historia de los ataques de reinstalación de clave y damos un resumen de otros trabajos de seguridad de protocolo
y Wi-Fi.

### 7.1 Ataques de reinstalación de clave
No tenemos conocimiento de trabajo previo sobre los ataques de reinstalación de claves.
Esta falta de trabajo previo es probablemente una de las razones por las cuales los handshake de Wi-Fi criptográficos que investigamos aún
eran vulnerables a estos ataques. Por ejemplo, solo ahora descubrimos que el 4-way handshake de 14 años es vulnerable a los ataques de
reinstalación de clave.
Además, este defecto no solo está presente en las implementaciones, sino también en la especificación del protocolo (estándar).


Un escenario algo relacionado que también conduce a la reutilización del nonce son las fallas de energía.
Aquí, después de un corte de energía, la clave se restaura desde la memoria no violada al arrancar, pero el nonce será restaurado a su valor inicial.
Zenner propone soluciones sugeridas para este problema.
Sin embargo, a diferencia de los ataques de reinstalación clave, no se pueden desencadenar fallos de energia de forma remota a través de una red.
En cambio, esto requiere acceso físico al dispositivo que está siendo atacado.
Además, las fallas de energía no afectan la seguridad de los protocolos que estudiamos, ya que estos handshake se usan precisamente para evitar mantener el estado entre conexiones antiguas y conexiones nuevas.

En [16], Bock et al. descubrió que algunos servidores TLS estaban usando nonces estáticos.
Esto fue causado por una implementación defectuosa del protocolo de capa de grabación TLS.
Es decir, no fue causado por una reinstalación de una clave que ya estaba en uso.
Además, algunos servidores utilizan nonces generados aleatoriamente, lo que significa que en la práctica es probable que se produzca la reutilización debido a la paradoja del cumpleaños.
Por el contrario, los ataques de reinstalación de clave permiten a un atacante forzar la reutilización de nonces según demanda repitiendo los mensajes del handshake, y son causados por fallas en la especificación (o implementación) del protocolo del handshake.

McGrew hizo una encuesta de mejores prácticas para generar IVs y nonces, y resume cómo son generadaos y utilizados en varios protocolos [51].
Sin embargo, en la discusión de los riesgos de seguridad, no se mencionan (variaciones de) los ataques de reinstalación de clave.

Otro trabajo algo relacionado es el de Beurdouche et al. [14] y el de de Ruiter y Poll [27].
Descubrieron que varias implementaciones de TLS contenían mecánismos de estado defectuosos.
En particular, ciertas implementaciones permitian erróneamente que los mensajes del handshake se repitieran.
Sin embargo, no pudieron crear ejemplos de ataques que explotaran la capacidad de repetir mensajes.
Supongamos que un atacante puede repetir ciertos mensajes para engañar a un endpoint para reinstalar las claves de sesión TLS,
es decir, un ataque de reinstalación de clave podría ser posible.
Consideramos interesante el trabajo futuro para determinar si esto lleva a ataques prácticos.

La reutilización de IVs también es un problema en el roto protocolo WEP [17, 18].
En particular, Borisov et al. descubrió que ciertas tarjetas de red inalámbricas inicializaban el WEP IV a cero cada vez que se (re) inicializaban.
En consecuencia, es probable que se reutilicen keystreams correspondientes a IV pequeños [18].
Sin embargo, a diferencia de los ataques de reinstalación de clave, estos reinicios IV no se pueden desencadenar de forma remota.

### 7.2 Wi-Fi y la Seguridad del Protocolo de Red
En uno de los primeros análisis formales del 4-way handshake, Él y Mitchell descubrieron una vulnerabilidad de denegación de servicio [38, 55].
Esto llevo a la estandarización de un 4-way handshake ligeramente mejorado [1].
En 2005, Él et al. presentó una prueba formal de corrección tanto del 4-way handshake como del handshake de clave grupal [39].
Sin embargo, no modelaron explícitamente la selección de cifrado y la protección de degradación.
Esto le permitió a Vanhoef y Piessens llevar a cabo un ataque de degradación contra el 4-way handshake [72].
En su ataque, se engaña al AP para que use RC4 para encriptar la clave grupal cuando se transporta en el mensaje 3.

Este ataque solo es posible si la red soporta WPA-TKIP, que ya se sabía que era un cifrado débil [66, 69].
Además, los modelos empleados en [39] no definen cuándo instalar la clave de sesión negociada o la clave de grupo transportada.
Sin embargo, mostramos que este momento es, de hecho, esencial, ya que de lo contrario podrían ser posibles los ataques de reinstalación.

El FT handshake se basa en el 4-way handshake [5], pero no hay un análisis de seguridad formal del mismo.
En cambio, los trabajos existentes se centran en el rendimiento del apretón de manos, por ejemplo [11, 46].
Varios trabajos estudian mecanismos de autenticación que negocian claves maestras (PMK) [19, 21, 59, 75].
Algunos de estos mecanismos se basan en establecer primero una sesión TLS segura [9].
Como resultado, los ataques recientes a TLS también afectan estos mecanismos, por ejemplo [10, 14, 15, 27, 62].
En este documento, no estudiamos los mecanismos que negocian las claves maestras, sino que nos centramos en los handshake que derivan nuevas claves de sesión a partir de una clave maestra negociada o precompartida.

Con respecto a los protocolos de confidencialidad de datos, Beck y Tews encontraron el primer ataque práctico a WPA-TKIP [66].
Mostraron cómo descifrar un pequeño paquete TKIP, recuperaron la clave MIC y, posteriormente, falsificaron los paquetes.
Su ataque fue mejorado aún más en varios trabajos [36, 67, 69, 70].
Los investigadores también atacaron la construcción débil de la clave por paquete de TKIP explotando prejuicios en RC4 [6, 57, 71].
Hoy en día, TKIP está en desuso por la Wi-Fi Alliance debido a sus problemas de seguridad [74].

Aunque CCMP recibió algunas críticas [60], se ha demostrado que proporciona garantías de seguridad similares a modos tales como OCB [42].
En [31], Fouque et al. discute ataques teóricos de falsificación de mensajes cuando los nonces se repiten en CCMP.

Se sabe que el cifrado GCM es débil cuando se usan etiquetas de autenticación cortas [29] y cuando se vuelven a usar nonces [43].
Böck et al. Investigua empíricamente la reutilización de nonce cuando se usa GCM en TLS [16], y descubrió varios servidores que reutilizan nonces.
Nuestro ataque a GCMP en 802.11 es único porque podemos controlar cuándo un endpoint reutiliza un nonce, y ya que GCMP usa la misma clave (de autenticación) en ambas direcciones de la comunicación.
Varios criptógrafos se refirieron recientemente a GCM como frágil [35, 56].

Finalmente, otros trabajos resaltaron problemas de seguridad en las implementaciones Wi-Fi o en las tecnologías circundantes.
Por ejemplo, se descubrieron fallas de diseño en Wi-Fi Protected Setup (WPS) [73], se encontraron vulnerabilidades en los controladores [13, 20], se encontró que los routers usaban claves precompartidas predecibles [45], y así sucesivamente.

## CONCLUSIÓN
A pesar de las pruebas de seguridad del 4-way handshake y el handshake de clave grupal, mostramos que ambos son vulnerables a ataques de reinstalación de clave.
Estos ataques no violan las propiedades de seguridad de las pruebas formales, pero resaltan limitaciónes en los modelos empleados por ellos.
En particular, los modelos no especifican cuando una clave debe ser instalada para ser usada por el protocolo de confidencialidad de datos.
Adicionalmente, mostramos que los handshake PeerKey y Fast BSS Transition son vulnerables a ataques de reinstalación de clave.

Todos los clientes Wi-Fi que probamos resultaron vulnerables a nuestro ataque contra el 4-way handshake. Esto le permite a un atacante repetir
tramas de difusión y multi-difusión. Cuando el 4-way o Fast BSS handshake es atacado, el impacto preciso depende del protocolo de confidencialidad siendo usado. Aunque en todos los casos, es posible descifrar tramas y por ende secuestrar conexiones TCP.
Esto permite la inyección de datos en conexiones HTTP no cifradas.
Además nuestro ataque contra Android 6.0 activo la instalación de una clave de sólo ceros, ignorando completamente cualquier garantía de seguridad.

Aún más preocupante, nuestro ataque de reinstalación de clave incluso ocurre de manera espontanea si ciertos mensajes del handshake se pierden
debido al ruido de fondo. Esto significa que bajo ciertas circunstancias, las implementaciones están reutilizando nonces incluso sin que un
atacante esté presente.

Una interesante dirección de investigación futura es determinar si otras implementaciones de protocolos son también vulnerables a ataques
de reinstalación de clave. Los protocolos que parecen particularmente vulnerables son aquellos que deben tener en cuenta que un mensaje podría perderse.
Después de todo estos protocolos están explicitamente diseñados para procesar tramas retransmitidas, y posiblemente están reinstalando claves mientras lo hacen.

## RECONOCIMIENTOS
Esta investigación está fundada parcialmente por el Fondo de Investigación KU Leuven y por el proyecto imec High Impact Initiative Distributed Trust.

## REFERENCIAS
1. IEEE Std 802.11. 2016. Wireless LAN Medium Access Control (MAC) and Physical Layer (PHY) Spec.
1. IEEE Std 802.11ac. 2013. Amendment 4: Enhancements for Very High Throughput for Operation in Bands below 6 GHz.
1. IEEE Std 802.11ad. 2012. Amendment 3: Enhancements for Very High Throughput in the 60 GHz Band.
1. IEEE Std 802.11i. 2004. Amendment 6: Medium Access Control (MAC) Security Enhancements.
1. IEEE Std 802.11r. 2008. Amendment 2: Fast Basic Service Set (BSS) Transition.
1. Nadhem J AlFardan, Daniel J Bernstein, Kenneth G Paterson, Bertram Poettering, and Jacob CN Schuldt. 2013. On the Security of RC4 in TLS.. In USENIX Security.
1. Wi-Fi Alliance. 2010. Hotspot 2.0 (Release 2) Technical Specification v1.1.0.
1. Apple. 2017. Wi-Fi network roaming with 802.11k, 802.11r, and 802.11v on iOS. (2017). Retrieved May 19, 2017 from https://support.apple.com/en-us/HT202628
1. N. Asokan, Valtteri Niemi, and Kaisa Nyberg. 2002. Man-in-the-Middle in Tunnelled Authentication Protocols. Cryptology ePrint Archive, Report 2002/163. (2002).
1. Nimrod Aviram, Sebastian Schinzel, Juraj Somorovsky, Nadia Heninger, Maik Dankel, Jens Steube, Luke Valenta, David Adrian, J Alex Halderman, Viktor Dukhovni, et al. 2016. DROWN: breaking TLS using SSLv2. In USENIX Security.
1. Sangeetha Bangolae, Carol Bell, and Emily Qi. 2006. Performance study of fast BSS transition using IEEE 802.11 r. In Proceedings of the 2006 international conference on Wireless communications and mobile computing.
1. Mihir Bellare and Phillip Rogaway. 1993. Entity authentication and key distribution. In Annual International Cryptology Conference.
1. Gal Beniamini. 2017. Over The Air: Exploiting Broadcom’s Wi-Fi Stack. (2017). Retrieved May 19, 2017 from https://googleprojectzero.blogspot.be/2017/04/overair-exploiting-broadcoms-wi-fi_4.html
1. Benjamin Beurdouche, Karthikeyan Bhargavan, Antoine Delignat-Lavaud, Cédric Fournet, Markulf Kohlweiss, Alfredo Pironti, Pierre-Yves Strub, and Jean Karim Zinzindohoue. 2015. A messy state of the union: Taming the composite state machines of TLS. In IEEE S&P.
1. Karthikeyan Bhargavan and Gaëtan Leurent. 2016. On the practical (in-) security of 64-bit block ciphers: Collision attacks on HTTP over TLS and OpenVPN. In CCS.
1. Hanno Böck, Aaron Zauner, Sean Devlin, Juraj Somorovsky, and Philipp Jovanovic. 2016. Nonce-Disrespecting Adversaries: Practical Forgery Attacks on GCM in TLS. In USENIX WOOT.
1. Nikita Borisov, Ian Goldberg, and David Wagner. 2001. Analysis of 802.11 Security, or Wired Equivalent Privacy Isn’t. In Mac Crypto Workshop.
1. Nikita Borisov, Ian Goldberg, and David Wagner. 2001. Intercepting mobile communications: the insecurity of 802.11. In MobiCom.
1. Sebastian Brenza, Andre Pawlowski, and Christina Pöpper. 2015. A practical investigation of identity theft vulnerabilities in eduroam. In WiSec.
1. Laurent Butti and Julien Tinnes. 2008. Discovering and exploiting 802.11 wireless driver vulnerabilities. Journal in Computer Virology 4, 1 (2008), 25–37.
1. Aldo Cassola, William Robertson, Engin Kirda, and Guevara Noubir. 2013. A Practical, Targeted, and Stealthy Attack Against WPA Enterprise Authentication. In NDSS Symp.
1. CERT/CC. 2017. Vulnerability Note VU#228519: WPA2 protocol vulnerabilities. (2017). http://www.kb.cert.org/vuls/id/228519
1. Alessandro Cimatti, Edmund Clarke, Enrico Giunchiglia, Fausto Giunchiglia, Marco Pistore, Marco Roveri, Roberto Sebastiani, and Armando Tacchella. 2002. Nusmv 2: An opensource tool for symbolic model checking. In International Conference on Computer Aided Verification. Springer.
1. Cisco. 2008. Wireless-G Exterior Access Point with Power Over Ethernet Business Series: User Guide. (2008). Retrieved May 17, 2017 from http://www.cisco.com/c/dam/en/us/td/docs/wireless/access_point/csbap/ wap200e/administration/guide/WAP200E_V10_UG_C_web.pdf
1. corbixgwelt. 2011. Timejacking & Bitcoin: The Global Time Agreement Puzzle. (2011). Retrieved May 13, 2017 from http://culubas.blogspot.be/2011/05/ timejacking-bitcoin_802.html
1. dd wrt. 2017. QCA Wireless Settings: Key Renewal Interval. (2017). Retrieved May 17, 2017 from https://www.dd-wrt.com/wiki/index.php/QCA_wireless_settings# Key_Renewal_Interval
1. Joeri De Ruiter and Erik Poll. 2015. Protocol state fuzzing of TLS implementations. In USENIX Security.
1. Morris Dworkin. 2007. Recommendation for block cipher modes of operation: Galois/Counter Mode (GCM) for confidentiality and authentication. In NIST Special Publication 800-38D.
1. Niels Ferguson. 2005. Authentication weaknesses in GCM. Comments submitted to NIST Modes of Operation Process (2005). Retrieved May 16, 2017 from http://csrc.nist.gov/groups/ST/toolkit/BCM/documents/comments/CWCGCM/Ferguson2.pdf
1. Scott Fluhrer, Itsik Mantin, and Adi Shamir. 2001. Weaknesses in the key scheduling algorithm of RC4. In SAC.
1. Pierre-Alain Fouque, Gwenaëlle Martinet, Frédéric Valette, and Sébastien Zimmer. 2008. On the Security of the CCM Encryption Mode and of a Slight Variant. In Applied Cryptography and Network Security.
1. Google. 2017. Codenames, Tags, and Build Numbers. (2017). Retrieved August 29, 2017 from https://source.android.com/source/build-numbers
1. Google. 2017. Dashboards: Platform Versions. (2 May 2017). Retrieved May 15, 2017 from https://developer.android.com/about/dashboards/index.html
1. Google Git. 2017. wpa supplicant 8. (2017). Retrieved May 15, 2017 from https://android.googlesource.com/platform/external/wpa_supplicant_8/+refs
1. Shay Gueron and Vlad Krasnov. 2014. The fragility of aes-gcm authentication algorithm. In 11th International Conference on Information Technology: New Generations (ITNG).
1. Finn M. Halvorsen, Olav Haugen, Martin Eian, and Stig F. Mjølsnes. 2009. An Improved Attack on TKIP. In NordSec.
1. B. Harris and R. Hunt. 1999. Review: TCP/IP security threats and attack methods. Computer Communications 22, 10 (1999), 885–897.
1. Changhua He and John C Mitchell. 2004. Analysis of the 802.1 i 4-Way Handshake. In WiSe. ACM.
1. Changhua He, Mukund Sundararajan, Anupam Datta, Ante Derek, and John C Mitchell. 2005. A modular correctness proof of IEEE 802.11i and TLS. In CCS.
1. Lieven Hollevoet. 2014. xAP and xPL Getting started. (2014). Retrieved August 29, 2017 from https://github.com/hollie/misterhouse/wiki/xAP-and-xPL---Gettingstarted
1. Yih-Chun Hu, Adrian Perrig, and David B Johnson. 2006. Wormhole attacks in wireless networks. IEEE journal on selected areas in communications (2006).
1. Jakob Jonsson. 2002. On the security of CTR+ CBC-MAC. In SAC.
1. Antoine Joux. 2006. Authentication failures in NIST version of GCM. Retrieved 8 May 2017 from http:// csrc.nist.gov/groups/ST/ toolkit/BCM/documents/ Joux_ comments.pdf (2006).
1. J. Klein. 2013. Becoming a time lord - implications of attacking time sources. In Shmoocon Firetalks.
1. Eduardo Novella Lorente, Carlo Meijer, and Roel Verdult. 2015. Scrutinizing WPA2 password generating algorithms in wireless routers. In USENIX WOOT.
1. Przemyslaw Machan and Jozef Wozniak. 2013. On the fast BSS transition algorithms in the IEEE 802.11 r local area wireless networks. Telecommunication Systems (2013).
1. Aanchal Malhotra, Isaac E Cohen, Erik Brakke, and Sharon Goldberg. 2016. Attacking the Network Time Protocol. (2016).
1. Aanchal Malhotra and Sharon Goldberg. 2016. Attacking NTP’s Authenticated Broadcast Mode. ACM SIGCOMM Computer Communication Review (2016).
1. Jouni Malinen. 2015. 802.11e support? (2015). Retrieved May 17, 2017 from http://lists.shmoo.com/pipermail/hostap/2015-June/032952.html
1. Jouni Malinen. 2015. Fix TK configuration to the driver in EAPOL-Key 3/4 retry case. Hostap commit ad00d64e7d88. (1 Oct. 2015).
1. David McGrew. 2013. IETF Internet Draft: Generation of Deterministic Initialization Vectors (IVs) and Nonces. (2013). Retrieved August 29, 2017 from https://tools.ietf.org/html/draft-mcgrew-iv-gen-03
1. Microsoft. 2017. Fast Roaming with 802.11k, 802.11v, and 802.11r. (2017). Retrieved May 19, 2017 from https://docs.microsoft.com/en-us/windows-hardware/drivers/ network/fast-roaming-with-802-11k--802-11v--and-802-11r
1. D. Mills, J. Martin, J. Burbank, and W. Kasch. 2010. Network Time Protocol Version 4: Protocol and Algorithms Specification.
1. David L Mills. 2011. Computer network time synchronization (2 ed.). CRC Press.
1. John Mitchell and Changhua He. 2005. Security Analysis and Improvements for IEEE 802.11i. In NDSS.
1. Kenneth G. Paterson. 2015. Countering Cryptographic Subversion. (2015). Retrieved May 16, 2017 from https://hyperelliptic.org/PSC/slides/paterson-PSC.pdf
1. Kenneth G. Paterson, Bertram Poettering, and Jacob C. N. Schuldt. 2014. Plaintext Recovery Attacks Against WPA/TKIP. In FSE.
1. Grand View Research. 2017. Wireless Gigabit (WiGig) Market Size To Reach $7.42 Billion By 2024. (2017). Retrieved May 10, 2017 from http://www. grandviewresearch.com/press-release/global-wireless-gigabit-wigig-market
1. Pieter Robyns, Bram Bonné, Peter Quax, and Wim Lamotte. 2014. Short paper: exploiting WPA2-enterprise vendor implementation weaknesses through challenge response oracles. In WiSec.
1. P. Rogaway and D. Wagner. 2003. A Critique of CCM. Cryptology ePrint Archive, Report 2003/070. (2003).
1. J. Selvi. 2015. Breaking SSL using time synchronisation attacks. In DEF CON Hacking Conference.
1. Juraj Somorovsky. 2016. Systematic Fuzzing and Testing of TLS Libraries. In CCS.
1. Robert Stacey, Adrian Stephens, Jesse Walker, Herbert Liondas, and Emily Qi. 2010. Rekeying Protocol Fix. (2010). Retrieved August 19, 2017 from https:// mentor.ieee.org/802.11/dcn/10/11-10-0313-01-000m-rekeying-protocol-fix.ppt
1. Robert Stacey, Adrian Stephens, Jesse Walker, Herbert Liondas, and Emily Qi. 2010. Rekeying Protocol Fix Text. (2010). Retrieved August 19, 2017 from https://mentor. ieee.org/802.11/dcn/10/11-10-0314-00-000m-rekeying-protocol-fix-text.doc
1. Adam Stubblefield, John Ioannidis, Aviel D Rubin, et al. 2002. Using the Fluhrer, Mantin, and Shamir Attack to Break WEP. In NDSS.
1. Erik Tews and Martin Beck. 2009. Practical attacks against WEP and WPA. In WiSec.
1. Yosuke Todo, Yuki Ozawa, Toshihiro Ohigashi, and Masakatu Morii. 2012. Falsification Attacks against WPA-TKIP in a Realistic Environment. IEICE Transactions (2012).
1. Mathy Vanhoef. 2017. Chromium Bug Tracker: WPA1/2 all-zero session key & key reinstallation attacks. (2017). Retrieved August 29, 2017 from https:// bugs.chromium.org/p/chromium/issues/detail?id=743276
1. Mathy Vanhoef and Frank Piessens. 2013. Practical verification of WPA-TKIP vulnerabilities. In ASIA CCS. ACM, 427–436.
1. Mathy Vanhoef and Frank Piessens. 2014. Advanced Wi-Fi attacks using commodity hardware. In ACSAC.
1. Mathy Vanhoef and Frank Piessens. 2015. All your biases belong to us: Breaking RC4 in WPA-TKIP and TLS. In USENIX Security.
1. Mathy Vanhoef and Frank Piessens. 2016. Predicting, Decrypting, and Abusing WPA2/802.11 Group Keys. In USENIX Security.
1. Stefan Viehböck. 2011. Brute forcing Wi-Fi protected setup. (2011). Retrieved May 9, 2017 from http://packetstorm.foofus.com/papers/wireless/viehboeck_wps.pdf
1. Wi-Fi Alliance. 2015. Technical Note: Removal of TKIP from Wi-Fi Devices.
1. Joshua Wright. 2003. Weaknesses in LEAP challenge/response. In DEF CON Hacking Conference.
1. Erik Zenner. 2009. Nonce Generators and the Nonce Reset Problem. In International Conference on Information Security.
