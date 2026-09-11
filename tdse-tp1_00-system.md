## FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N° 1 - Diagramas de Estado - Modelado
### Archivo: `tdse-tp1_00-system.md`

---

## Modelo System

El modelo `System` tiene como objetivo **procesar** los eventos recibidos desde el módulo `Sensor` y determinar las acciones que deben realizarse sobre el módulo `Actuator`.

El módulo se implementa mediante un modelo de código C temporizado:

**Update by Time Code, period = 1 ms**

Por lo tanto, el modelo `System` se ejecuta periódicamente cada 1 ms, procesa los eventos recibidos y genera las acciones correspondientes sin utilizar código bloqueante.

---

## Eventos del modelo System

Los eventos del modelo `System` son generados principalmente por el módulo `Sensor` como consecuencia de los cambios detectados en las entradas digitales.

Para la implementación se consideran las siguientes entradas:

- `Camera`: representada mediante una llave On/Off.
- `Button`: representado mediante un pulsador.
- `Sensor Coil`: representado mediante una llave On/Off.

Siguiendo la convención de identificadores:

`signal → EV_SYS_NAME`

los eventos recibidos por el modelo `System` representan los cambios detectados y validados por el modelo `Sensor`.

Se consideran los siguientes eventos:

- `EV_SYS_CAMERA_ON`: indica la activación de la entrada correspondiente a `Camera`.
- `EV_SYS_CAMERA_OFF`: indica la desactivación de la entrada correspondiente a `Camera`.
- `EV_SYS_BTN_PRESSED`: indica que se confirmó la pulsación del botón.
- `EV_SYS_BTN_NOT_PRESSED`: indica que se confirmó la liberación del botón.
- `EV_SYS_SENSOR_COIL_ON`: indica que se detectó la activación del `Sensor Coil`.
- `EV_SYS_SENSOR_COIL_OFF`: indica que se detectó la desactivación del `Sensor Coil`.

Estos eventos actúan como **triggers** para producir las transiciones correspondientes dentro del modelo `System`.

---

## Acciones del modelo System

Las acciones del modelo `System` se producen como consecuencia de los eventos recibidos y del estado actual del sistema.

Las acciones pueden:

- Generar **signals (Eventos)** destinados al modelo `Actuator`.
- Ejecutar funciones.
- Inicializar o modificar variables de control.
- Inicializar o modificar variables de temporización (`timer`).

Las variables de control o temporización pueden utilizarse como `guard` para condicionar las transiciones del modelo.

Para la implementación del prototipo se considera como actuador únicamente la barrera:

- `Barrier` → LED.

Por lo tanto, las acciones principales que puede generar el modelo `System` hacia el modelo `Actuator` son:

- `EV_ACT_BARRIER_OPEN`: solicita la apertura de la barrera.
- `EV_ACT_BARRIER_CLOSE`: solicita el cierre de la barrera.

El modelo `System` no modifica directamente la salida digital asociada a la barrera. En su lugar, genera un evento para que el módulo `Actuator` realice la acción correspondiente.

---

## Procesamiento del modelo System

El módulo `System` recibe los eventos generados por el módulo `Sensor` y los procesa de acuerdo con el estado actual del sistema.

Su funcionamiento general consiste en:

1. Verificar si existe un mensaje proveniente del módulo `Sensor`.
2. Cargar el mensaje recibido.
3. Identificar el evento correspondiente.
4. Procesar el evento de acuerdo con el estado actual del sistema.
5. Evaluar las condiciones (`guards`) asociadas a las posibles transiciones.
6. Realizar la transición de estado cuando corresponda.
7. Ejecutar las acciones asociadas a la transición.
8. Generar, cuando sea necesario, un evento destinado al módulo `Actuator`.

Todo este procesamiento se realiza de manera no bloqueante y el módulo devuelve el control de la CPU al finalizar cada actualización.

---

## Triggers, Guards y Effects

Las transiciones del modelo `System` pueden representarse mediante:

`trigger [guard] / effect`

donde:

- **trigger:** evento que provoca la evaluación de una transición.
- **guard:** condición que debe cumplirse para permitir la transición.
- **effect:** acción que se ejecuta como consecuencia de la transición.

Los `triggers` corresponden principalmente a los eventos recibidos desde el módulo `Sensor`.

Los `guards` pueden depender de variables de control o temporización utilizadas por el modelo `System`.

Los `effects` pueden modificar variables internas, ejecutar funciones o generar eventos destinados al módulo `Actuator`.

Por ejemplo, cuando las condiciones del sistema determinan que debe permitirse el ingreso del vehículo, el modelo `System` puede generar:

`EV_ACT_BARRIER_OPEN`

De manera equivalente, cuando se determina que el vehículo ya ingresó y corresponde cerrar la barrera, puede generar:

`EV_ACT_BARRIER_CLOSE`

---

## Modelo de ejecución temporizado

El modelo `System` utiliza:

**Update by Time Code, period = 1 ms**

Esto significa que el módulo es actualizado periódicamente cada 1 ms.

La ejecución debe ser **no bloqueante**, por lo que el módulo realiza el procesamiento correspondiente y devuelve el control de la CPU sin realizar esperas activas.

Esto permite mantener el comportamiento comunitario del sistema, evitando que un único módulo se apropie del uso del microprocesador.

---

## Relación con los módulos Sensor y Actuator

El flujo general de información es:

```text
        SENSOR
          │
          │ Eventos
          ▼
       SYSTEM
          │
          │ Eventos
          ▼
      ACTUATOR
          │
          ▼
       BARRIER
         (LED)
