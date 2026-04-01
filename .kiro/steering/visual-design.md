# Visual Design

## Sprite Rendering Patterns

- Load all sprites at init using `new Image()` with `onload` callbacks. Only start the game loop once all assets report ready.
- Provide a fallback for failed sprite loads: draw a colored rectangle matching the entity's bounding box. Check `image.complete && image.naturalWidth > 0` before calling `drawImage`.
- Always specify destination width and height in `ctx.drawImage(img, x, y, w, h)` to avoid implicit scaling from the source image dimensions.
- For Ghosty, apply velocity-based rotation via `ctx.save()` → `ctx.translate(centerX, centerY)` → `ctx.rotate(angle)` → `ctx.drawImage(img, -w/2, -h/2, w, h)` → `ctx.restore()`.
- Render pipes as filled rectangles with a darker cap rectangle at the opening end. Use `ctx.fillRect` for the body and a slightly wider, shorter rect for the cap — no sprite needed.
- Draw clouds as semi-transparent filled ellipses using `ctx.ellipse()`. Set `ctx.globalAlpha = cloud.opacity` before drawing, restore to 1.0 after.
- Render the ground as a dark filled rectangle spanning the full canvas width at the bottom. Optionally add a subtle texture line at the top edge.

## Animation Systems

- Use delta-time-driven animation, not frame counting. Track elapsed time for oscillations and transitions.
- Ghosty idle bob on the start screen: `y = baseY + Math.sin(elapsed * 2) * 5`. Use a module-scoped `elapsed` counter incremented by `dt` each frame.
- Score pop effect: when score increments, briefly scale the score text up (e.g., 1.3x) and lerp back to 1.0 over 200ms. Track `scoreAnimTimer` on game state.
- Game over transition: fade in a semi-transparent dark overlay using `ctx.globalAlpha` that increases from 0 to 0.5 over 300ms. Display "Game Over" text and restart prompt on top.
- Cloud drift runs in all game states (start, playing, gameOver) to keep the scene alive. Never pause cloud animation.
- For any timed animation, use a simple lerp pattern: `value = start + (end - start) * Math.min(1, elapsed / duration)`. Reset `elapsed` to 0 when the animation triggers.

## Particle Effect Guidelines

- Keep particles lightweight — plain objects with `{ x, y, vx, vy, life, maxLife, size, opacity }`. No classes.
- Store particles in a flat array on game state: `gameState.particles`. Cap the array at a reasonable max (e.g., 30) to avoid GC pressure.
- On game over collision, emit a small burst of white particles from Ghosty's position. Give each particle a random outward velocity and a short lifespan (300-500ms).
- Update particles each frame: apply velocity, decrement life by `dt`, fade opacity as `particle.life / particle.maxLife`. Remove dead particles after the update pass.
- Render particles as small filled circles with `ctx.arc()`. Set `ctx.globalAlpha` to the particle's current opacity.
- Particles are purely cosmetic — they don't affect collision, scoring, or game state. Keep them in the render layer only.
- Optional: add subtle trail particles behind Ghosty during gameplay — spawn one every few frames at Ghosty's position with low opacity and short life. Skip this if performance is a concern.

## Ghosty Character Animations

- Ghosty has three visual states: idle (start screen), flying (playing), and falling (game over).
- Idle: gentle sine-wave bob at `baseY + Math.sin(elapsed * 2) * 5`. No rotation. Eyes centered.
- Flying: velocity-based tilt. Map velocity to rotation angle: `targetAngle = clamp(velocity * 0.002, -0.5, 0.8)` radians (~-30° to ~45°). Smooth toward target with damped lerp each frame.
- Falling (game over): let gravity continue pulling Ghosty down. Increase rotation toward nose-down (~90°) as Ghosty falls. Stop updating once Ghosty hits the ground.
- Optional squash-and-stretch on jump: briefly compress Ghosty vertically (0.85x height, 1.15x width) on jump frame, then lerp back to 1.0 over 100ms. Subtle but adds juice.
- Draw order for Ghosty: translate to center → rotate → draw image offset by half-width/height. Always use `ctx.save()`/`ctx.restore()` to isolate the transform.

## Wall Textures

