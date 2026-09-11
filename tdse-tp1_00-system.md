## FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N° 1 - Diagramas de Estado - Modelado
### Archivo: `tdse-tp1_00-system.md`

---

# Modelo System

El modelo `System` tiene como función **procesar** los eventos recibidos desde
el módulo `Sensor` y determinar las acciones que deben ejecutarse de acuerdo
con el estado actual del sistema.

El módulo se implementa como un módulo de código C temporizado:

**Update by Time Code, period = 1 ms**

Por lo tanto, el módulo `System` se ejecuta periódicamente cada 1 ms,
procesando los mensajes disponibles y devolviendo luego el control de la CPU.

La implementación debe ser no bloqueante.

---

# Paso 08 - Eventos y Acciones del modelo System

## Eventos del modelo System

Los eventos del modelo `System` son generados a partir de los cambios
detectados por el módulo `Sensor`.

De acuerdo con el comportamiento de la `Parking Ticket Dispenser Machine (Entry)`,
se consideran los siguientes eventos:

- `EV_SYS_CAR_ARRIVES`: indica que un vehículo llegó a la terminal de entrada.
- `EV_SYS_BTN_PRESSED`: indica que el usuario presionó el botón para solicitar el ticket.
- `EV_SYS_CAR_LEAVES`: indica que el vehículo abandonó la zona correspondiente al sensor de entrada.
- `EV_SYS_CAR_INSIDE`: indica que el vehículo completó el ingreso al estacionamiento.

Estos eventos funcionan como **triggers** para las transiciones del modelo
`System`.

La relación con las entradas del sistema es:

- `Camera` → genera el evento asociado a `Car arrives`.
- `Button` → genera el evento asociado a `Button is pressed`.
- `Sensor Coil` → genera los eventos asociados al movimiento del vehículo
  durante el ingreso.

Para el prototipo, las entradas reales pueden ser reemplazadas por:

- `Camera` → llave On/Off.
- `Button` → pulsador.
- `Sensor Coil` → llave On/Off.

---

## Acciones del modelo System

Las acciones del modelo `System` se producen como consecuencia de los eventos
recibidos y del estado actual del sistema.

Las acciones pueden:

- Generar signals o eventos destinados a otros módulos.
- Ejecutar funciones.
- Inicializar o modificar variables de control.
- Inicializar o modificar variables de temporización (`timer`).

Las variables de control pueden utilizarse como `guard` para condicionar una
transición mediante:

`trigger [guard] / effect`

De acuerdo con la secuencia de funcionamiento de la máquina de entrada, se
consideran las siguientes acciones:

- `EV_ACT_WELCOME`: solicitar la visualización del mensaje de bienvenida.
- `EV_ACT_PRINT_TICKET`: solicitar la impresión del ticket.
- `EV_ACT_BARRIER_OPEN`: solicitar la apertura de la barrera.
- `EV_ACT_BARRIER_CLOSE`: solicitar el cierre de la barrera.
- `EV_ACT_CAR_INSIDE`: informar que el vehículo ingresó al estacionamiento.

Estas acciones corresponden a los elementos:

- `Welcome` → `Display`.
- `Print ticket` → `Printer`.
- `Open barrier` → `Barrier`.
- `Close barrier` → `Barrier`.
- `Car inside` → `Server`.

En la implementación física simplificada, el actuador real que se reemplaza
explícitamente por una salida digital es:

- `Barrier` → LED.

El LED permite representar visualmente el estado de la barrera durante las
pruebas del prototipo.

---

## Secuencia de funcionamiento

El comportamiento general del sistema puede representarse mediante:

`Car arrives`

↓

`Welcome`

↓

`Button is pressed`

↓

`Print ticket`

↓

`Open barrier`

↓

`Car leaves`

↓

`Close barrier`

↓

`Car inside`

El módulo `System` recibe los eventos correspondientes y genera las acciones
necesarias para continuar con la secuencia de ingreso.

---

# Paso 09 - Tabla de Estados y Excitaciones del modelo System

Para representar el comportamiento del módulo `System` se propone dividir la
secuencia de ingreso en diferentes estados.

Siguiendo la convención:

`state → ST_SYS_NAME`

se definen los siguientes estados:

- `ST_SYS_IDLE`: estado inicial o de reposo. El sistema espera la llegada de un vehículo.
- `ST_SYS_WAIT_FOR_BTN`: se detectó la llegada de un vehículo y el sistema espera que el usuario presione el botón.
- `ST_SYS_WAIT_FOR_CAR_LEAVES`: se solicitó el ticket y la apertura de la barrera; el sistema espera que el vehículo abandone la zona de entrada.
- `ST_SYS_WAIT_FOR_CAR_INSIDE`: el vehículo abandonó la zona de entrada y el sistema espera confirmar que se encuentra dentro del estacionamiento.

---

## Tabla de Estados y Excitaciones

