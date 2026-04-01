# Game Mechanics

## Physics Constants

- All physics values live in `CONFIG` and `game-config.json`. Reference `CONFIG.gravity`, `CONFIG.jumpVelocity`, etc. — never hardcode numbers in logic functions.
- Gravity is expressed in px/s² (800). Jump velocity in px/s (-300). Pipe/wall speed in px/s (120). These are delta-time-based, not per-frame.
- When tuning, adjust one constant at a time. Gravity and jump velocity are tightly coupled — changing one usually requires tweaking the other to keep the feel right.
- Gap size (140px) and wall spacing (350px) control difficulty. Smaller gap = harder. Closer spacing = less reaction time. Keep both in CONFIG for easy balancing.
- Define a `CONFIG.kiro.hitboxInset` (1-2px) to shrink the collision box slightly. This makes near-misses feel fair rather than frustrating.

## Movement Algorithms

- Kiro's vertical movement uses simple Euler integration:
  ```
  velocity += gravity * dt
  y += velocity * dt
  ```
- On jump, set velocity directly: `kiro.velocity = CONFIG.jumpVelocity`. Don't add to existing velocity — this gives consistent jump height regardless of current fall speed.
- Clamp Kiro's position after physics update: `kiro.y = Math.max(0, kiro.y)`. If clamped, also zero the velocity to prevent accumulation above the ceiling.
- Pipes move horizontally: `pipe.x -= CONFIG.wall.speed * dt`. No vertical movement.
- Clouds move at individual speeds derived from their opacity: `cloud.x -= cloud.speed * dt`. When a cloud's right edge passes the left side of the canvas, reposition it to the right edge with a new random Y.
- Ground scrolling is cosmetic only — use a repeating offset: `groundOffset = (groundOffset + CONFIG.wall.speed * dt) % tileWidth`.

## Collision Detection Patterns

- All collision uses AABB (axis-aligned bounding box). No pixel-perfect checks.
- For each pipe pair, derive two rectangles from the pipe state:
  ```
  topPipe:    { x: pipe.x, y: 0, w: pipe.width, h: pipe.gapY - pipe.gapSize / 2 }
  bottomPipe: { x: pipe.x, y: pipe.gapY + pipe.gapSize / 2, w: pipe.width, h: canvasHeight - groundHeight - (pipe.gapY + pipe.gapSize / 2) }
  ```
- Apply hitbox inset to Kiro before testing:
  ```
  kiroBox = { x: kiro.x + inset, y: kiro.y + inset, w: kiro.width - 2*inset, h: kiro.height - 2*inset }
  ```
- Ground collision: `kiro.y + kiro.height >= CONFIG.canvas.height - CONFIG.ground.height`.
- Early exit: skip pipes where `pipe.x + pipe.width < kiro.x` (already behind) or `pipe.x > kiro.x + kiro.width + CONFIG.wall.speed` (too far ahead to matter this frame).
- Pipe pass detection: a pipe is "passed" when `kiro.x > pipe.x + pipe.width` and `pipe.passed === false`. Mark it and increment score.
- Keep `checkCollision` pure — return a boolean. The caller decides what to do (trigger game over, play sound, save score).

## Ghosty Movement Physics

