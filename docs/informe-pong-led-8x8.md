# Pong en matriz LED 8×8 con lógica discreta (TTL/CMOS)
Autor: Equipo | Repositorio: pong-led-matrix | Herramienta: Proteus 8.17

## 1. Resumen ejecutivo
Este proyecto implementa una versión simplificada del juego Pong sobre una matriz LED 8×8 utilizando únicamente lógica discreta (familia 74HC), sin microcontrolador. Se desarrolló en tres etapas:
- Día 1: Multiplexado y escaneo estable de la matriz.
- Día 2: Paletas con controles de usuario (botones con anti-rebote y sincronización) y renderizado en columnas extremas.
- Día 3: Bola con FSM simple, detección de colisiones con paletas y paredes, y demo final; scoring opcional.

Estado actual: Día 1 completado y validado en Proteus. Listo para iniciar Día 2.

## 2. Objetivos
- Obtener un escaneo estable de una matriz LED 8×8 mediante 74HC161 (contador) + 74HC138 (decodificador de filas) y 74HC595 (columnas).
- Integrar controles de usuario con anti-rebote y sincronización al “game tick”.
- Implementar lógica de paletas de altura fija, movimiento por botones y saturación de límites.
- Desarrollar la lógica de la bola (posición y dirección), colisiones y demo del juego.

## 3. Arquitectura y principio de funcionamiento
- Multiplexado por filas:
  - Un 74HC161 genera un conteo binario de 3 bits (QA, QB, QC). El 74HC138 decodifica a 8 salidas activas LOW (Y0…Y7), activando una fila por vez. Frecuencia de escaneo típica: 2–5 kHz para evitar parpadeo.
- Columnas con 74HC595:
  - Se cargan 8 bits y se latchean a Q0..Q7. Como solo una fila está activa, los bits determinan qué LEDs de esa fila encienden.
- Controles de usuario (Día 2):
  - Botones UP/DOWN por jugador con RC + 74HC14 (Schmitt) para anti-rebote y un sincronizador (2 FF por botón) al “game tick” (8–15 Hz), generando un pulso limpio por tick.
- Paletas:
  - Posición vertical por jugador en un contador up/down (p.ej., 74HC193, usando 3 bits útiles). Altura fija (2–3 LEDs).
  - Señal “row-match” enciende la paleta cuando la fila activa r ∈ [pos, pos+alto−1]. Se gatean las columnas 0 y 7 con esta señal.
- Bola (Día 3):
  - Registros de posición (x, y) y dirección (dx, dy). En cada tick, la posición se actualiza e invierte dirección en colisiones. El LED de la bola resulta del AND de “columna==x” y “fila==y”.

## 4. Flujo de señales clave
- Reloj de escaneo → 74HC161.CLK → QA/QB/QC → 74HC138.A/B/C → Y0..Y7 (filas activas LOW).
- Carga de columnas → 74HC595: SER, SH_CP (shift), ST_CP (latch), OE, MR → Q0..Q7.
- Game tick (8–15 Hz) → sincronizadores de botones → pulsos UP/DOWN limpios.
- Posición paletas (contador) + decodificación de filas (row-match) → gateo de Q0/Q7 del 74HC595.

## 5. Implementación por etapas
- Día 1 (completo):
  - 74HC161 + 74HC138 cableados, validación Y0→Y7 en secuencia.
  - 74HC595 operando, resistencias en columnas, prueba de una columna “one-hot”.
  - Frecuencia de escaneo ajustada para brillo y sin parpadeos visibles.
- Día 2 (en progreso):
  - Botones con RC + 74HC14, sincronizadores a game tick.
  - 74HC193 por paleta (3 bits efectivos), saturación 0..5 (altura 3).
  - Row-match y gateo de columnas 0 y 7, sin reescritura por fila.
- Día 3 (pendiente):
  - FSM de bola, colisiones, demo final, scoring opcional.

## 6. Pruebas y validación
- Prueba de filas (manual y automática): verificar Y0…Y7 en orden con columnas fijas.
- Prueba de columnas: cargar patrón one-hot en 74HC595 y observar línea vertical estable.
- Prueba de paletas: movimiento por tick, sin rebotes visibles y sin salir del rango.
- Ajustes: frecuencia de escaneo (2–5 kHz) y game tick (~10 Hz) para compromiso entre brillo, ghosting y jugabilidad.

