## FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N°: 1 - Diagramas de Estado - Modelado
### Archivo: tdse-tp1_00-problem_approach.md

---

## 1. Solución de COMA Electronics:

El documento toma como modelo comercial de referencia el Intelligent Parking Management System de la firma COMA Electronics. Esta solución integra los siguientes componentes principales y flujo de trabajo:

## Arquitectura General del Sistema
   La infraestructura del sistema de estacionamiento está compuesta por:
   - Servidor Central (Parking System Server): Centraliza el control y procesamiento de datos.
   - Terminales de Acceso: Módulos de entrada (Entry Machine) y de salida (Exit Machine).
   - Estaciones de Cobro: Computadora de peaje (Toll Computer) y/o Estación de Pago Automático (Automatic Pay Station).

## Flujo Operativo del Sistema (Automated Parking System)
   El ciclo completo de control de acceso y cobro comprende tres fases:
   - Ingreso: El vehículo se aproxima a la terminal de entrada. Al presionar el botón (Ticket Button), la máquina emite un ticket o tarjeta con un número de serie, fecha y hora, y envía la señal de apertura a la barrera de acceso (Barrier gate). El área de entrada integra cámaras, el dispenser de tickets, escáner y bobinas sensoras de presencia (sensor coils).
   - Estacionamiento y Pago: El cliente estaciona su vehículo y, antes de regresar a él, efectúa el pago en la caja central o en la estación automática de cobro (ej. P20 Automatic Pay Station). Al pagar, el ticket o tarjeta queda validado para la salida con un tiempo de gracia preestablecido.
   - Egreso: El vehículo se presenta en la terminal de salida, donde el escáner lee el ticket validado y envía la señal para abrir la barrera de salida.

## Componentes de la Terminal de Entrada (Parking Ticket Dispenser Machine - Entry)
La terminal física de entrada de COMA incluye:

   - Pantalla LCD de 7" e indicador por voz (Voice Prompt).
   - Botón de ticket (Ticket Button) y ranura de emisión (Ticket Slot).
   - Lector de tarjetas (Card Reader) y botón de ayuda (Help Button).
   - Cámara motorizada con luz automática e intercomunicador de video (opcional).
   - Barrera de alta velocidad activada por radar.
   - Visualizador LED de plazas vacantes (por ejemplo: "Vacant: 128")

## 2. Implementación de la Parking Ticket Dispenser Machine (Entry):

## Arquitectura Modular (Escrutar, Procesar, Actuar)

El comportamiento del sistema embebido se desglosa en una estructura modular organizada en tres capas:
### 1. Escrutar (Scrutinize / Sensores - Digital Inputs):
   Encargado de capturar el estado de los elementos de entrada: la cámara (Camera), el pulsador de ticket (Button) y el detector de presencia vehicular (Sensor Coil).

### 2. Procesar (Process / Sistema - System):
   Gestiona la lógica de control y la secuencia de estados del prototipo: llegada del vehículo (Car arrives) $\rightarrow$ mensaje de bienvenida (Welcome) $\rightarrow$ presión del botón (Button is pressed) $\rightarrow$ impresión del ticket (Print ticket) $\rightarrow$ apertura de barrera (Open barrier) $\rightarrow$ vehículo ingresado (Car inside) $\rightarrow$ cierre de barrera (Close barrier) $\rightarrow$ egreso del área de entrada (Car leaves).

### 3.Actuar (Act / Actuadores - Digital Outputs):
   Maneja las señales enviadas a las salidas físicas: pantalla (Display), impresora (Printer), barrera de acceso (Barrier) y servidor (Server).

## Sustitución para Prototipado

En caso de no disponer de los sensores o actuadores físicos reales para las pruebas del MVP, la especificación permite reemplazarlos por componentes discretos simples:

   - Entradas (Sensores): La cámara (Camera) y la bobina sensora (Sensor coil) se representan mediante llaves de tipo On/Off, mientras que el botón (Button) se implementa con un pulsador.
   - Salidas (Actuadores): La barrera (Barrier) se simula mediante un indicador LED.

## Sincronización y modelo de ejecución

   - Comunicación por Mensajes: La interacción y sincronización entre los módulos de sensores, sistema y actuadores se realiza exclusivamente mediante el intercambio de mensajes (Messages)
   - Ejecutivo Cíclico No Bloqueante (Cyclic Executive):
        - La ejecución se realiza mediante un ciclo no bloqueante con un período de 1 ms (each 1mS).
        - En cada iteración de 1 ms, el sistema ejecuta secuencialmente: la revisión de cambios en los sensores (y emisión de mensajes), la lectura/procesamiento de mensajes en el sistema (y generación de nuevos mensajes) y la actualización de los actuadores en función de los mensajes recibidos.
        - Regla de diseño clave: Debe garantizarse un comportamiento comunitario donde ningún módulo se apropie del uso del microprocesador; el uso de código bloqueante es inaceptable.

## 3. Modelos para describir el comportamiento de cada uno de los módulos de código en C:

### 1. Escrutar $\rightarrow$ Sensor (Digital Inputs)
   Encargado de inspeccionar las entradas digitales (cámara, botón y bobina sensora).
   
   Algoritmo de ejecución (cada 1 ms):
   
   - Recorre iterativamente cada sensor desde 1 hasta N (Sensor (from 1 to N)).
   - Evalúa si ocurrió algún cambio en el estado del sensor (Any Change?).
   - Si se detecta un cambio, genera y deposita un mensaje (Put Message).   
   - Verifica si se evaluó el último sensor (Last Sensor?) para finalizar el ciclo de escrutinio.

### 2.  Procesar $\rightarrow$ System (Interface / System)
   Encargado de la lógica de control del sistema y la gestión de la máquina de estados.
   
   Algoritmo de ejecución (cada 1 ms):
   
      - Comprueba si existe algún mensaje entrante proveniente de los sensores (Any Message?).
      - Si hay un mensaje, lo lee y carga (Load Message).
      - Procesa el mensaje y determina si produce una transición o cambio en el estado del sistema (Any Change?).
      - Si corresponde un cambio, genera y deposita un mensaje saliente destinado a los actuadores (Put Message).

### 3. Actuar $\rightarrow$ Actuator (Digital Outputs)
   Encargado de modificar el estado de los componentes físicos de salida (pantalla, impresora, barrera, servidor).
   
   Algoritmo de ejecución (cada 1 ms):
   
      - Recorre iterativamente cada actuador desde 1 hasta N (Actuator (from 1 to N)).
      - Verifica si hay algún mensaje de comando destinado a ese actuador (Any Message?).
      - Si existe el mensaje, lo carga (Load Message).
      - Determina si el mensaje exige modificar el estado de la salida (Any Change?).
      - En caso afirmativo, ejecuta la acción física sobre la salida digital (Make Action).
      - Comprueba si se alcanzó el último actuador (Last Actuator?) para concluir la tarea.