- Ghosty (the ghost sprite from `assets/ghosty.png`) has a fixed horizontal position (`CONFIG.kiro.x`). Only vertical movement is player-controlled.
- Apply a subtle rotation based on velocity to give Ghosty visual tilt: nose up when rising (negative velocity), nose down when falling (positive velocity). Cap rotation to ~±30 degrees.
- Use `ctx.save()`, `ctx.translate()`, `ctx.rotate()`, `ctx.drawImage()`, `ctx.restore()` for the tilt effect. Only apply this transform for the Ghosty sprite, not other entities.
- On the start screen, apply a gentle idle bob using a sine wave: `kiro.y = baseY + Math.sin(time * 2) * 5`. This gives Ghosty life before the game starts.
- When game over triggers, let Ghosty continue falling with gravity until hitting the ground (don't freeze in place). This feels more natural.

## Wall Generation Algorithms

- Walls (pipe pairs) spawn at `x = CONFIG.canvas.width` (right edge) and scroll left at `CONFIG.wall.speed * dt`.
- Track spawn timing with accumulated distance or elapsed time rather than frame count — this keeps spacing consistent across frame rates. Spawn a new wall when `distanceSinceLastSpawn >= CONFIG.wall.spacing`.
- Randomize gap center Y within a safe range:
  ```
  minGapY = CONFIG.wall.gapSize / 2 + margin
  maxGapY = CONFIG.canvas.height - CONFIG.ground.height - CONFIG.wall.gapSize / 2 - margin
  gapY = minGapY + Math.random() * (maxGapY - minGapY)
  ```
  Use a margin (e.g., 40px) to prevent gaps from hugging the ceiling or ground.
- Optionally constrain consecutive gap positions so they don't differ by more than a max delta (e.g., 150px). This prevents impossible sequences where the player can't physically reach the next gap in time.
- Each wall gets a `passed: false` flag for scoring. The pipe manager sets it to `true` once Ghosty clears it.
- Remove walls from the array when `pipe.x + pipe.width < 0`. Do this after the update pass, not during iteration.

## Scoring System Patterns

- Score increments by 1 each time Ghosty's left edge passes a wall's right edge and `pipe.passed === false`.
- Check pass condition in the update phase, not the render phase. Mark the pipe as passed immediately to prevent double-counting.
- Format the score display as `"Score: {score} | High: {highScore}"`. Build this string only when the score changes, not every frame — cache it in `gameState.scoreText`.
- High score updates in real time: `gameState.highScore = Math.max(gameState.score, gameState.highScore)`. Run this check each time score increments.
- Persist high score to `localStorage` under key `flappyKiroHigh` on game over only — not on every score change. Wrap in try-catch for private browsing compatibility.
- On load, read high score with `parseInt(localStorage.getItem('flappyKiroHigh')) || 0`. The `|| 0` handles both `null` and `NaN`.
- Consider a brief visual flash or scale-up on the score text when it increments — subtle feedback that the player scored without being distracting.

## Precise Collision Boundaries

- Define pipe cap dimensions explicitly in CONFIG: `CONFIG.pipe.capWidth` (pipe width + 6px overhang per side) and `CONFIG.pipe.capHeight` (e.g., 24px). Test collision against both the pipe body and the cap as separate rects.
- For the top pipe cap, the rect sits at the bottom of the top pipe. For the bottom pipe cap, it sits at the top of the bottom pipe. These are the edges closest to the gap.
- When Ghosty's hitbox inset is applied, use consistent rounding — always `Math.floor` for position and `Math.ceil` for dimensions to avoid sub-pixel gaps in collision.
- Test collision in world-space coordinates, never screen-space. Since there's no camera offset in this game, they're the same — but keep the habit for correctness.
- For ground collision, treat it as a horizontal line at `CONFIG.canvas.height - CONFIG.ground.height`. Ghosty collides when `kiro.y + kiro.height - inset >= groundY`.

## Momentum Calculations

- Ghosty has no horizontal momentum — only vertical. Velocity is a single scalar, not a vector.
- Terminal velocity: cap downward velocity to prevent Ghosty from falling unreasonably fast after long drops. Add `CONFIG.kiro.maxFallSpeed` (e.g., 600 px/s). Clamp after gravity: `kiro.velocity = Math.min(kiro.velocity, CONFIG.kiro.maxFallSpeed)`.
- Jump always overrides current velocity (set, don't add). This means momentum doesn't carry — a jump from terminal fall speed feels the same as a jump from hover. This is intentional for consistent feel.
- No horizontal momentum on pipes or clouds — they move at fixed speeds. Momentum is purely a Ghosty concern.

## Smooth Animation Interpolation

- Use linear interpolation (lerp) for all timed visual transitions: `lerp(a, b, t) = a + (b - a) * t` where `t` is clamped to [0, 1].
- For score pop, Ghosty tilt, and overlay fades, compute `t = Math.min(1, elapsed / duration)`. Use ease-out for snappier feel: `t = 1 - Math.pow(1 - t, 2)`.
- Ghosty's rotation angle should interpolate smoothly toward the target angle (derived from velocity) rather than snapping. Use `angle = lerp(currentAngle, targetAngle, 0.15)` per frame for a damped follow.
- For the game over overlay fade, interpolate `globalAlpha` from 0 to 0.5 over 300ms using ease-out.
- Cloud positions don't need interpolation — they move at constant speed. But if frame drops cause visible jumps, consider storing previous position and lerping between `prevX` and `nextX` based on frame progress.
- Keep a shared `lerp` utility function at the top of the script. It's used across multiple systems and should be defined once.

## Difficulty Progression

- Increase pipe speed gradually as the score climbs. Use a formula like: `currentSpeed = CONFIG.wall.speed + score * CONFIG.difficulty.speedIncrement`. Cap at `CONFIG.difficulty.maxSpeed`.
- Add difficulty config values:
  ```
  CONFIG.difficulty = {
    speedIncrement: 2,    // px/s added per point scored
    maxSpeed: 200,        // px/s ceiling
    minGap: 100,          // smallest gap size allowed
    gapShrinkRate: 1,     // px reduction per point scored
    gapShrinkStart: 10    // score threshold before gap starts shrinking
  }
  ```
- Gap shrinking: after `gapShrinkStart` points, reduce gap size by `gapShrinkRate` per additional point. Clamp to `minGap`: `gap = Math.max(CONFIG.difficulty.minGap, CONFIG.wall.gapSize - Math.max(0, score - gapShrinkStart) * gapShrinkRate)`.
- Wall spacing stays constant — don't reduce it. Faster speed already means less reaction time between pipes. Shrinking spacing on top of that gets unfair fast.
- Apply the current speed to both pipe movement and ground scroll so they stay in sync. Clouds keep their own independent parallax speeds.
- On game reset, difficulty resets to base values. Don't carry over difficulty from the previous run.
- Keep progression subtle — the player should feel the game getting harder without it being obvious. Test by playing to score 30+ and checking if it still feels fair.
