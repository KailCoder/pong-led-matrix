Game data map and update logic (conceptual)

Coordinate representation:
- x: 3 bits (0..7) — column index
- y: 3 bits (0..7) — row index
- dx: 1 bit (0 = left, 1 = right)
- dy: 1 bit (0 = up, 1 = down)

Paletas:
- Left paddle stored as 3-bit position (top row index for paddle) placed at column 0
- Right paddle similar at column 7

Render mapping (for each scan row R and active column C):
- Column shift register stores byte where bit[y] = 1 if LED at (C,y) on.
- To draw the ball: set bit at column x, row y = 1; clear previous position.

Game tick update (on slow clock):
1. next_x = x + (dx ? +1 : -1)
2. next_y = y + (dy ? +1 : -1)
3. If next_y outside [0..7]: invert dy (dy = !dy), adjust next_y inside bounds.
4. If next_x == 0 (left edge) and left paddle covers next_y -> invert dx (bounce), else if next_x == 0 and no paddle -> right player scores; reset ball to center.
5. If next_x == 7 (right edge) and right paddle covers next_y -> invert dx, else left player scores; reset ball.
6. Update x,y <= next_x,next_y. Update column registers accordingly: clear old bit, set new bit in column x.

Paddle control:
- Each paddle has a 3-bit 'top' index p_top in [0..5] (paddle height = 3).
- Buttons increment/decrement p_top with saturation (no wrap) on game tick (debounced).

Timing:
- fast_scan_clock: drives row scan (aim: total frame ≥ 200 Hz; e.g., row clock 2 kHz -> 8 rows => 250 Hz frame).
- game_tick_clock: ~8–15 Hz (slower) for visible movement and human reaction.

Simplifications to implement fast:
- Paddle size = 2 or 3 (3 looks better); use 2 bits if want less FFs but 3 is recommended.
- Score: single LED per player that toggles when score increases (simple visual), or use 7‑segment if time permits.