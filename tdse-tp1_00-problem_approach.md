## FIUBA - Electrónica - Taller de Sistemas Embebidos
## Trabajo Práctico N°: 1 - Diagramas de Estado - Modelado
### Archivo: tdse-tp1_00-problem_approach.md

La solución de COMA Electronics descrita en la presentación comprende la arquitectura completa de un sistema inteligente de gestión de estacionamientos y el detalle de funcionamiento de la terminal de entrada para la emisión de tickets

1. Estructura del Sistema (Intelligent Parking Management System)
   
La arquitectura general de la solución integra los siguientes componentes centrales:
Servidor Central (Parking System Server): Centraliza el control y almacenamiento de los datos del estacionamiento
Terminales de Entrada y Salida (Entry / Exit Machines): Estaciones de control de acceso para los vehículos
Estación de Pago (Toll Computer / Automatic Pay Station): Equipos de cobro centralizado manuales o automáticos

2. Flujo Operativo del Sistema (Automated Parking System)
   
El ciclo de trabajo del estacionamiento automático consta de tres etapas:

  Ingreso: El vehículo se aproxima a la terminal de entrada. El usuario presiona el botón de ticket (Ticket Button) y la máquina emite un ticket o tarjeta codificado con número de serie, fecha y hora. Simultáneamente, se envía la señal de apertura a la barrera de acceso (Barrier gate). El sistema registra el paso mediante sensores de bobina (sensor coils) y cámara de entrada

  Pago: El cliente estaciona su vehículo. Antes de regresar al auto, acude a la caja central o a la estación automática de cobro (P20 Automatic Pay Station) para abonar la tarifa. Una vez efectuado el pago, el ticket queda validado con un tiempo de gracia preestablecido para salir.
  
  Egreso: Al llegar a la terminal de salida, el escáner (Ticket scanner) lee y valida el ticket abonado, emitiendo la señal de apertura para la barrera de salida.
  
3. Dispensador de Tickets de Entrada (Parking Ticket Dispenser Machine - Entry)
   
Componentes e Interfaz Física

La máquina de entrada de COMA está equipada con:

  Pantalla LCD de 7" y panel indicadores por voz (Voice Prompt).
  Botón de solicitud de ticket (Ticket Button) y ranura de emisión (Ticket Slot).
  Lector de tarjetas (Card Reader).
  Botón de ayuda (Help Button) e intercomunicador opcional de video/voz (Intercom).
  Visualizador LED de plazas vacantes (ej. "Vacant: 128").
  Cámara motorizada con luz automática y barrera de alta velocidad activada por sensor o radar

Modelo de Implementación Embebida (Proyecto TA134)

Para el desarrollo del prototipo (MVP) en el Taller de Sistemas Embebidos, esta máquina de entrada se desglose en una arquitectura modular basada en el patrón Escrutar → Procesar → Actuar

  Escrutar (Scrutinize / Sensores): Captura de entradas digitales como la cámara, el pulsador de ticket y los sensores de detección de presencia de vehículo (Sensor Coil)

  Procesar (Process / Sistema): Lógica interna encargada de gestionar los estados y reglas del negocio
Actuar (Act / Actuadores): Gestión de salidas digitales como la pantalla de visualización (Display), la impresora (Printer), el control de la barrera (Barrier) y la comunicación con el servidor (Server).

  Sincronización: Los módulos intercambian información mediante mensajes dentro de un esquema de ejecución cíclica no bloqueante (cada 1 ms) para garantizar un comportamiento comunitario del microcontrolador
