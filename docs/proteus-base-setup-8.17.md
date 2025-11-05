# Proyecto base Proteus 8.17 — Matriz 8×8 con 74HC595 + 74HC161 + 74HC138

Este documento guía la creación del diseño en Proteus (ISIS) con los tres CIs ya cableados para multiplexar una matriz 8×8. Incluye nombres de partes como aparecen en el "Pick Devices" de Proteus 8.17 y un netlist CSV para verificar conexiones.

## 1) Crear proyecto
1. File → New Project…
2. Project Type: Schematic only
3. Carpeta sugerida: `proteus/` (p.ej. `proteus/pong-matrix.dsn`)
4. Grid: 100 mil, Snap: 25 mil (View → Grid).

## 2) Dispositivos a colocar (BOM mínimo)
- U1: 74HC595 (SN74HC595N)
- U2: 74HC161 (SN74HC161N)
- U3: 74HC138 (SN74HC138N)
- M1: LED MATRIX 8X8 (si no está, usa 64 LED + resistencias o un "LED Matrix 8x8" del librario "Indicators")
- RCOL0..RCOL7: 8 × Resistor 330 Ω (o un RESPACK-8 a 330R si prefieres)
- C1..C3: 3 × 100 nF (decoupling, capacitor cerámico)
- Terminals: VCC, GND
- JCTRL: Header 1X04 (SER, SRCLK, RCLK, OE) [opcional]
- CLK1: DIGITAL CLOCK (fuente de reloj para el contador) [opcional]
- LA1: LOGIC ANALYZER [opcional, para ver QA..QC/Yx]

Sugerencia de colocación:
- U1 (595) a la izquierda; salidas Q0..Q7 hacia la derecha → RCOL → M1 (columnas).
- U2 (161) arriba-centro; U3 (138) arriba-derecha con Y0..Y7 hacia filas de M1.
- M1 a la derecha; columnas arriba y filas abajo (según símbolo).

## 3) Alimentación y desacoplos
- VCC a +5 V, GND a 0 V.
- Coloca C1..C3 (100 nF) entre VCC y GND pegados a cada CI.

## 4) Etiquetas de red (Net Labels)
Usa Terminals → Net Label para nombrar redes:
- SER, SRCLK, RCLK, (OE opcional)
- CLK_SCAN
- A, B, C (del 161 al 138)
- ROW0..ROW7, COL0..COL7

## 5) Cableado esencial (resumen)
- 74HC595 (U1):
  - VCC=pin16 → VCC; GND=pin8 → GND
  - MR=pin10 → VCC; OE=pin13 → GND (o a net OE si quieres blanking)
  - SH_CP=pin11 → SRCLK; ST_CP=pin12 → RCLK; DS=pin14 → SER
  - Q0..Q7 (pins 15,1,2,3,4,5,6,7) → RCOL0..RCOL7 (330 Ω) → COL0..COL7 → M1
- 74HC161 (U2):
  - VCC=16, GND=8
  - CLR=5, LOAD=6, ENP=7, CEP=9, CET=10 → todos HIGH (VCC)
  - CLK=15 → CLK_SCAN (desde DIGITAL CLOCK, ~2 kHz)
  - QA=14 → A; QB=13 → B; QC=12 → C (QD=11 sin usar)
- 74HC138 (U3):
  - VCC=16, GND=8
  - A=1 ← A (QA); B=2 ← B (QB); C=3 ← C (QC)
  - G1=6 → VCC; G2A=4 → GND; G2B=5 → GND
  - Y0..Y7 (15,14,13,12,11,10,9,7) → ROW0..ROW7 de M1 (activas LOW)

## 6) Prueba rápida (sin MCU)
- Reloj: DIGITAL CLOCK (CLK1) a 2 kHz → U2.CLK (CLK_SCAN).
- Forzar columnas: pon SER=1 (lógica alta), aplica pulsos a SRCLK para cargar 8 unos, y luego un pulso a RCLK para latchear; deberían encenderse columnas mientras U3 conmuta las filas.
- Observa QA..QC y Y0..Y7 en LA1 (Logic Analyzer) para confirmar multiplexado.

## 7) Siguientes pasos recomendados
- Implementa "blanking" con OE: sube OE=HIGH al cambiar de fila y bájalo tras RCLK.
- Si el dispositivo "LED MATRIX 8X8" no coincide, usa 64 LED con orientación acorde (filas cátodo común con salidas LOW del 138).
- Aumenta CLK_SCAN si ves parpadeo; objetivo: ≥ 120 Hz totales (≈15 Hz por fila × 8) — típicamente 1–5 kHz.

## 8) Referencia de conexiones
Consulta `docs/proteus-netlist.csv` para un listado explícito pin→net.
