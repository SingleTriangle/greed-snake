# Greed Snake

A retro pixel-art snake game built with HTML5 Canvas and Web Audio API — no external dependencies, fully self-contained in a single HTML file.

[Live Demo](https://github.com/SingleTriangle/greed-snake) · [简体中文](./GAME_GUIDE.md)
<img width="1005" height="1008" alt="image" src="https://github.com/user-attachments/assets/ad966cdd-b5a2-4398-9435-dedefba3640a" />


---

## Features

- **5 progressive levels** with increasing speed and challenge
- **10 obstacles per level** — half static, half slow-moving with boundary bounce
- **Power-up system** — speed boost (gold) and speed slow (blue), 5-second duration
- **Wall wrapping** — snake wraps to the opposite side instead of dying
- **Procedural audio** — all sound effects synthesized via Web Audio API oscillators
- **Pixel-art aesthetic** — neon glow effects, body gradient, pulsing food animation
- **Local high score** — persists via `localStorage`

---

## Quick Start

### Option 1 — Local HTTP Server (recommended)

```bash
# Python 3
python3 -m http.server 8088

# Node.js (if installed)
npx serve .
```

Then open `http://localhost:8088` in your browser.

### Option 2 — Direct file open

Double-click `index.html`. Note: some browsers restrict Web Audio on `file://` — use Option 1 if sounds don't play.

### Option 3 — VS Code Live Server

Right-click `index.html` → **Open with Live Server**.

---

## Controls

| Key | Action |
|-----|--------|
| `↑` / `W` | Move up |
| `↓` / `S` | Move down |
| `←` / `A` | Move left |
| `→` / `D` | Move right |
| `Space` | Start / Restart |
| `P` / `Esc` | Pause / Resume |

---

## Game Rules

- Eat **pink food** to score (+10 per food) and grow (+1 segment)
- Avoid **gray obstacles** — collision shortens the snake by 1 segment; game over at length 0
- Collect **gold (+)** for 1.5× speed or **blue (-)** for 0.75× speed, lasting 5 seconds
- Snake **wraps through walls** — no death from wall collision
- Reaching the score threshold advances to the next level (5 levels total)

### Level Progression

| Level | Score to Pass | Base Speed |
|-------|--------------|------------|
| 1 | 50 | 120ms/move |
| 2 | 120 | 110ms/move |
| 3 | 220 | 100ms/move |
| 4 | 350 | 90ms/move |
| 5 | 500 | 80ms/move |

Each level regenerates 10 obstacles and 3 power-ups randomly.

---

## Technical Details

### Architecture

Single `index.html` (~700 lines), zero external dependencies:

| Section | Lines | Description |
|---------|-------|-------------|
| CONSTANTS | 92–112 | Grid, speed, level thresholds, colors |
| AUDIO | 114–183 | Web Audio API oscillator synthesis |
| STATE | 185–219 | Central `state` object |
| INPUT | 226–281 | Keyboard event handling with direction buffer |
| HELPERS | 283–307 | Utility functions |
| GAME LOGIC | 309–551 | Snake movement, collision, level system |
| RENDERING | 553–668 | Canvas draw calls with glow effects |
| GAME LOOP | 670–688 | `requestAnimationFrame`-based timing |

### Collision Detection

Order of checks in `moveSnake()`:
1. Wrap at boundaries
2. Self-collision → truncate from hit point (not just tail removal)
3. Obstacle collision → shorten 1 segment; game over at length 0
4. Food / power-up pickup

### Audio Synthesis

All sounds use Web Audio API `OscillatorNode` + `GainNode` with no audio files:

| Event | Technique |
|-------|-----------|
| Eat food | Sine wave 400→800 Hz glide |
| Power-up | Dual-tone 660 Hz + 880 Hz |
| Obstacle hit | Square wave 100 Hz burst |
| Game over | Descending arpeggio 400→100 Hz |
| Level up | Ascending arpeggio C5→C6 |
| Win | Full C4→E4→G4→C5→G4→E4→C4 scale |

---

## Project Structure

```
Greed_Snake/
├── index.html              # Complete game (HTML + CSS + JS, no deps)
├── SPEC.md                 # Game design specification
├── GAME_GUIDE.md           # Game guide (中文)
├── RUN_GUIDE.md            # How to run locally (中文)
├── CODE_DOCUMENTATION.md   # Code architecture docs (中文)
├── LICENSE                 # MIT License
├── README.md               # This file
└── CONTRIBUTING.md         # Contribution guidelines
```

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for development workflow, code style, and PR guidelines.

---

## License

MIT © 2026
