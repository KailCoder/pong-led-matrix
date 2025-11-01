Step-by-step assembly & quick plan (Proteus 8.17) — Día 1→3

Preparación:
- Abre Proteus 8.17, crea nuevo proyecto.
- Busca y coloca los componentes listados en BOM.md.

Día 1 (multiplex y scan) — ≈2 h
1. Coloca ClockGenerator. Configure a fast clock for scan (ej. 2 kHz). Option: also add second ClockGenerator at ~10 Hz for game tick (o use a divider).
2. Coloca 74HC161 (scan counter) y 74HC138 (decoder).
   - Conecta CLK de 74HC161 al fast clock.
   - Q0..Q2 del 161 -> A,B,C del 74HC138.
   - Salidas Y0..Y7 del 74HC138 -> filas (activan una fila a la vez).
3. Coloca 2 × 74HC595 para columnas.
   - Conecta outputs Q0..Q7 a columnas (a través de 330Ω a los anodos/cátodos según el tipo de matriz).
   - SRCLK <- fast clock; RCLK controlled to latch outputs.
4. Wire a simple pattern: tie data inputs so that one column shows a lit LED that moves. Pulse RCLK appropriately to update columns.
5. Simula y ajusta frecuencia hasta que no haya parpadeo.

Día 2 (paletas y controles) — ≈2 h
1. Implementa registros para paletas: usa 74HC74 or 74HC175 to store paddle top index.
2. Add buttons (up/down per player) with RC debounce (10k + 100nF) -> 74HC14 -> synchronizer (two D‑FF clocked by game_tick).
3. Implement logic to move paddle variable p_top (increment/decrement with saturation).
4. Map paddle bits into column 0 and 7 data of the 74HC595 (when latch updated, show paddles).
5. Test manual movement while scan runs.

Día 3 (FSM bola, colisiones y demo) — ≈2–3 h
1. Implement ball registers (x,y,dx,dy) using D-FFs clocked by game_tick.
2. Implement combinational logic for next_x,next_y, collision detection with paddles and walls (per EQUATIONS.md).
3. On bounce invert dx or dy accordingly; on miss increment score LED and reset ball to center.
4. On each state update: update column shift registers data (clear previous ball position bit and set new).
   - To avoid glitches, update 74HC595 latch when row decoder selects a row that is not the one being updated, or simply update during the RCLK period.
5. Test scenarios: auto-play (no input) and manual play (move paddles to bounce ball). Show scoring LED changes.

Consejos de debug:
- Añade probes en: Q bits of ball/paddle registers, RCLK, SRCLK, row select lines, fast and slow clocks.
- Si el patrón parpadea: aumentar fast clock; si se ven glitches al actualizar columnas, asegúrate de pulsar RCLK sólo cuando SR outputs estables.
- Si botones rebotan: revisar RC values y el 74HC14 conexión.

Checklist para la presentación:
- Abrir Proteus con el proyecto.
- Mostrar probe del fast clock y game tick.
- Ejecutar ciclo de scan con bola en movimiento (auto-play).
- Pedir que alguien presione botones de paleta para mostrar interacción.
- Forzar fallo (no mover paleta) para mostrar scoring/reset.