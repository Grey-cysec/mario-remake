# Homing Bow Bros

A Mario-style platformer that lives in a **single Python file** — with a self-aiming bow, procedurally generated levels, GPU post-processing shaders, and a built-in developer panel. No image assets: every sprite, tile and scenery layer is drawn in code.

## Features

- **Procedural infinite levels** — ground segments, pits, brick platforms, spikes, coins, checkpoints, and a boss guarding the flag
- **Homing bow** — arrows steer themselves toward the nearest enemy in range; triple-shot power-up spreads three at once
- **Three enemy types** — patrolling walkers, flapping bats, and a fireball-throwing boss with a health bar
- **Feather-light movement feel** — coyote time, jump buffering, variable jump height, double jump, squash-and-stretch
- **GPU post-processing** (OpenGL 3.3 via moderngl) — real bloom, colour grading, vignette, subtle film grain
- **Hand-built animated art** — archer run/idle/jump frames, enemy animations, 4-layer parallax scenery, drifting pollen
- **Borderless fullscreen** with aspect-correct scaling (falls back to a plain renderer if OpenGL is unavailable)
- **F1 developer panel** — FPS graph, god mode, live physics sliders, live shader sliders, level controls

## Run it

```bash
pip install pygame moderngl
python3 "mario remake"
```

## Controls

| Key | Action |
| --- | --- |
| Arrows / A D | Move |
| Space / Up / W | Jump — hold for height, tap again mid-air to double jump |
| F | Fire the bow |
| Enter | Start / restart |
| F1 | Developer panel |
| Esc | Quit |

## Tech notes

- One file, ~1,200 lines, standard library + pygame + moderngl only
- Everything renders to an internal 800×450 canvas; the shader stage scales it to your display
- Requires OpenGL 3.3+ for shaders — detected at startup, with an automatic plain-renderer fallback
- Tested with Python 3.9, pygame 2.6, moderngl 5.13 on macOS (Apple silicon)
