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

## Ataque realizado  

### 1. Escenario
- **Máquina atacante:** Kali Linux  
- **Máquina víctima:** Movil Samsung  
- **Antena**

### 2. Ataque: Pasos y Archivo de configuración 

- Conectar la antena: Verificamos con lsusb que la antena esta conectada a la Kali atacante:
<img width="918" height="122" alt="image" src="https://github.com/user-attachments/assets/8d85c9ca-8b86-4245-bbe3-f67b8940bd18" />
- Ponemos la antena en modo monitor con el comando sudo airmon-ng start wlan0. Esto permite que la tarjeta "escuche" todos los paquetes que viajan por el aire y, posteriormente, emita su propia red.
<img width="886" height="329" alt="image" src="https://github.com/user-attachments/assets/f68cee60-266c-4fda-8eda-8aa4beec2471" />

-  Limpieza de procesos :  sudo airmon-ng check kill
Este comando detiene NetworkManager y wpa_supplicant. Esto sirve para evitar que  la red falsa podría caerse constantemente o cambiar de canal sin aviso
<img width="319" height="150" alt="image" src="https://github.com/user-attachments/assets/6791f33d-ddd9-412e-a439-5789cd122096" />

- Identificación: Con sudo airodump-ng wlan0 escaneamos el aire para ver qué redes hay cerca. Aquí identifico el SSID (nombre) que quieres clonar y el Canal en el que está operando la red legítima. Identificamos la red CETI con el canal 1
<img width="838" height="421" alt="image" src="https://github.com/user-attachments/assets/8c32df49-d4eb-41ab-972d-b740d2d6b626" />
-  Configuración del ataque con hostapd-wpe: : Modificamos el archivo /etc/hostapd-wpe/hostapd-wpe.conf y pondremos lo siguiente:
Interface: wlan0
SSID: CETI
channel: 1
<img width="933" height="353" alt="image" src="https://github.com/user-attachments/assets/ba3aba2c-79bf-4ff3-a77f-abc14cfda031" />

