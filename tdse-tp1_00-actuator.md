## FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N° 1 - Diagramas de Estado - Modelado
### Archivo: `tdse-tp1_00-actuator.md`

---

# Modelo Actuator

El modelo `Actuator` tiene como función **actuar** sobre las salidas digitales
de acuerdo con los eventos recibidos desde el modelo `System`.

Para la implementación del prototipo se considera un único actuador:

- `Barrier` → LED.

El LED representa el estado de la barrera:

- LED encendido → barrera abierta.
- LED apagado → barrera cerrada.

El módulo se implementa como un módulo de código C temporizado:

**Update by Time Code, period = 1 ms**

Por lo tanto, el modelo `Actuator` se ejecuta periódicamente cada 1 ms y debe
realizar sus acciones sin utilizar código bloqueante.

---

# Paso 10 - Eventos y Acciones del modelo Actuator

## Eventos del modelo Actuator

Los eventos del modelo `Actuator` son generados por el modelo `System`.

Se consideran los siguientes eventos:

- `EV_ACT_BARRIER_OPEN`: solicita la apertura de la barrera.
- `EV_ACT_BARRIER_CLOSE`: solicita el cierre de la barrera.

Estos eventos funcionan como **triggers** para producir las transiciones del
modelo `Actuator`.

La comunicación entre los modelos se puede representar como:

`System → Actuator → Barrier (LED)`

---

## Acciones del modelo Actuator

Las acciones del modelo `Actuator` modifican la salida digital asociada al LED
que representa la barrera.

Se consideran las siguientes acciones:

- `LED_ON()`: enciende el LED, representando la barrera abierta.
- `LED_OFF()`: apaga el LED, representando la barrera cerrada.
- `tick = 0`: inicializa la variable de temporización.
- `tick++`: incrementa la variable de temporización cada 1 ms.

Las acciones también pueden modificar variables de control que posteriormente
pueden utilizarse como `guard` para condicionar las transiciones.

---

## Temporización del modelo Actuator

Para controlar acciones temporizadas sin utilizar código bloqueante se utiliza
la variable:

`tick`

Debido a que el modelo `Actuator` se actualiza cada 1 ms, el incremento de
`tick` permite medir el tiempo transcurrido.

Se define:

`DEL_ACT_NAME`

como el tiempo necesario para completar la acción correspondiente.

De esta manera, una transición puede expresarse mediante:

`trigger [guard] / effect`

Por ejemplo:

`[tick >= DEL_ACT_NAME] / LED_ON()`

indica que la acción se ejecuta cuando se alcanza el tiempo establecido.

El uso de `tick` evita utilizar retardos bloqueantes y permite que los demás
módulos continúen ejecutándose mientras transcurre el tiempo de actuación.

---

# Paso 11 - Tabla de Estados y Excitaciones del modelo Actuator

Para representar el funcionamiento del LED asociado a la barrera se definen
cuatro estados.

Siguiendo la convención:

`state → ST_ACT_NAME`

se consideran:

- `ST_ACT_LED_OFF`: el LED se encuentra apagado y la barrera se representa como cerrada.
- `ST_ACT_LED_TURNING_ON`: se recibió la orden de apertura y se está temporizando la acción.
- `ST_ACT_LED_ON`: el LED se encuentra encendido y la barrera se representa como abierta.
- `ST_ACT_LED_TURNING_OFF`: se recibió la orden de cierre y se está temporizando la acción.

---

## Tabla de Estados y Excitaciones

| Current State | Event | [Guard] | Next State | Actions |
|---|---|---|---|---|
| `ST_ACT_LED_OFF` | `EV_ACT_BARRIER_OPEN` | - | `ST_ACT_LED_TURNING_ON` | `tick = 0` |
| `ST_ACT_LED_TURNING_ON` | - | `tick < DEL_ACT_NAME` | `ST_ACT_LED_TURNING_ON` | `tick++` |
| `ST_ACT_LED_TURNING_ON` | - | `tick >= DEL_ACT_NAME` | `ST_ACT_LED_ON` | `LED_ON()` |
| `ST_ACT_LED_ON` | `EV_ACT_BARRIER_CLOSE` | - | `ST_ACT_LED_TURNING_OFF` | `tick = 0` |
| `ST_ACT_LED_TURNING_OFF` | - | `tick < DEL_ACT_NAME` | `ST_ACT_LED_TURNING_OFF` | `tick++` |
| `ST_ACT_LED_TURNING_OFF` | - | `tick >= DEL_ACT_NAME` | `ST_ACT_LED_OFF` | `LED_OFF()` |

