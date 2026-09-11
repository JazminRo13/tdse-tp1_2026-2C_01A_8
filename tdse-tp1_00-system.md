## FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N° 1 - Diagramas de Estado - Modelado
### Archivo: `tdse-tp1_00-system.md`

---

## Modelo System

El modelo `System` tiene como objetivo **procesar** los eventos recibidos desde el módulo `Sensor` y determinar las acciones que deben realizarse sobre los actuadores de la máquina de entrada.

El módulo se implementa mediante un modelo temporizado:

**Update by Time Code, period = 1 ms**

Por lo tanto, el sistema verifica periódicamente los eventos recibidos y ejecuta la lógica correspondiente sin utilizar código bloqueante.

---

## Eventos del modelo System

Los eventos del modelo `System` provienen principalmente de los mensajes generados por el módulo `Sensor`.

Entre los eventos que pueden intervenir se encuentran:

- `EV_SYS_CAR_DETECTED`: indica que se detectó la presencia de un vehículo.
- `EV_SYS_BTN_PRESSED`: indica que se confirmó la pulsación del botón de solicitud de ticket.
- `EV_SYS_BTN_NOT_PRESSED`: indica que se confirmó la liberación del botón.
- `EV_SYS_CAR_INSIDE`: indica que el vehículo atravesó la zona de ingreso.
- `EV_SYS_CAR_LEFT`: indica que el vehículo abandonó la zona controlada por el sensor.

Estos eventos funcionan como **triggers** que pueden provocar transiciones en el modelo `System`.

---

## Acciones del modelo System

Las acciones del modelo `System` se ejecutan como consecuencia de los eventos recibidos y del estado actual del sistema.

Estas acciones pueden:

- Generar señales o eventos destinados al módulo `Actuator`.
- Ejecutar funciones.
- Inicializar o modificar variables de control.
- Inicializar o modificar temporizadores (`timer`).
- Utilizar variables como `guard` para condicionar una transición.

Algunas acciones posibles son:

- Generar `EV_ACT_DISPLAY_WELCOME` para solicitar la visualización del mensaje de bienvenida.
- Generar `EV_ACT_PRINT_TICKET` para solicitar la impresión del ticket.
- Generar `EV_ACT_OPEN_BARRIER` para solicitar la apertura de la barrera.
- Generar `EV_ACT_CLOSE_BARRIER` para solicitar el cierre de la barrera.
- Generar `EV_ACT_SERVER` para indicar una comunicación con el servidor.

De esta manera, el módulo `System` no modifica directamente las salidas físicas, sino que genera los eventos necesarios para que el módulo `Actuator` ejecute las acciones correspondientes.

---

## Secuencia general de funcionamiento

El comportamiento general de la máquina de entrada puede representarse mediante la siguiente secuencia:

**Car arrives → Welcome → Button is pressed → Print ticket → Open barrier → Car inside → Close barrier → Car leaves**

El módulo `System` procesa los eventos asociados a esta secuencia y determina las acciones correspondientes en cada etapa.

---

## Uso de Guards y Timers

Las transiciones del modelo `System` pueden estar condicionadas mediante variables de control o temporizadores.

La forma general de una transición es:

`trigger [guard] / effect`

donde:

- `trigger`: evento que provoca la evaluación de la transición.
- `guard`: condición que debe cumplirse para permitir la transición.
- `effect`: acción ejecutada cuando la transición ocurre.

Por ejemplo:

`EV_SYS_BTN_PRESSED [ticket_available] / EV_ACT_PRINT_TICKET`

En este caso:

- El evento `EV_SYS_BTN_PRESSED` actúa como trigger.
- `ticket_available` actúa como guard.
- `EV_ACT_PRINT_TICKET` es la acción generada hacia el módulo `Actuator`.

El uso de timers también permite controlar acciones que requieren una espera temporal sin utilizar código bloqueante.

---

## Resumen

El módulo `System` se encarga de:

- Recibir eventos provenientes del módulo `Sensor`.
- Procesar dichos eventos de acuerdo con el estado actual del sistema.
- Evaluar condiciones o `guards`.
- Administrar variables de control y temporizadores.
- Generar eventos destinados al módulo `Actuator`.

Por lo tanto, el flujo de información es:

**Sensor → System → Actuator**

donde `System` cumple la función de **procesar** la lógica central del sistema.
