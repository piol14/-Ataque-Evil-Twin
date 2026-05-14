# Ataque Evil Twin contra WPA2 Enterprise

##  Definición
Ataque que consiste en crear un punto de acceso Wi-Fi fraudulento que parece ser una red legítima realizando una clonacion de su SSID. El objetivo es engañar a los dispositivos cercanos para que se conecten automáticamente al atacante. Se utiliza para capturar hashes de autenticación (MS-CHAPv2).

---

##  Funcionamiento

1.  **Reconocimiento y Clonación:** El atacante identifica una red objetivo y configura un punto de acceso propio con el mismo nombre (SSID) y, opcionalmente, la misma dirección física (MAC). Se suele configurar con una potencia de señal mayor para que los dispositivos lo prefieran frente al router original.
2.  **Negociación de Autenticación:** Cuando la víctima intenta conectar, el "Gemelo Malvado" (usando herramientas como `hostapd-wpe`) no intenta validar la clave contra una base de datos real, sino que acepta cualquier intento para forzar el intercambio de los valores del protocolo (en este caso, MS-CHAPv2).
3.  **Captura del Handshake/Hash:** Durante el proceso de conexión, el atacante captura el desafío (challenge) y la respuesta (response) que el dispositivo de la víctima genera a partir de su contraseña. Estos datos quedan registrados en los logs del atacante en forma de hash.
4.  **Cracking Offline:** Una vez obtenido el hash, el ataque Wi-Fi termina y el atacante procede a descifrar la contraseña en su propio equipo usando `hashcat` para descubrir la contraseña.

---

##  Ataque realizado

### 1. Escenario
* **Máquina atacante:** Kali Linux
* **Máquina víctima:** Movil Samsung
* **Antena**

### 2. Ataque: Pasos y Archivo de configuración

#### - Paso 1: Preparación del Hardware
Conectar la antena: Verificamos con `lsusb` que la antena esta conectada a la Kali atacante:

![lsusb](https://github.com/user-attachments/assets/8d85c9ca-8b86-4245-bbe3-f67b8940bd18)

#### - Paso 2: Modo Monitor
Ponemos la antena en modo monitor con el comando `sudo airmon-ng start wlan0`. Esto permite que la tarjeta "escuche" todos los paquetes que viajan por el aire y, posteriormente, emita su propia red.

![airmon-ng start](https://github.com/user-attachments/assets/f68cee60-266c-4fda-8eda-8aa4beec2471)

#### - Paso 3: Limpieza de procesos
`sudo airmon-ng check kill`
Este comando detiene NetworkManager y wpa_supplicant. Esto sirve para evitar que la red falsa podría caerse constantemente o cambiar de canal sin aviso.

![check kill](https://github.com/user-attachments/assets/6791f33d-ddd9-412e-a439-5789cd122096)

#### - Paso 4: Identificación del Objetivo
Con `sudo airodump-ng wlan0` escaneamos el aire para ver qué redes hay cerca. Aquí identifico el SSID (nombre) que quieres clonar y el Canal en el que está operando la red legítima. Identificamos la red **CETI** con el **canal 1**.

![airodump-ng](https://github.com/user-attachments/assets/8c32df49-d4eb-41ab-972d-b740d2d6b626)

#### - Paso 5: Configuración de hostapd-wpe
Modificamos el archivo `/etc/hostapd-wpe/hostapd-wpe.conf` y pondremos lo siguiente:
* Interface: `wlan0`
* SSID: `CETI`
* channel: `1`

![config file](https://github.com/user-attachments/assets/35e0b20b-11b0-4e91-8200-edab40275370)

#### - Paso 6: Lanzamiento del Evil Twin
Lanzamos el ataque con `sudo hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf`, creando una web falsa para que la victima intente conectarse:

![lanzamiento](https://github.com/user-attachments/assets/61cf763b-18f5-413b-b9f8-5a2ddd13056f)

#### - Paso 7: Interacción de la Víctima
Vamos al movil victima y podremos ver que hay 2 redes CETI:

<img src="https://github.com/user-attachments/assets/5d699bc0-7f9f-4b3b-a563-d4fc516ca73f" width="30%" />

Ponemos el usuario y contraseña que en este caso sera `ceti` `ceti`:

<img src="https://github.com/user-attachments/assets/c266f8b6-09cf-4bae-b1e0-03d04804866d" width="30%" />

Dara error de conexión ya que esta red no tiene salida a internet:

<img src="https://github.com/user-attachments/assets/51042f71-4e85-4aa0-a900-e8aa17666459" width="30%" />

#### - Paso 8: Captura del Handshake
Volviendo a la Kali atacante podremos ver que esta ha capturado el nombre del usuario y el hash de la contraseña:

![captura hash](https://github.com/user-attachments/assets/6a827c85-1835-4f7a-99a9-15fec7e495d1)

#### - Paso 9: Crackeo del hash
Con el comando `hashcat -m 5500 -O "ceti::::3bfe0ba926296a303838784b198430e75d02059ac8f8e4db:586c7468948ace3b" pass.txt --outfile resultado.txt` usamos la herramienta hashcat para descubrir la contraseña mediante el diccionario `pass.txt` que incluye contraseñas entre ellas, la de ceti y redigimos el resultado a un txt llamado `resultado.txt`.

![hashcat](https://github.com/user-attachments/assets/02833e5a-b816-4cb5-86d3-8145f2c09a29)

Abrimos el txt con resultado y podremos ver el usuario, el hash y tras el hash la contraseña:

![resultado final](https://github.com/user-attachments/assets/7370a817-be92-45b0-9d46-112fbb1db070)
