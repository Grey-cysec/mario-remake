# Homing Bow Bros — Game Flowchart

Every step of the game, from launch to game over. Open this file on GitHub to
see it rendered, or open `flowchart.html` in a browser for a full-screen view.

## Legend

| Shape / Color | Meaning |
| --- | --- |
| ⬛ Rounded | Startup / shutdown |
| 🟦 Blue | Input & event handling |
| 🟩 Green | Gameplay / physics |
| 🟨 Yellow | Rendering |
| 🟥 Red | Damage, death & reset |
| 🟪 Purple | Decisions |

---

## 1. Launch

```mermaid
flowchart TD
    A["Run: python3 mario remake"] --> B["pygame.init() + read desktop size"]
    B --> C{"OpenGL 3.3 core available?"}
    C -->|"yes"| D["GL pipeline: frame -> 800x450 canvas -> bright pass -> blur -> bloom + grade + vignette -> fullscreen"]
    C -->|"no"| E["Fallback: plain scaled renderer"]
    D --> F["Build art: 96-pose run cycle, idle/jump/fall, walker, bat, boss,<br/>4 parallax layers, tiles, pollen"]
    E --> F
    F --> G["Level 0 generated: ground, pits, bricks, spikes, coins,<br/>checkpoints, walkers, bats, boss, flag"]
    G --> H["State = TITLE"]
```

## 2. Main loop (runs 60 times per second)

```mermaid
flowchart TD
    A["Frame start: dt = min(1/30, clock.tick(60))"] --> B["Poll events"]
    B --> C{"Esc or Quit?"}
    C -->|"yes"| Z["pygame.quit() -> exit"]
    C -->|"no"| D{"Enter pressed<br/>on title / game-over?"}
    D -->|"yes"| E["start_game(): lives=3, score=0,<br/>regenerate level, state=PLAY"]
    D -->|"no"| F{"F1 pressed?"}
    F -->|"yes"| G["Toggle debug panel"]
    F -->|"no"| H["Track keydown / keyup / focus-loss<br/>in event-based key set"]
    E --> H
    G --> H
    H --> I["update(dt)"]
    I --> J["draw() -> 800x450 canvas"]
    J --> K["present() -> shader composite<br/>-> screen (1 flip per frame)"]
    K --> A
```

## 3. update() — one gameplay tick

```mermaid
flowchart TD
    A["update(dt)"] --> B{"state = PLAY?"}
    B -->|"no"| STOP["return (title / game-over frozen)"]
    B -->|"yes"| C["Tick timers: banner, flash, squash,<br/>shot cooldown, invincibility, triple-shot,<br/>ground-contact probe, run-phase, displacement"]
    C --> D{"Player dead?"}

    D -->|"yes"| E["Death fall + spin for 1.5 s"]
    E --> F{"Lives left?"}
    F -->|"no"| G["state = GAME-OVER"]
    F -->|"yes"| H["Lives -1, respawn at checkpoint<br/>on safe footing, 2 s invincibility,<br/>clear arrows/fireballs, reset coins,<br/>killed mobs STAY dead"]

    D -->|"no"| I["Read keys (focus-safe)"]
    I --> J["Move: accel / friction, max speed"]
    J --> K["Jump: 0.12 s input buffer + 0.1 s coyote,<br/>variable height, double jump, squash-stretch"]
    K --> L["Gravity, cap 900 px/s"]
    L --> M["move_x: swept tile collision"]
    M --> N["move_y: tile collision + 1 px ground probe<br/>(stable ground state)"]
    N --> O{"F held?"}
    O -->|"yes"| P["shoot(): arrow every 0.35 s<br/>=2 dmg to boss"]
    O -->|"no"| Q{"Touching spike<br/>or below screen?"}
    P --> Q
    Q -->|"yes"| R["kill_player(): shake + flash"]
    Q -->|"no"| S["Coins, power-ups (+1 life / triple shot)"]

    S --> T{"Boss alive<br/>at the gate?"}
    T -->|"yes"| U["Clamp player before flag"]
    T -->|"no"| V{"Past the flag?"}
    V -->|"yes"| W["Score +100, level +1,<br/>regenerate level, flash"]
    V -->|"no"| X["Checkpoints: save respawn,<br/>banner + heal burst"]

    H --> EN
    R --> EN
    W --> EN
    U --> EN
    X --> EN

    subgraph EN["Entities"]
        EN1["Walkers: patrol toward player,<br/>freeze legs when standing"]
        EN2["Bats: sine-wave chase within 800 px"]
        EN3["Boss: walk, hop every 3.4 s,<br/>fireball every 1.5-2.8 s"]
        EN4["Arrows: home to nearest enemy,<br/>prev-position hit test, die on wall"]
        EN5["Fireballs: player contact = death,<br/>fiery trail, die on wall"]
        EN6["Particles + drifting pollen"]
    end
    EN --> Y["Camera follows player, shake decay"]
    Y --> STOP
```

## 4. Combat & scoring detail

```mermaid
flowchart TD
    A{"Arrow hits enemy?"} -->|"walker / bat"| B["One hit = kill, +30 score,<br/>burst, body falls then freezes"]
    A -->|"boss"| C["Boss -2 HP, white flash,<br/>+500 on kill"]
    A -->|"wall / 0.7 s old"| D["Arrow despawns"]
    E{"Player touches enemy?"} -->|"stomp: falling on top"| F["Enemy takes 1 dmg (stomp),<br/>player bounces -380"]
    E -->|"side / below"| G["kill_player()"]
    H{"Enemy died?"} --> I["Stays dead for this level —<br/>no respawn on death or reset"]
```

## 5. draw() + present() — every frame

```mermaid
flowchart TD
    A["draw()"] --> B["Sky gradient + sun halo<br/>4 parallax layers: ridge, mountains, clouds, hills"]
    B --> C["Tiles: only visible columns,<br/>grass caps, flowers, waving flag"]
    C --> D["Coins with spin + glint, checkpoints, power-ups,<br/>boss health bar, flag pole"]
    D --> E["Enemies (animated, off-screen culled)"]
    E --> F["Arrows with glow trail"]
    F --> G["Player: idle / run 96-pose / jump / fall,<br/>blink when invincible, squash-stretch, death spin"]
    G --> H["Pollen + HUD: score, lives, level,<br/>power timer, progress bar"]
    H --> I["Overlays: title, level banner,<br/>game-over, screen flash"]
    I --> J{"F1 panel open?"}
    J -->|"yes"| K["Debug panel: FPS graph, god mode,<br/>buttons, physics + shader sliders"]
    J -->|"no"| L["present()"]
    K --> L
    L --> M{"GL available?"}
    M -->|"yes"| N["Upload canvas -> bright pass<br/>-> horizontal + vertical blur<br/>-> final: bloom, saturation, contrast,<br/>vignette, film grain, letterbox"]
    M -->|"no"| O["Scale to window with letterbox"]
    N --> P["display.flip() — exactly once"]
    O --> P
```

## 6. State summary

| State | Entered when | Exits when |
| --- | --- | --- |
| `title` | Launch | Enter pressed |
| `play` | Enter on title / restart | Death with 0 lives |
| `over` | Lives run out | Enter → new game |
| *(quit)* | Esc / window close | Process ends |