---

## Descripción de las transiciones

### 1. Barrera cerrada

El modelo comienza en:

`ST_ACT_LED_OFF`

En este estado el LED permanece apagado, representando que la barrera está
cerrada.

Cuando el modelo recibe:

`EV_ACT_BARRIER_OPEN`

se inicializa:

`tick = 0`

y el modelo pasa al estado:

`ST_ACT_LED_TURNING_ON`

---

### 2. Temporización de apertura

Mientras el modelo se encuentra en:

`ST_ACT_LED_TURNING_ON`

la variable `tick` se incrementa cada 1 ms.

Mientras se cumpla:

`tick < DEL_ACT_NAME`

el modelo permanece en el mismo estado.

Cuando se cumple:

`tick >= DEL_ACT_NAME`

se ejecuta:

`LED_ON()`

y el modelo pasa al estado:

`ST_ACT_LED_ON`

El LED encendido representa que la barrera está abierta.

---

### 3. Barrera abierta

Mientras el modelo se encuentra en:

`ST_ACT_LED_ON`

el LED permanece encendido.

Cuando el modelo recibe:

`EV_ACT_BARRIER_CLOSE`

se inicializa nuevamente:

`tick = 0`

y el modelo pasa al estado:

`ST_ACT_LED_TURNING_OFF`

---

### 4. Temporización de cierre

Mientras el modelo se encuentra en:

`ST_ACT_LED_TURNING_OFF`

la variable `tick` se incrementa cada 1 ms.

Mientras:

`tick < DEL_ACT_NAME`

el modelo permanece en el mismo estado.

Cuando se cumple:

`tick >= DEL_ACT_NAME`

se ejecuta:

`LED_OFF()`

y el modelo regresa al estado:

`ST_ACT_LED_OFF`

El LED apagado representa que la barrera se encuentra cerrada.

---

## Triggers, Guards y Effects

Las transiciones del modelo se representan mediante:

`trigger [guard] / effect`

donde:

- **Trigger:** evento recibido desde el modelo `System`.
- **Guard:** condición que debe cumplirse para realizar una transición.
- **Effect:** acción ejecutada como consecuencia de la transición.

Por ejemplo:

`EV_ACT_BARRIER_OPEN / tick = 0`

indica que el evento de apertura inicia la temporización.

Posteriormente:

`[tick >= DEL_ACT_NAME] / LED_ON()`

indica que, una vez transcurrido el tiempo establecido, el LED se enciende.

De manera equivalente:

`EV_ACT_BARRIER_CLOSE / tick = 0`

inicia la temporización correspondiente al cierre y:

`[tick >= DEL_ACT_NAME] / LED_OFF()`

apaga el LED cuando se cumple el tiempo establecido.

---

## Ejecución temporizada

El modelo `Actuator` utiliza:

**Update by Time Code, period = 1 ms**

En cada actualización el módulo:

1. Verifica si recibió algún evento desde el modelo `System`.
2. Procesa el evento de acuerdo con el estado actual.
3. Actualiza la variable `tick` cuando corresponde.
4. Evalúa las condiciones (`guards`).
5. Ejecuta la acción correspondiente.
6. Actualiza el estado del modelo.
7. Devuelve el control de la CPU.

De esta manera, la temporización se realiza sin utilizar retardos bloqueantes.

---

## Resumen del modelo Actuator

El funcionamiento puede resumirse como:

`ST_ACT_LED_OFF`

↓ `EV_ACT_BARRIER_OPEN / tick = 0`

`ST_ACT_LED_TURNING_ON`

↓ `[tick >= DEL_ACT_NAME] / LED_ON()`

`ST_ACT_LED_ON`

↓ `EV_ACT_BARRIER_CLOSE / tick = 0`

`ST_ACT_LED_TURNING_OFF`

↓ `[tick >= DEL_ACT_NAME] / LED_OFF()`

`ST_ACT_LED_OFF`

Por lo tanto, el flujo completo de la implementación queda:

`Sensor → System → Actuator → Barrier (LED)`
