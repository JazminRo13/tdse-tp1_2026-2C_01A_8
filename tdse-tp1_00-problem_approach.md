## FIUBA - Electrónica - Taller de Sistemas Embebidos

## Trabajo Práctico N° 1 - Diagramas de Estado - Modelado

### Archivo: `tdse-tp1_00-problem_approach.md`

---

## 1. Solución de COMA Electronics

Como referencia para el desarrollo del proyecto se toma el **Intelligent Parking Management System** de COMA Electronics, un sistema destinado a automatizar la gestión de estacionamientos.

La solución está compuesta principalmente por:

* **Parking System Server:** servidor encargado de centralizar la información y gestión del estacionamiento.
* **Entry Machine:** terminal encargada de gestionar el ingreso de vehículos.
* **Exit Machine:** terminal encargada de gestionar el egreso de vehículos.
* **Toll Computer / Automatic Pay Station:** dispositivos destinados a gestionar el cobro.

Dentro de esta solución se toma como referencia la **Parking Ticket Dispenser Machine (Entry)**, correspondiente a la terminal de entrada. Esta permite detectar la llegada de un vehículo, interactuar con el usuario, emitir un ticket y controlar la apertura de la barrera para permitir el ingreso al estacionamiento.

---

## 2. Implementación de la Parking Ticket Dispenser Machine (Entry)

Para el proyecto se propone implementar un **Producto Mínimo Viable (MVP)** de la terminal de entrada.

El comportamiento del sistema se divide en tres módulos de código C:

* **Sensor:** encargado de **escrutar** las entradas del sistema, como `Camera`, `Button` y `Sensor Coil`.
* **System:** encargado de **procesar** los eventos recibidos desde los sensores y determinar las acciones que debe realizar el sistema.
* **Actuator:** encargado de **actuar** sobre las salidas, como `Display`, `Printer`, `Barrier` y `Server`.

Por lo tanto, la arquitectura general implementada es:

**Sensor (Escrutar) → System (Procesar) → Actuator (Actuar)**

Los módulos se comunican y sincronizan mediante **mensajes**.

---

## 3. Modelos de comportamiento de los módulos

Los tres módulos se implementan como módulos de código C temporizados mediante:

**Update by Time Code, period = 1 ms**

### Sensor → Escrutar

El módulo `Sensor` se ejecuta periódicamente cada 1 ms y se encarga de revisar el estado de las entradas digitales.

* Recorre los sensores.
* Comprueba si ocurrió algún cambio (`Any Change?`).
* Si detecta un cambio, genera el mensaje correspondiente (`Put Message`).
* Continúa hasta verificar el último sensor (`Last Sensor?`).

### System → Procesar

El módulo `System` se ejecuta periódicamente cada 1 ms y contiene la lógica de control del sistema.

* Comprueba si existe un mensaje (`Any Message?`).
* Si existe, carga el mensaje (`Load Message`).
* Procesa el evento recibido.
* Determina si debe producirse algún cambio (`Any Change?`).
* Si corresponde, genera un mensaje destinado al módulo `Actuator` (`Put Message`).

### Actuator → Actuar

El módulo `Actuator` se ejecuta periódicamente cada 1 ms y se encarga de controlar las salidas digitales.

* Recorre los actuadores.
* Comprueba si existe un mensaje (`Any Message?`).
* Si existe, carga el mensaje (`Load Message`).
* Determina si debe modificarse la salida (`Any Change?`).
* Ejecuta la acción correspondiente (`Make Action`).
* Continúa hasta verificar el último actuador (`Last Actuator?`).

La ejecución de los módulos debe ser **no bloqueante**, evitando que alguno de ellos se apropie del uso de la CPU.

---

## 4. Reemplazo de sensores y actuadores

Para realizar el prototipo sin disponer de los sensores y actuadores reales, se reemplazan por entradas y salidas digitales simples.

### Digital Inputs → Sensor

| Sensor real   | Reemplazo              |
| ------------- | ---------------------- |
| `Camera`      | Interruptor DIP Switch |
| `Button`      | Pulsador               |
| `Sensor Coil` | Interruptor DIP Switch |

Los interruptores DIP Switch permiten representar estados digitales estables, mientras que el pulsador permite representar la acción momentánea de solicitar un ticket.

### Digital Outputs → Actuator

| Actuador real | Reemplazo |
| ------------- | --------- |
| `Display`     | LED       |
| `Printer`     | LED       |
| `Barrier`     | LED       |
| `Server`      | LED       |

Los LEDs permiten representar visualmente la activación de cada una de las salidas digitales durante las pruebas del prototipo.
