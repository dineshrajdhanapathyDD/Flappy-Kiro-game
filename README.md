# Flappy Kiro

A retro browser-based endless side-scroller game where you guide Ghosty the ghost through green pipes, red impossible gates, and blast obstacles with a laser. Built as a single HTML file with vanilla JavaScript and the HTML5 Canvas API.

![Flappy Kiro](img/example-ui.png)

## How to Play

1. Open `index.html` in a browser (or serve via `python -m http.server 8080`)
2. Click, tap, or press **Space** to start and jump
3. Press **L** to fire the laser (20s recharge)
4. Navigate through pipe gaps to score points
5. Fly through **red gates** to activate Impossible Mode (reversed gravity for 10s)

## Controls

| Input | Action |
|-------|--------|
| Space / Click / Tap | Jump (or start/restart) |
| L | Fire laser |

## Features

- Gravity-based physics with delta-time integration
- Parallax cloud layers with varying opacity and speed
- Green pipes with two-tone rendering and caps
- Red impossible gates that reverse gravity for 10 seconds
- Laser beam that blasts holes through pipes
- Score and high score persistence via localStorage
- Screen shake on collision
- Velocity-based Ghosty tilt animation
- Idle bob animation on start screen
- Touch support for mobile

## Architecture

### Game State Machine

```mermaid
stateDiagram-v2
    [*] --> Start
    Start --> Playing : Space / Click / Tap
    Playing --> GameOver : Collision
    GameOver --> Playing : Space / Click / Tap (after 300ms)
```

### Game Loop (Playing State)

```mermaid
flowchart LR
    A[Process Input] --> B[Update Physics]
    B --> C[Spawn Pipes/Gates]
    C --> D[Move Entities]
    D --> E[Check Collisions]
    E --> F[Update Score]
    F --> G[Update Timers]
    G --> H[Render Frame]
    H --> A
```

### System Modules

```mermaid
graph TD
    CONFIG[CONFIG Object] --> Physics
    CONFIG --> PipeManager
    CONFIG --> CloudManager
    CONFIG --> CollisionDetector
    CONFIG --> Renderer
    CONFIG --> LaserSystem
    CONFIG --> GateSystem

    subgraph Core Logic
        Physics[Physics Engine<br/>updateKiro, jump]
        PipeManager[Pipe Manager<br/>spawnPipe, updatePipes, checkPipePass]
        CloudManager[Cloud Manager<br/>initClouds, updateClouds]
        CollisionDetector[Collision Detector<br/>checkCollision, isInHole]
        Scoring[Scoring<br/>formatScore, updateHighScore]
        GateSystem[Gate System<br/>spawnGate, updateGates, checkGatePass]
        LaserSystem[Laser System<br/>fireLaser, updateLaser]
    end

    subgraph Presentation
        Renderer[Renderer<br/>render]
        AudioManager[Audio Manager<br/>playSound]
    end

    subgraph Infrastructure
        InputHandler[Input Handler<br/>handleInput]
        GameLoop[Game Loop<br/>gameLoop, idleLoop]
        Persistence[Persistence<br/>loadHighScore, saveHighScore]
    end

    GameLoop --> Physics
    GameLoop --> PipeManager
    GameLoop --> CloudManager
    GameLoop --> CollisionDetector
    GameLoop --> Scoring
    GameLoop --> GateSystem
    GameLoop --> LaserSystem
    GameLoop --> Renderer
    InputHandler --> GameLoop
    InputHandler --> LaserSystem
```

### Render Pipeline

```mermaid
flowchart TD
    A[Clear Canvas] --> B[Screen Shake Offset]
    B --> C[Sky Background]
    C --> D[Clouds - semi-transparent ellipses]
    D --> E[Pipes - green with holes]
    E --> F[Gates - red impossible gates]
    F --> G[Ground]
    G --> H[Ghosty Sprite - with tilt]
    H --> I[Laser Beam Effect]
    I --> J[Score HUD]
    J --> K[Impossible Timer]
    K --> L[Laser Recharge Timer]
    L --> M[Overlay Messages]
```

### Collision Detection Flow

```mermaid
flowchart TD
    A[checkCollision] --> B{Ground?}
    B -->|Yes| C[Return true]
    B -->|No| D{For each pipe}
    D --> E{AABB overlap<br/>with top pipe?}
    E -->|Yes| F{Inside a hole?}
    F -->|Yes| D
    F -->|No| C
    E -->|No| G{AABB overlap<br/>with bottom pipe?}
    G -->|Yes| F
    G -->|No| D
    D -->|Done| H{For each gate}
    H --> I{Same AABB + hole check}
    I -->|Collision| C
    I -->|No collision| H
    H -->|Done| J[Return false]
```

### Impossible Mode Flow

```mermaid
sequenceDiagram
    participant P as Player
    participant G as Game Loop
    participant Gate as Gate System
    participant Phys as Physics

    P->>G: Fly through red gate gap
    G->>Gate: checkGatePass() → true
    Gate->>G: Activate impossible mode
    G->>G: gravityReversed = true
    G->>G: impossibleTimer = 10s
    loop Each Frame
        G->>Phys: updateKiro(kiro, dt, reversed=true)
        Phys->>Phys: Apply negative gravity
        G->>G: impossibleTimer -= dt
    end
    G->>G: Timer expires → gravityReversed = false
```

### Laser System Flow

```mermaid
sequenceDiagram
    participant P as Player
    participant I as Input Handler
    participant L as Laser System
    participant Pipes as Pipes/Gates

    P->>I: Press L key
    I->>L: fireLaser(gameState)
    L->>L: Check laserReady
    alt Ready
        L->>L: laserReady = false
        L->>L: rechargeTimer = 20s
        L->>L: beamTimer = 0.3s
        L->>Pipes: Add holes at beam Y position
        loop Each Frame
            L->>L: rechargeTimer -= dt
        end
        L->>L: Timer expires → laserReady = true
    else Recharging
        L->>L: Ignore
    end
```

## Project Structure

```
├── index.html          # Single-file game (HTML + CSS + JS)
├── game-config.json    # Physics parameters reference
├── assets/
│   ├── ghosty.png      # Ghost character sprite
│   ├── jump.wav        # Jump sound effect
│   └── game_over.wav   # Game over sound effect
├── img/
│   └── example-ui.png  # Screenshot
└── .kiro/
    ├── specs/flappy-kiro/
    │   ├── requirements.md
    │   ├── design.md
    │   └── tasks.md
    └── steering/
        ├── game-coding-standards.md
        ├── game-architecture.md
        ├── game-mechanics.md
        ├── canvas-and-collision.md
        └── visual-design.md
```

## CONFIG Object

All tunable constants live in a single `CONFIG` object exposed on `window.CONFIG` for live console tweaking:

| Path | Default | Description |
|------|---------|-------------|
| `gravity` | 800 | Downward acceleration (px/s²) |
| `jumpVelocity` | -300 | Upward velocity on jump (px/s) |
| `pipe.speed` | 120 | Horizontal pipe speed (px/s) |
| `pipe.gap` | 140 | Vertical gap between pipes (px) |
| `wall.spacing` | 350 | Distance between pipe spawns (px) |
| `impossible.duration` | 10 | Reversed gravity duration (s) |
| `laser.rechargeTime` | 20 | Laser cooldown (s) |
| `laser.holeSize` | 60 | Size of laser holes (px) |

## Credits

Created by DD. Built with [Kiro](https://kiro.dev).

## License

See [LICENCE.md](LICENCE.md).
