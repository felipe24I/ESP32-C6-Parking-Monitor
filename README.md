# ESP32C6 Parking Monitor

**Autor:** Felipe Idárraga Quintero

**Nombre de la Práctica:** Práctica 1

**Curso:** Desarrollo de Sistemas IoT

**Departamento:** Departamento de Ingeniería Electrica, Electronica y Computacion

---

## Descripción de la práctica

Este repositorio contiene una práctica titulada **ESP32C6 Parking Monitor**, cuyo objetivo es implementar un sistema básico de monitoreo de parqueaderos usando un **ESP32-C6**. 

En este proyecto los sensores de parqueo no son físicos, sino que se simulan en el firmware: el ESP32-C6 genera de forma aleatoria (con una probabilidad configurada) si cada puesto está libre u ocupado, imitando la entrada y salida de vehículos. Estos estados se envían por MQTT y se muestran en una página web sencilla que permite ver en tiempo real la ocupación del parqueadero.

Este trabajo es una adaptación del proyecto original **“Proyecto IoT: ESP32 LED RGB con MQTT y Servidor Web”** de los autores **Edward Fabian Goyeneche Velandia** y **Juan Sebastian Giraldo Duque**, que se encuentra en https://github.com/Edward1304/Demo_IoT_MQTT_SERVER. 

A partir de esa base se realizaron cambios en la lógica, en el manejo de MQTT y en la interfaz web para orientarlo al monitoreo de parqueaderos con ESP32-C6 y sensores simulados.

## Componentes de la práctica

- **ESP32-C6-DevKitC-1 v1.2** con LED RGB WS2812 integrado
- **Servidor AWS EC2** con Ubuntu
- **Broker MQTT Mosquitto** con soporte WebSocket
- **Servidor Web Apache** 
- **Página Web** con visualización en tiempo real

##  Arquitectura IoT

