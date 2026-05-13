# Ataque-Evil-Twin
 Ataque Evil Twin contra WPA2 Enterprise
## Definición 
Ataque que consiste en crear un punto de acceso Wi-Fi fraudulento que parece ser una red legítima realizando una clonacion de su SSID. El objetivo es engañar a los dispositivos cercanos para que se conecten automáticamente al atacante. Se utiliza para realizar ataques de Man-in-the-Middle (MITM), capturar hashes de autenticación (MS-CHAPv2) o realizar phishing mediante portales cautivos.


## Funcionamiento 

-  Reconocimiento y Clonación: El atacante identifica una red objetivo y configura un punto de acceso propio con el mismo nombre (SSID) y, opcionalmente, la misma dirección física (MAC). Se suele configurar con una potencia de señal mayor para que los dispositivos lo prefieran frente al router original.

- Desautenticación : Se envían paquetes de desautenticación a los clientes conectados a la red real. Esto provoca que los dispositivos se desconecten y, al intentar reconectar, seleccionen automáticamente la red del atacante por ser la más cercana o potente.

- Negociación de Autenticación: Cuando la víctima intenta conectar, el "Gemelo Malvado" (usando herramientas como hostapd-wpe) no intenta validar la clave contra una base de datos real, sino que acepta cualquier intento para forzar el intercambio de los valores del protocolo (en este caso, MS-CHAPv2).

- Captura del Handshake/Hash: Durante el proceso de conexión, el atacante captura el desafío (challenge) y la respuesta (response) que el dispositivo de la víctima genera a partir de su contraseña. Estos datos quedan registrados en los logs del atacante en forma de hash.

- Cracking Offline: Una vez obtenido el hash, el ataque Wi-Fi termina y el atacante procede a descifrar la contraseña en su propio equipo usando haschat para descubrir la contraseña.
