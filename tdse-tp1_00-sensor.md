## FIUBA - Electrónica - Taller de Sistemas Embebidos

## Trabajo Práctico N° 1 - Diagramas de Estado - Modelado

### Archivo: `tdse-tp1_00-sensor.md`

---

## Modelo Sensor - Botón

El modelo `Sensor` tiene como objetivo **escrutar** periódicamente el estado de un único botón utilizado como entrada digital del sistema.

El botón es un sensor binario que puede presentar dos posiciones:

* **Not Pressed:** botón no presionado.
* **Pressed:** botón presionado.

Estas posiciones actúan como **triggers (Eventos)** del modelo `Sensor`.

El módulo se implementa mediante un modelo temporizado:

**Update by Time Code, period = 1 ms**

Por lo tanto, el estado de la entrada se comprueba periódicamente cada 1 ms sin utilizar código bloqueante.

---

## Eventos del modelo Sensor

Un botón binario genera dos eventos asociados a sus dos posibles posiciones.

Siguiendo la convención:

`event → EV_BTN_NAME`

se definen los siguientes eventos:

* `EV_BTN_NOT_PRESSED`: indica que el botón se encuentra en la posición **Not Pressed**.
* `EV_BTN_PRESSED`: indica que el botón se encuentra en la posición **Pressed**.

Estos eventos funcionan como **triggers** que pueden producir transiciones dentro del modelo `Sensor`.

---

## Acciones del modelo Sensor

Cuando el modelo determina que ocurrió un cambio válido en la posición del botón, debe generar una acción que informe dicho cambio al modelo `System`.

Siguiendo la convención:

`signal → EV_SYS_NAME`

se consideran las siguientes acciones:

* Generar `EV_SYS_BTN_PRESSED` cuando se confirma el cambio de **Not Pressed → Pressed**.
* Generar `EV_SYS_BTN_NOT_PRESSED` cuando se confirma el cambio de **Pressed → Not Pressed**.

Estas acciones funcionan como **signals (Eventos)** para el modelo `System`.

De esta manera, el módulo `Sensor` se encarga de detectar y validar el cambio físico del botón, mientras que el módulo `System` recibe el evento correspondiente para procesarlo.

---

## Rebote del pulsador

Un pulsador mecánico no produce necesariamente una transición instantánea entre los estados abierto y cerrado.

Al presionarlo o liberarlo, sus contactos pueden generar múltiples cambios rápidos entre los niveles lógicos antes de alcanzar un valor estable. Este fenómeno se conoce como **rebote del pulsador (switch bounce)**.

Por ejemplo, una pulsación podría ser leída inicialmente como:

```text
NOT_PRESSED → PRESSED → NOT_PRESSED → PRESSED → PRESSED
```

Si cada cambio fuese considerado como un evento válido, una única pulsación podría interpretarse erróneamente como varias pulsaciones.

Por este motivo es necesario implementar un mecanismo de **debouncing**.

---

## Temporización para Debouncing

Para eliminar los efectos del rebote se utiliza una variable de temporización:

`tick`

con valores comprendidos entre:

`0 ... DEL_BTN_NAME`

donde `DEL_BTN_NAME` representa el tiempo establecido para considerar estable una nueva posición del botón.

El contador `tick` se actualiza cada **1 ms**, aprovechando que el módulo utiliza:

**Update by Time Code, period = 1 ms**

Cuando se detecta una posible modificación en la posición del botón, se inicia o modifica `tick`. La nueva posición solamente se considera válida cuando permanece estable durante el tiempo definido por `DEL_BTN_NAME`.

De esta manera, `tick` puede utilizarse como **guard** para condicionar una transición.

La estructura general de una transición puede expresarse como:

```text
trigger [guard] / effect
```

Por ejemplo:

```text
EV_BTN_PRESSED [tick >= DEL_BTN_NAME] / EV_SYS_BTN_PRESSED
```

La interpretación es:

* **Trigger:** se detecta `EV_BTN_PRESSED`.
* **Guard:** se comprueba que se haya alcanzado el tiempo de validación `DEL_BTN_NAME`.
* **Effect:** se genera `EV_SYS_BTN_PRESSED` para informar al modelo `System`.

De manera equivalente, al liberar el botón:

```text
EV_BTN_NOT_PRESSED [tick >= DEL_BTN_NAME] / EV_SYS_BTN_NOT_PRESSED
```

---

## Resumen de Eventos y Acciones

| Tipo            | Identificador            | Descripción                                        |
| --------------- | ------------------------ | -------------------------------------------------- |
| Evento Sensor   | `EV_BTN_PRESSED`         | Botón en posición Pressed                          |
| Evento Sensor   | `EV_BTN_NOT_PRESSED`     | Botón en posición Not Pressed                      |
| Acción / Signal | `EV_SYS_BTN_PRESSED`     | Informa al System que se confirmó la pulsación     |
| Acción / Signal | `EV_SYS_BTN_NOT_PRESSED` | Informa al System que se confirmó la liberación    |
| Timer           | `tick`                   | Contador utilizado para el debouncing              |
| Delay           | `DEL_BTN_NAME`           | Tiempo requerido para validar una posición estable |

El modelo `Sensor` permite así escrutar el botón periódicamente, eliminar los cambios producidos por el rebote mecánico y comunicar al modelo `System` únicamente los cambios de posición considerados válidos.
