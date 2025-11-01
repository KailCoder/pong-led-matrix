Bill of Materials (Proteus 8.17 part names)

- Clock: ClockGenerator (Proteus virtual clock)
- 2 × 74HC595 (8-bit shift register + storage register) -> '74HC595'
- 1 × 74HC161 or 74HC163 (4-bit binary counter) -> '74HC161' / '74HC163' (scan counter)
- 1 × 74HC138 (1-of-8 decoder) -> '74HC138' (or 74HC154)
- 1 × 74HC74 or 74HC175 (D-FFs or 4-bit register) -> '74HC74' or '74HC175' (for ball/paddle registers)
- 1 × 74HC00 / 74HC08 / 74HC32 / 74HC04 (basic gates & inverters) -> use as needed
- LEDs: 8×8 matrix component (or 64 individual 'LED') -> 'LED'
- Resistors 330Ω -> 'RESISTOR' (for LEDs)
- Resistors 10kΩ -> 'RESISTOR' (pull-ups for buttons)
- Capacitors 100nF -> 'CAPACITOR' (debounce)
- Pushbuttons x4 (player1 up/down, player2 up/down) -> 'SWITCH (SPST)'
- Optional: small breadboard virtual connectors, wires, VCC/GND rails

Notas:
- Si prefieres menos chips, reemplaza 74HC74 por 74HC175 (registro 4-bit). 74HC595 facilita multiplexado de columnas (shift + latch).
- Usar ClockGenerator para dos dominios: fast scan clock (kHz) y slow game tick (≈8–15 Hz).