| Current State | Event | [Guard] | Next State | Actions |
|---|---|---|---|---|
| `ST_SYS_IDLE` | `EV_SYS_CAR_ARRIVES` | - | `ST_SYS_WAIT_FOR_BTN` | `EV_ACT_WELCOME` |
| `ST_SYS_WAIT_FOR_BTN` | `EV_SYS_BTN_PRESSED` | - | `ST_SYS_WAIT_FOR_CAR_LEAVES` | `EV_ACT_PRINT_TICKET`, `EV_ACT_BARRIER_OPEN` |
| `ST_SYS_WAIT_FOR_CAR_LEAVES` | `EV_SYS_CAR_LEAVES` | - | `ST_SYS_WAIT_FOR_CAR_INSIDE` | `EV_ACT_BARRIER_CLOSE` |
| `ST_SYS_WAIT_FOR_CAR_INSIDE` | `EV_SYS_CAR_INSIDE` | - | `ST_SYS_IDLE` | `EV_ACT_CAR_INSIDE` |

---

## Descripción de las transiciones

### 1. Llegada del vehículo

El sistema comienza en:

`ST_SYS_IDLE`

Cuando recibe:

`EV_SYS_CAR_ARRIVES`

se detecta que un vehículo llegó a la terminal de entrada.

Como acción, el sistema genera:

`EV_ACT_WELCOME`

para solicitar la presentación del mensaje de bienvenida.

Luego, el sistema pasa a:

`ST_SYS_WAIT_FOR_BTN`

y queda a la espera de que el usuario presione el botón.

---

### 2. Solicitud del ticket

Cuando el sistema se encuentra en:

`ST_SYS_WAIT_FOR_BTN`

y recibe:

`EV_SYS_BTN_PRESSED`

se interpreta que el usuario solicitó el ticket.

El sistema genera las acciones:

- `EV_ACT_PRINT_TICKET`
- `EV_ACT_BARRIER_OPEN`

La primera solicita la impresión del ticket y la segunda solicita la apertura
de la barrera.

Luego, el sistema pasa al estado:

`ST_SYS_WAIT_FOR_CAR_LEAVES`

donde espera que el vehículo abandone la zona de entrada.

---

### 3. Vehículo abandona la zona de entrada

Cuando el sistema se encuentra en:

`ST_SYS_WAIT_FOR_CAR_LEAVES`

y recibe:

`EV_SYS_CAR_LEAVES`

se interpreta que el vehículo abandonó la zona correspondiente al sensor de
entrada.

Como acción, el sistema genera:

`EV_ACT_BARRIER_CLOSE`

para solicitar el cierre de la barrera.

Luego, el sistema pasa a:

`ST_SYS_WAIT_FOR_CAR_INSIDE`

---

### 4. Vehículo dentro del estacionamiento

Cuando el sistema se encuentra en:

`ST_SYS_WAIT_FOR_CAR_INSIDE`

y recibe:

`EV_SYS_CAR_INSIDE`

se confirma que el vehículo completó el ingreso.

Como acción, el sistema genera:

`EV_ACT_CAR_INSIDE`

para informar esta situación.

Finalmente, el modelo regresa al estado:

`ST_SYS_IDLE`

quedando preparado para procesar el ingreso de un nuevo vehículo.

---

## Triggers, Guards y Effects

Las transiciones pueden representarse mediante:

`trigger [guard] / effect`

donde:

- **Trigger:** evento que provoca la evaluación de una transición.
- **Guard:** condición que debe cumplirse para realizar la transición.
- **Effect:** acción ejecutada como consecuencia de la transición.

En este modelo básico no se requieren condiciones adicionales para las
transiciones, por lo que la columna `[Guard]` se representa mediante `-`.

Por ejemplo:

`EV_SYS_BTN_PRESSED / EV_ACT_PRINT_TICKET, EV_ACT_BARRIER_OPEN`

indica que la pulsación validada del botón produce la transición de estado y,
como consecuencia, se solicita la impresión del ticket y la apertura de la
barrera.

---

## Ejecución temporizada

El modelo `System` se ejecuta mediante:

**Update by Time Code, period = 1 ms**

En cada actualización, el módulo:

1. Verifica si existen mensajes disponibles.
2. Carga el evento recibido.
3. Procesa el evento según el estado actual.
4. Evalúa las condiciones de transición.
5. Actualiza el estado cuando corresponde.
6. Genera las acciones o mensajes necesarios.
7. Devuelve el control de la CPU.

La implementación debe ser no bloqueante, garantizando que ningún módulo se
apropie del uso del microprocesador.

---

## Resumen del modelo

El comportamiento del modelo `System` puede resumirse como:

`ST_SYS_IDLE`

↓ `EV_SYS_CAR_ARRIVES / EV_ACT_WELCOME`

`ST_SYS_WAIT_FOR_BTN`

↓ `EV_SYS_BTN_PRESSED / EV_ACT_PRINT_TICKET + EV_ACT_BARRIER_OPEN`

`ST_SYS_WAIT_FOR_CAR_LEAVES`

↓ `EV_SYS_CAR_LEAVES / EV_ACT_BARRIER_CLOSE`

`ST_SYS_WAIT_FOR_CAR_INSIDE`

↓ `EV_SYS_CAR_INSIDE / EV_ACT_CAR_INSIDE`

`ST_SYS_IDLE`
