## Eventos y acciones del modelo sensor.

### 1. Eventos (Triggers de entrada al módulo):
Los eventos son las variaciones del estado físico en el pin de entrada digital del microcontrolador al interactuar con el pulsador.
Un evento es una ocurrencia, cambio de condición o lectura que dispara la evaluación de una transición de estado o la ejecución de una tarea
. Se dividen en:
  - Eventos de Posición Binaria / Entrada Física (Módulo Sensor): Reflejan el valor binario del estado del pin de entrada digital (Digital Input) leídos en cada ciclo de 1 ms:
      - `EV_BTN_UP`: El pulsador o interruptor está físicamente en posición liberada (not pressed / abierto).
      - `EV_BTN_DOWN`: El pulsador o interruptor está físicamente en posición presionada (pressed / cerrado).
  - Evento Temporal del Sistema:
      - `Tick (1 ms)`: Evento periódico generado por el ejecutor cíclico cada 1 ms (each 1mS) que permite decrementar temporizadores y procesar filtros.
  - Eventos Lógicos / Señales Intermodulares (Módulo System): Mensajes o eventos de nivel superior emitidos cuando un módulo confirma un cambio.
      - `EV_SYS_BTN_DOWN` / `EV_SYS_BTN_UP`: Señal emitida hacia el módulo System una vez estabilizado el filtro de antirrebote (debouncing).
      - Eventos del flujo de estacionamiento: Detección de vehículo (Car arrives), pulsación confirmada (Button is pressed), vehículo dentro (Car inside) o egreso del área (Car leaves)

### 2. Acciones (Effects / Salidas del Módulo):
Al ser un módulo de la capa de entradas (Digital Inputs / Scrutinize), las acciones no encienden o apagan periféricos físicos, sino que consisten en la emisión de mensajes intermodulares hacia el módulo de procesamiento (System).
Una acción es la respuesta, trabajo o modificación ejecutada a consecuencia de un evento o cambio de estado. Se clasifican según su proposito:
- Acciones sobre Variables de Control (Temporizadores / tick):
    - Carga/Inicialización: Asignar el tiempo de espera a la variable de control (`tick = DEL_BTN_DEBOUNCE` o `tick = DEL_SYS_TIMEOUT`).
    - Modificación/Decremento: Decrementar la variable en cada tick de 1 ms (`tick--`), la cual actúa como guarda (guard `[tick > 0]` / `[tick == 0]`) para condicionar las transiciones.
- Acciones de Comunicación (Señales / Mensajes intermodulares):
    - Depósito de Mensajes (`Put Message`): Acción del módulo Sensor o System para enviar una señal notificando un cambio a la siguiente capa
 (por ejemplo: `EV_ACT_WELCOME`, `EV_ACT_PRINT_START`, `EV_ACT_BARRIER_UP`, `EV_ACT_BARRIER_DOWN`).
- Acciones Físicas sobre Actuadores (Módulo Actuator):
    - Ejecutar Acción (Make Action): Modificación del estado de los puertos de salida digital (Digital Outputs) para encender o apagar los LEDs que simulan la barrera (Barrier), la pantalla (Display), la impresora (Printer) o el servidor (Server)



