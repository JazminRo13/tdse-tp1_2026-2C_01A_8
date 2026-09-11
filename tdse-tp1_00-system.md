## FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N° 1 - Diagramas de Estado - Modelado
### Archivo: `tdse-tp1_00-system.md`

---

# Modelo System

El modelo `System` tiene como función **procesar** los eventos recibidos desde
el módulo `Sensor` y determinar las acciones que deben realizarse de acuerdo
con el estado actual del sistema.

El módulo se implementa como un módulo de código C temporizado:

**Update by Time Code, period = 1 ms**

Esto significa que el módulo `System` se ejecuta periódicamente cada 1 ms,
procesando los eventos recibidos y generando las acciones correspondientes.

La ejecución debe ser no bloqueante, evitando que el módulo se apropie del
uso de la CPU.

---

# Paso 08 - Eventos y Acciones del modelo System

## Eventos del modelo System

Los eventos del modelo `System` son generados a partir de los cambios
detectados por el módulo `Sensor`.

Siguiendo la convención de identificadores:

`signal → EV_SYS_NAME`

se consideran los siguientes eventos:

- `EV_SYS_CAR_ARRIVES`: indica que un vehículo llegó a la zona de entrada.
- `EV_SYS_BTN_PRESSED`: indica que el usuario presionó el botón para solicitar
  el ingreso.
- `EV_SYS_CAR_LEAVES`: indica que el vehículo abandonó la zona de entrada.

Estos eventos funcionan como **triggers** para producir las transiciones
correspondientes dentro del modelo `System`.

La secuencia de eventos considerada es:

`Car arrives → Button is pressed → Car leaves`

---

## Acciones del modelo System

Las acciones del modelo `System` se producen como consecuencia de los eventos
recibidos y del estado actual del sistema.

Las acciones pueden generar **signals (Eventos)** destinados al modelo
`Actuator`, ejecutar funciones o inicializar/modificar variables de control.

Para la implementación del prototipo se considera como salida digital la
barrera (`Barrier`), representada físicamente mediante un LED.

Por lo tanto, las acciones del modelo `System` son:

- `EV_ACT_BARRIER_OPEN`: solicita al modelo `Actuator` la apertura de la
  barrera.
- `EV_ACT_BARRIER_CLOSE`: solicita al modelo `Actuator` el cierre de la
  barrera.

Estas acciones funcionan como **signals (Eventos)** enviados desde el modelo
`System` hacia el modelo `Actuator`.

El modelo `System` no modifica directamente la salida digital asociada a la
barrera. El módulo genera el evento correspondiente y el modelo `Actuator`
es el encargado de actuar sobre la salida física.

---

## Relación entre Eventos y Acciones

El comportamiento considerado para el modelo `System` es:

1. Se detecta la llegada de un vehículo (`Car arrives`).
2. El sistema espera que el usuario presione el botón (`Button is pressed`).
3. Cuando se confirma la pulsación, se solicita la apertura de la barrera
   (`Open barrier`).
4. El sistema espera que el vehículo abandone la zona de entrada (`Car leaves`).
5. Cuando el vehículo abandona la zona, se solicita el cierre de la barrera
   (`Close barrier`).

Por lo tanto:

`Car arrives → Button is pressed → Open barrier → Car leaves → Close barrier`

---

# Paso 09 - Tabla de Estados y Excitaciones del modelo System

Para describir el comportamiento del modelo `System` se definen los estados
necesarios para representar la secuencia de ingreso del vehículo.

Siguiendo la convención:

`state → ST_SYS_NAME`

se definen los siguientes estados:

- `ST_SYS_IDLE`: estado de reposo. El sistema espera la llegada de un vehículo.
- `ST_SYS_WAIT_FOR_BTN`: se detectó la llegada de un vehículo y el sistema
  espera que el usuario presione el botón.
- `ST_SYS_WAIT_FOR_CAR`: se solicitó la apertura de la barrera y el sistema
  espera que el vehículo abandone la zona de entrada.

---

## Tabla de Estados y Excitaciones

| Current State | Event | [Guard] | Next State | Actions |
|---|---|---|---|---|
| `ST_SYS_IDLE` | `EV_SYS_CAR_ARRIVES` | - | `ST_SYS_WAIT_FOR_BTN` | - |
| `ST_SYS_WAIT_FOR_BTN` | `EV_SYS_BTN_PRESSED` | - | `ST_SYS_WAIT_FOR_CAR` | `EV_ACT_BARRIER_OPEN` |
| `ST_SYS_WAIT_FOR_CAR` | `EV_SYS_CAR_LEAVES` | - | `ST_SYS_IDLE` | `EV_ACT_BARRIER_CLOSE` |