- Pipe body: use a two-tone green fill. Main body is `#4CAF50`, with a 3px highlight strip on the left edge at `#66BB6A` to simulate a light source from the left.
- Pipe cap: slightly wider than the body (3-4px overhang per side). Use a darker green `#388E3C` for the cap fill, with a 1px top border at `#2E7D32`.
- Draw pipe caps as separate `fillRect` calls on top of the body. Cap height ~24px.
- For the retro sketchy feel, optionally add a 1px dark green (`#2E7D32`) outline around both body and cap using `ctx.strokeRect`. Set `ctx.lineWidth = 1`.
- Pipe rendering order: draw body first, then cap on top. Draw bottom pipe body upward from the bottom, top pipe body downward from the top.
- Keep all pipe colors in CONFIG for easy theming: `CONFIG.pipe.bodyColor`, `CONFIG.pipe.highlightColor`, `CONFIG.pipe.capColor`, `CONFIG.pipe.outlineColor`.

## Background Parallax Effects

- Use three visual depth layers, each scrolling at a different speed:
  1. Far clouds: lowest opacity (0.2-0.35), slowest speed (0.3-0.5 px/s). Small size.
  2. Mid clouds: medium opacity (0.35-0.55), medium speed (0.5-1.0 px/s). Medium size.
  3. Near clouds: highest opacity (0.55-0.7), fastest speed (1.0-1.5 px/s). Largest size.
- Assign each cloud a layer at creation time based on its randomized opacity. Speed is derived from opacity — no separate layer tracking needed.
- The sky background itself is static (solid light blue `#87CEEB` fill). It doesn't scroll.
- Ground scrolls at the same speed as pipes (`CONFIG.wall.speed`) to feel anchored to the gameplay. Use a repeating offset pattern for seamless tiling.
- Clouds wrap individually: when a cloud's right edge passes `x = 0`, reposition it to `x = canvasWidth + random offset` with a new random Y. This prevents all clouds from bunching up.
- Draw clouds behind pipes but in front of the sky. Render order: sky → far clouds → mid clouds → near clouds → pipes → ground → Ghosty → HUD.

## Sound Effect Integration

- Preload both audio files (`jump.wav`, `game_over.wav`) as `new Audio()` objects at init. Store references on an `assets` object.
- Use a `playSound(audio)` helper that does `audio.currentTime = 0; audio.play().catch(() => {})`. The `.catch` silently handles autoplay restrictions on first interaction.
- Play `jump.wav` on every jump input — even rapid taps. The `currentTime = 0` reset allows overlapping triggers without creating new Audio instances.
- Play `game_over.wav` once on collision. Don't replay on restart input.
- If audio fails to load (`onerror`), set the reference to `null`. The `playSound` helper should check for null and skip silently.
- Keep audio calls in the input handler and game-over transition — not in the physics or render systems. Audio is a side effect and belongs at the edges.

## Screen Shake Mechanics

- On game over collision, apply a brief screen shake by offsetting the canvas translate before rendering.
- Track shake state on `gameState`: `{ shakeTimer: 0, shakeIntensity: 6 }`. Set `shakeTimer` to a duration (e.g., 0.3s) when collision triggers.
- Each frame while `shakeTimer > 0`:
  ```
  offsetX = (Math.random() - 0.5) * 2 * intensity * (shakeTimer / duration)
  offsetY = (Math.random() - 0.5) * 2 * intensity * (shakeTimer / duration)
  ctx.save(); ctx.translate(offsetX, offsetY);
  // ... render everything ...
  ctx.restore();
  shakeTimer -= dt;
  ```
- Intensity decays linearly with the timer — shake starts strong and fades out.
- Only apply shake during the game over state. Never shake during normal gameplay.
- Keep shake parameters in CONFIG: `CONFIG.shake.duration`, `CONFIG.shake.intensity`.

## UI Animation Patterns

- Score text: on increment, scale up to 1.3x over 50ms then ease back to 1.0 over 150ms. Use `ctx.save()` → `ctx.translate(textCenterX, textCenterY)` → `ctx.scale(s, s)` → draw text → `ctx.restore()`.
- "Game Over" text: fade in with the overlay (0 to 1.0 alpha over 300ms). Optionally slide down from 20px above final position using lerp.
- "Tap to start" / "Tap to restart" prompt: pulse opacity between 0.5 and 1.0 using `0.75 + 0.25 * Math.sin(elapsed * 3)`. Runs continuously while waiting for input.
- High score badge: when the current run beats the high score, briefly flash the high score text in a highlight color (e.g., gold `#FFD700`) for 500ms, then fade back to white.
- All UI animations use the same `lerp` and ease-out utilities defined in the game mechanics. Don't create separate animation systems for UI.
