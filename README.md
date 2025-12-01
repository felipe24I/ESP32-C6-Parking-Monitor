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

Finalmente, después de revisar la configuración anterior, se procede a **iniciar la instancia EC2**.  

Desde el panel de EC2 se verifica que el estado de la instancia cambie a **“running”**, lo que indica que el servidor está encendido y listo para aceptar conexiones.

Una vez la instancia está en ejecución, ya es posible acceder por **SSH** al servidor y continuar con la configuración del **broker MQTT (Mosquitto)** dentro de esta máquina virtual.

![Launch instance 1](./img/9.png)

![Launch instance 2](./img/11.png)

