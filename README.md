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

**1.1 Ingreso y creación de la instancia EC2 en AWS**

Primero, ingresa a **Amazon Web Services (AWS)** y crea una cuenta (puede ser con la capa gratuita).  
Luego accede a la **AWS Management Console** y, en la barra de búsqueda, escribe **“EC2 (Amazon Elastic Compute Cloud)”**.  
Dentro del servicio EC2 selecciona la opción **“Launch instance”** para crear una nueva instancia.