## 7. Lista de Materiales (BOM)
Notas:
- Familia recomendada: 74HC a 5 V. Un condensador de 100 nF por cada CI, cercano a VCC/GND.
- Confirmar polaridad de la matriz LED (filas cátodo común recomendado para 138 activo LOW) y ajustar resistencias según brillo deseado.

Electrónica base (Día 1)
- 1× Matriz LED 8×8 (especificar modelo y polaridad)
- 1× 74HC161 (contador 4-bit, usar 3 bits)
- 1× 74HC138 (decodificador 3→8, salidas activas LOW)
- 1× 74HC595 (registro de desplazamiento con latch) — considerar 2× si se requiere más patrones
- 8× Resistencias 330 Ω (limitación en columnas; ajustar a brillo y topología)
- 1× Generador de reloj de escaneo (NE555 o generador de Proteus)
- 1× 74HC14 (opcional para limpiar reloj de escaneo)
- 1× Regulador 5 V (7805 o similar) + 2× 10 µF + 1× 100 nF
- 10–15× 100 nF (desacoplo, 1 por CI)

Entradas y control (Día 2)
- 4× Pulsadores (UP/DOWN por jugador)
- 4× Resistencias 10 kΩ (pull-ups/downs) + 4× Condensadores 100 nF (RC anti-rebote)
- 1× 74HC14 (Schmitt; 6 canales, suficiente para 4 botones y reloj)
- Sincronización (2 FF por botón):
  - 4× 74HC74 (cada uno trae 2 FF) o 2× 74HC175 (4 FF c/u) + 1× 74HC08 + 1× 74HC04
- Game tick (~8–15 Hz):
  - 1× NE555 (o un 74HC161 adicional como divisor)
- Posición de paletas:
  - 2× 74HC193 (contadores up/down; usar 3 bits efectivos)
  - 1× 74HC08 (AND) + 1× 74HC04 (NOT) + 1× 74HC32 (OR) para saturación/gating
- Row-match (encendido en pos..pos+H−1):
  - Opción A: 1× 74HC283 (suma) + 2–3× 74HC85 (comparadores) + 74HC32
  - Opción B: 2× 74HC86 (XOR) + 74HC04 + 74HC08 + 74HC32 (XNOR por bit + AND/OR)
- Gateo de columnas:
  - 1× 74HC08 para combinar Q0/Q7 del 74HC595 con PADDLE_ON (ajustar polaridad si hace falta)

Bola y colisiones (Día 3)
- 2× 74HC193 (x e y)
- 1× 74HC175 o 2× 74HC74 (dx, dy y flags)
- 1× 74HC138 adicional (para decodificar x a one-hot si no se reescribe el 595 por tick)
- 1× 74HC85 (r == y) + 74HC08 (AND para LED de bola)
- 74HC86/74HC08/74HC32/74HC04 adicionales para la FSM y colisiones

Varios
- Cables/jumpers, protoboard o PCB, borneras
- Bulk capacitors (10–47 µF) por bloque de alimentación
- Si se requiere mayor corriente en columnas/filas: arrays ULN2803 o MOSFETs y recalcular resistencias

## 8. Consideraciones de diseño
- Desacoplo: 100 nF por CI y bulk de 10–47 µF por segmento de alimentación.
- Polaridad: con 74HC138 activo LOW en filas, normalmente las columnas deben ir a HIGH para encender.
- Blank por fila (opcional): usar OE del 74HC595 para minimizar ghosting durante el cambio de fila.
- Observabilidad: añadir “probes” en QA/QB/QC, Y0..Y7, SH_CP/ST_CP/SER, OE, game tick, y pulsos UP/DOWN.
- Seguridad: usar 74HC14 para limpiar relojes y bordes de botones.

## 9. Trabajo futuro
- Implementar la FSM completa de la bola y reglas de colisión con paletas/bordes.
- Añadir scoring (LEDs o displays de 7 segmentos).
- Ajustes finos de frecuencias para jugabilidad y brillo.

---

### Anexo: Exportar a PDF
- Opción rápida (VSCode): Instalar extensión “Markdown PDF” y exportar.
- Con Pandoc:
  - Requisitos: pandoc y un motor LaTeX (TeX Live/MiKTeX).
  - Comando:
    ```
    pandoc docs/informe-pong-led-8x8.md -o docs/informe-pong-led-8x8.pdf --pdf-engine=xelatex -V geometry:margin=2.5cm
    ```