![parking_monitor (2)](https://github.com/user-attachments/assets/ab077ebc-2ec0-4298-a759-c949b297178a)

En la arquitectura IoT, el ESP32-C6 actúa como cliente MQTT conectado por WiFi al router, el cual da acceso a un servidor AWS EC2 que aloja el broker Mosquitto y el servidor web Apache2. El dashboard web se sirve vía HTTP y se comunica en tiempo real con el broker mediante MQTT sobre WebSocket

## Implementación del sistema

### 1. Creación y configuración del servidor EC2 en AWS

**1.1 Creación de la cuenta y acceso a EC2**

Primero, ingresa a **Amazon Web Services (AWS)** y crea una cuenta (puede ser con la capa gratuita).  
Luego accede a la **AWS Management Console** y, en la barra de búsqueda, escribe **“EC2 (Amazon Elastic Compute Cloud)”**. 

![AWS EC2 Instance](./img/1.png)

Dentro del servicio EC2 selecciona la opción **“Launch instance”** para crear una nueva instancia.

![Lanzar instancia](./img/2.png)

**1.2 Configuración de la instancia EC2**

Al crear la instancia, se le puede asignar un nombre descriptivo, por ejemplo **esp-webserver**.  
Como **Amazon Machine Image (AMI)** se selecciona **Ubuntu Server 22.04 LTS**, y como **Instance Type** se elige **t3.micro**, que es compatible con la capa gratuita de AWS y suficiente para alojar el broker MQTT (Mosquitto) y el servidor web (Apache2) de esta práctica.

![Nombre server](./img/3.png)

**1.3 Configuración del grupo de seguridad (Security Group)**

En **Network settings**, selecciona la opción **“Create security group”** y configura las siguientes reglas de entrada:

- **SSH** desde `0.0.0.0/0`: permite conectarse por SSH a la instancia desde cualquier dirección IP.  

- **HTTP** desde Internet: permite acceder al servidor web por el puerto 80.

- **HTTPS** desde Internet: permite acceder por el puerto 443 (si se desea acceder al servidor web por este puerto).

![Grupo de seguridad 1](./img/4.png)

A continuación, haz clic en **“Add security group rule”** para agregar los puertos necesarios para MQTT:

- **Puerto 1883 (TCP)**: se utiliza para la conexión MQTT del **ESP32-C6** hacia el broker Mosquitto.
- **Puerto 9001 (TCP)**: se utiliza para la conexión del **dashboard web** mediante **MQTT sobre WebSocket**.

En ambos casos se puede dejar el origen como `0.0.0.0/0` para permitir el acceso desde cualquier IP

![Grupo de seguridad 2](./img/5.png)

![Grupo de seguridad 3](./img/6.png)

**1.4 Configuración del par de claves (Key pair)**

En el apartado **“Key pair (login)”** se debe crear o seleccionar un **par de claves (key pair)**. 

Un key pair es un mecanismo de autenticación que utiliza una **clave pública** (que queda almacenada en la instancia EC2) y una **clave privada** (que se descarga en tu computador) para permitir el acceso seguro por SSH, sin necesidad de usar una contraseña.

Al crear un nuevo key pair, AWS generará un archivo con extensión **`.pem`**, el cual se descargará automaticamente y se debe guardar en un lugar seguro.  

Este archivo `.pem` será necesario si se va a acceder a la instancia EC2 como cliente **SSH** desde **Windows PowerShell** (o desde cualquier otro cliente SSH).

![Key-par 1](./img/7.png)

 ![Key-par 2](./img/8.png)

 ![Key-par 3](./img/10.png)

### 1.5 Inicio de la instancia

**1.5 Inicio de la instancia**

Finalmente, después de revisar la configuración anterior, se hace clic en **“Launch instance”** para crear la máquina virtual.  

![Launch instance 1](./img/9.png)

Luego, en la sección “Instances”, se selecciona (se marca) la instancia creada y, si es necesario, se elige la opción **“Instance state” → “Start instance”** para asegurarse de que esté encendida

Desde el panel de EC2 se verifica que el estado de la instancia cambie a **“running”**, lo que indica que el servidor está encendido y listo para aceptar conexiones.

<img width="1919" height="500" alt="image" src="https://github.com/user-attachments/assets/40e76b7d-0bcd-4df7-8daa-f9421dff43d0" />

Una vez la instancia está en ejecución, ya es posible acceder por **SSH** al servidor y continuar con la configuración del **broker MQTT (Mosquitto)** dentro de esta máquina virtual.

## 2. Configuración del broker MQTT dentro del servidor AWS (EC2)

**2.1 Acceso por SSH a la instancia EC2**

Desde **Windows**, abre **PowerShell** en la carpeta donde guardaste el archivo `.pem` del key pair. 

![ssh connect](./img/12.png)

Una vez la instancia está en estado **“running”**, en el panel de EC2 se hace clic en el botón **“Connect”**.  
En la ventana que se abre, se selecciona la opción **“SSH client”** y, en la parte inferior, en la sección **“Example”**, aparece el comando completo para conectarse por SSH a la instancia. Es un comando similar a:

```bash
ssh -i "NOMBRE_DE_TU_CLAVE.pem" ubuntu@IP_PUBLICA_DE_LA_INSTANCIA
```

![ip-public](./img/13.png)

![ip-public](./img/14.png)

Este comando se debe copiar y ejecutar en Windows PowerShell, ubicándose primero en la carpeta donde se encuentra el archivo .pem.

![ip-public](./img/15.png)

De esta forma, se establece la conexión SSH con el servidor Ubuntu que está corriendo en la instancia EC2.

![ip-public](./img/16.png)

**2.2 Instalación de apache2**

```bash
sudo apt update
sudo apt install apache2
```

Se debe asegúrar de que Apache esté en ejecución. Puede verificar su estado con el siguiente comando:

```bash

sudo systemctl status apache2

```

![Untitled](./img/18.png)

Si Apache no se está ejecutando, se debe iniciar el servicio con:

```bash

sudo systemctl start apache2

```

**2.3 Instalación de Mosquitto**

Los siguientes comandos actualizarán la lista de paquetes disponibles y luego instalarán el servidor Mosquitto y la utilidad de línea de comandos Mosquitto Clients.

```bash

sudo apt update
sudo apt install mosquitto mosquitto-clients

```
Una vez que Mosquitto esté instalado, se puede habilitar el servicio y se debe de asegurar de que se inicie automáticamente al arrancar el sistema con los siguientes comandos:

```bash

sudo systemctl enable mosquitto
sudo systemctl start mosquitto

```

Para verificar si Mosquitto se está ejecutando correctamente ejecutando el siguiente comando:

```bash

sudo systemctl status mosquitto

```

Se deberia ver un mensaje que indique que el servicio está activo y en funcionamiento.

![Untitled](./img/22.png)

**2.4 Configuración de los puertos 1883 y 9001 (WebSocket)**

Por defecto, Mosquitto suele escuchar en el puerto 1883 (MQTT sobre TCP).
Para esta práctica también se configurará el puerto 9001 para MQTT sobre WebSocket, que usará el dashboard web.

Primero, se edita el archivo de configuración (por ejemplo):

```bash

sudo nano /etc/mosquitto/conf.d/mosquitto.conf

```

Y se agrega el siguiente contenido básico:

```bash

# Listener MQTT normal (ESP32-C6)

listener 1883

protocol mqtt


# Listener MQTT sobre WebSocket (dashboard web)

listener 9001

protocol websockets


```

Se guarda el archivo (Ctrl + O, Enter) y se sale del editor (Ctrl + X).

**2.5 Crear un archivo de contraseñas:**

- Abrir una terminal y ejecuta el siguiente comando para crear un archivo que almacene los usuarios y contraseñas:

```bash

sudo mosquitto_passwd -c /etc/mosquitto/passwd <nombre-de-usuario>

```

Reemplazar **`<nombre-de-usuario>`** con el nombre de usuario que  se desee. Será solicitado a ingresar una contraseña para ese usuario.

![Untitled](./img/24.png)

```bash
sudo chown mosquitto:mosquitto /etc/mosquitto/passwd
sudo chmod 640 /etc/mosquitto/passwdclear
```

**2.6 Reiniciar el servidor Mosquitto para que los cambios surtan efecto:**

Finalmente, se reinicia Mosquitto para que tome la nueva configuración:

```bash

sudo systemctl restart mosquitto

```

Ahora, cuando te conectes al servidor MQTT Mosquitto, deberás proporcionar un nombre de usuario y contraseña válidos para autenticarte.

En este punto, el servidor AWS EC2 ya tiene un broker MQTT escuchando en:

- 1883/TCP para el ESP32-C6.

- 9001/TCP (WebSocket) para el dashboard web.

**2.7 Prueba el Servidor**

Puedes utilizar el cliente Mosquitto para probar la funcionalidad del servidor MQTT. Abre una terminal y utiliza el siguiente comando para suscribirte a un tema y ver los mensajes que llegan:

```bash

mosquitto_sub -h localhost -t test -u esp32 -P esp32
musqu

```

Abrir otra terminal y publica un mensaje en el mismo tema:

```bash

mosquitto_pub -h localhost -t test -m "Hola, MQTT!" -u esp32 -P esp32

```

Deberías ver el mensaje "Hola, MQTT!" en la terminal donde te suscribiste al tema.

