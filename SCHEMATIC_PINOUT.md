Schematic pin-to-pin (Proteus 8.17) — conexiones recomendadas

Resumen de la arquitectura:
- Multiplex: seleccionar fila con 3-bit counter + 3→8 decoder (74HC161 + 74HC138).
- Column drivers: 8 columnas manejadas por 2 × 74HC595 (cada 595 controla 8 columnas o filas según montaje).
- Registros de columna: cada 74HC595 guarda el patrón para las 8 filas de la columna activa; usar latch (RCLK) para actualizar sin parpadeo.
- Game logic: registros D para x (columna 0..7), y (row 0..7), dx, dy (dirección), paletas (bits en columna 0 y 7).

Conexiones clave (verificar pinout del modelo en tu Proteus):
- 74HC595 (Ucol0, Ucol1):
  - SER (DS) <- parallel data from game logic via shift-in or preset lines (simpler: connect parallel bus via 74HC175 if not shifting)
  - SRCLK <- scan clock (fast)
  - RCLK  <- latch clock (pulse each scan row or when column data updated)
  - OE    <- GND (enable)
  - Q0..Q7 -> column outputs (through resistors to LED columns or to column drivers)

- 74HC161 (scan counter):
  - CLK <- fast scan clock (ClockGenerator)
  - Q0..Q2 -> inputs to 74HC138 (decoder)
  - LOAD/CLR/EN pins configured so counter runs continuously (or use LOAD to reset)

- 74HC138 (decoder):
  - A,B,C inputs <- Q0..Q2 of counter
  - Y0..Y7 outputs -> drive row enable lines (sink or source depending matrix wiring)

- 74HC74 / 74HC175 (game registers):
  - Use D-FFs for x (3 bits), y (3 bits), dx (1 bit), dy (1 bit), paddles (3 bits per paddle if paleta size=3)
  - Clock these D-FFs with the slow game tick (ClockGenerator second output or divided clock)

- Buttons (player controls):
  - Button -> RC debounce (10k + 100nF) -> 74HC14 Schmitt input -> synchronizer (two D-FFs clocked by slow tick) -> control signals

- Power rails:
  - VCC = +5V, GND = 0V. Tie all VCC/GND pins properly.

Consejo práctico:
- Para evitar tearing, update the 74HC595 column latch only between scan cycles (pulse RCLK when row currently inactive).
- Si no quieres diseñar shift-in logic, preload 74HC595 via parallel load trick (use temporary bus) or use 74HC175 registers as column latches.