---

## Descripción de las transiciones

### 1. Llegada del vehículo

El sistema se encuentra inicialmente en:

`ST_SYS_IDLE`

Cuando recibe el evento:

`EV_SYS_CAR_ARRIVES`

se interpreta que un vehículo llegó a la zona de entrada.

El sistema pasa al estado:

`ST_SYS_WAIT_FOR_BTN`

donde espera que el usuario presione el botón para solicitar el ingreso.

En esta transición no es necesario generar ninguna acción sobre la barrera.

---

### 2. Pulsación del botón

El sistema se encuentra en:

`ST_SYS_WAIT_FOR_BTN`

Cuando recibe el evento:

`EV_SYS_BTN_PRESSED`

se interpreta que el usuario solicitó el ingreso.

Como acción, el modelo `System` genera:

`EV_ACT_BARRIER_OPEN`

Este evento se envía al modelo `Actuator`, que será el encargado de activar
la salida digital correspondiente a la barrera.

Luego, el sistema pasa al estado:

`ST_SYS_WAIT_FOR_CAR`

donde espera que el vehículo abandone la zona de entrada.

---

### 3. Salida del vehículo de la zona de entrada

El sistema se encuentra en:

`ST_SYS_WAIT_FOR_CAR`

Cuando recibe el evento:

`EV_SYS_CAR_LEAVES`

se interpreta que el vehículo abandonó la zona de entrada.

Como acción, el modelo `System` genera:

`EV_ACT_BARRIER_CLOSE`

Este evento se envía al modelo `Actuator`, que será el encargado de
desactivar la salida digital correspondiente a la barrera.

Finalmente, el sistema regresa al estado:

`ST_SYS_IDLE`

quedando preparado para procesar el ingreso de un nuevo vehículo.

---

## Triggers, Guards y Effects

Las transiciones del modelo pueden representarse mediante:

`trigger [guard] / effect`

donde:

- **Trigger:** evento que provoca la evaluación de una transición.
- **Guard:** condición que debe cumplirse para permitir la transición.
- **Effect:** acción que se ejecuta cuando ocurre la transición.

Para el comportamiento definido en este modelo no es necesario utilizar
condiciones adicionales (`guards`), por lo que en la tabla se indica `-`.

Por ejemplo:

`EV_SYS_BTN_PRESSED / EV_ACT_BARRIER_OPEN`

indica que, cuando el sistema se encuentra esperando la pulsación del botón,
el evento `EV_SYS_BTN_PRESSED` provoca una transición y genera como efecto
la solicitud de apertura de la barrera.

---

## Ejecución temporizada

El modelo `System` utiliza:

**Update by Time Code, period = 1 ms**

En cada actualización el módulo:

1. Verifica si existe algún mensaje recibido desde el módulo `Sensor`.
2. Si existe un mensaje, lo carga.
3. Procesa el evento de acuerdo con el estado actual.
4. Evalúa la transición correspondiente.
5. Actualiza el estado del sistema.
6. Genera, cuando corresponda, un mensaje destinado al módulo `Actuator`.
7. Devuelve el control de la CPU.

La ejecución debe ser **no bloqueante**, garantizando un comportamiento
comunitario donde ningún módulo se apropie del uso del microprocesador.

---

## Resumen del modelo System

El comportamiento del modelo puede resumirse de la siguiente manera:

`ST_SYS_IDLE`

↓ `EV_SYS_CAR_ARRIVES`

`ST_SYS_WAIT_FOR_BTN`

↓ `EV_SYS_BTN_PRESSED / EV_ACT_BARRIER_OPEN`

`ST_SYS_WAIT_FOR_CAR`

↓ `EV_SYS_CAR_LEAVES / EV_ACT_BARRIER_CLOSE`

`ST_SYS_IDLE`

El flujo entre los módulos es:

`Sensor → System → Actuator → Barrier (LED)`

donde:

- `Sensor` se encarga de **escrutar** las entradas.
- `System` se encarga de **procesar** los eventos.
- `Actuator` se encarga de **actuar** sobre la salida digital correspondiente
  a la barrera.
