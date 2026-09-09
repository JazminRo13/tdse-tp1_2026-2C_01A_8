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

## 2. Procesar (Process / Sistema - System):
   Gestiona la lógica de control y la secuencia de estados del prototipo: llegada del vehículo (Car arrives) $\rightarrow$ mensaje de bienvenida (Welcome) $\rightarrow$ presión del botón (Button is pressed) $\rightarrow$ impresión del ticket (Print ticket) $\rightarrow$ apertura de barrera (Open barrier) $\rightarrow$ vehículo ingresado (Car inside) $\rightarrow$ cierre de barrera (Close barrier) $\rightarrow$ egreso del área de entrada (Car leaves)


