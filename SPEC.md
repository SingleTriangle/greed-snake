# Greed Snake Game Specification

## Canvas & Grid
- Canvas: 640×640 px, centered on page
- Grid: 20×20 cells (32px per cell)
- Background: #0a0a0f, faint grid lines #1a1a2e at 0.3 opacity
- Font: "Press Start 2P" (Google Fonts), fallback Courier New monospace

## Visual Style
Pixel art aesthetic with glowing effects.

| Element       | Color              | Effect                          |
|---------------|--------------------|---------------------------------|
| Snake head    | #39ff14 (neon green) | glow shadowBlur: 20           |
| Snake body    | #00ff88 → #00cc66  | gradient tail, shadowBlur: 10   |
| Food          | #ff0066 (hot pink) | pulse scale 0.9↔1.1, shadowBlur: 15 |
| Obstacle      | #888888            | dark gray, shadowBlur: 5        |
| Speed-up item | #ffcc00 (gold)     | glow shadowBlur: 15             |
| Speed-down    | #3399ff (sky blue) | glow shadowBlur: 15             |

## Game Mechanics

### Levels
5 levels total with score thresholds:

| Level | Score to pass |
|-------|--------------|
| 1     | 50           |
| 2     | 120          |
| 3     | 220          |
| 4     | 350          |
| 5     | 500          |

### Snake
- Initial length: 3 segments
- Initial direction: right
- Initial speed: 120ms per move
- Initial position: center of grid

### Obstacles
- 10 per level, randomly placed
- Static or slow-moving (random per obstacle)
- Moving obstacles: bounce at 200ms per move
- Collision: snake shortens 1 segment; GAME OVER if length = 0

### Food
- 1 food at a time
- Eating: +10 score, +1 length
- Pulses for attention

### Power-ups
- 3 per level
- Speed-up (gold): 1.5× speed for 5s
- Speed-down (blue): 0.75× speed for 5s

### Wall wrapping
- Snake wraps to the opposite side when hitting a wall (no death from wall collision)

## Controls
| Key        | Action              |
|------------|---------------------|
| ↑ / W      | Move up             |
| ↓ / S      | Move down           |
| ← / A      | Move left           |
| → / D      | Move right          |
| Space      | Start / Restart     |
| P / Escape | Pause / Resume       |

## Game States
- START, PLAYING, PAUSED, LEVEL_UP, GAME_OVER, WIN

## Scoring
- Food: +10
- High score in localStorage

## Audio
All synthesized via Web Audio API oscillators.
