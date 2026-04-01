# Game Coding Standards

## JavaScript Game Patterns

- Use a single `CONFIG` object for all tunable constants (physics, sizing, timing). Expose on `window` for dev console access.
- Structure game logic as pure functions that take state and return new state. Keep side effects (rendering, audio, DOM) at the edges.
- Use `requestAnimationFrame` for the game loop. Track delta time for frame-rate-independent physics.
- Manage game states (start, playing, gameOver) via a simple string status field — no need for a full state machine library.
- Preload all assets (images, audio) before starting the game loop. Provide fallbacks on load failure.

## Naming Conventions

- Functions: `camelCase` — e.g., `updateKiro`, `spawnPipe`, `checkCollision`
- Constants/config: `camelCase` nested under `CONFIG` — e.g., `CONFIG.pipe.speed`
- State objects: `PascalCase` suffix for type names in comments — e.g., `KiroState`, `PipeState`, `CloudState`
- Boolean fields: prefix with `is`, `has`, or `was` when clarity helps — e.g., `pipe.passed` is acceptable for simple flags
- Event handlers: prefix with `handle` — e.g., `handleInput`, `handleKeyDown`
- Files: `kebab-case` — e.g., `game-config.json`, `flappy-kiro.test.js`

## Performance Optimization

- Minimize object allocation per frame. Reuse arrays and objects where possible instead of creating new ones.
- Use `canvas.width` / `canvas.height` reads sparingly — cache in CONFIG or local variables.
- Batch canvas draw calls. Set `fillStyle` / `strokeStyle` once per color group, not per object.
- Avoid `getImageData` / `putImageData` in the hot loop — use geometric collision detection (bounding box) instead of pixel-perfect.
- Remove off-screen entities (pipes, clouds) promptly to keep arrays short.
- Use `Math.max`, `Math.min`, `Math.floor` over branching where applicable — they're well-optimized in V8.
- Keep the render function's draw order consistent: background → clouds → pipes → ground → character → HUD. This avoids unnecessary state changes.
- For audio, call `audio.currentTime = 0` before `play()` to allow rapid re-triggering without creating new Audio instances.

## Entity-Component Patterns

- Represent game entities as plain objects with data-only fields (position, velocity, size, flags). No methods on entities.
- Group entities by type in arrays on the game state — e.g., `gameState.pipes`, `gameState.clouds`.
- Process entities with system functions that iterate over arrays — e.g., `updatePipes(pipes)` updates all pipes, `renderClouds(ctx, clouds)` draws all clouds.
- Keep entity creation in factory functions — e.g., `spawnPipe()` returns a new `PipeState` object. This keeps construction logic centralized.
- Use a `passed` or `active` boolean flag on entities to mark state transitions instead of removing mid-iteration. Filter dead entities after the update pass.

## Game Loop Structure

- Use a single `requestAnimationFrame` callback as the loop driver.
- Compute `deltaTime` as the difference between the current and previous timestamp (in seconds). Cap it to prevent spiral-of-death on tab refocus (e.g., `Math.min(dt, 0.05)`).
- Structure each frame in fixed order: input → update → collision → scoring → render.
- Keep the loop function thin — delegate to named functions for each phase.
- Use `gameState.frameCount` for interval-based spawning (e.g., spawn pipe every N frames). Increment once per frame.
- In non-playing states (start, gameOver), run a reduced loop that only updates cosmetic elements (clouds) and renders — skip physics, pipes, and collision.

## Memory Management

- Pre-allocate entity arrays at game start with a reasonable capacity. Use `splice` or index swapping to remove entities rather than creating new arrays each frame.
- Avoid closures inside the game loop — define handler functions outside and reference them. Closures created per-frame generate garbage.
- Reuse `Audio` objects by resetting `currentTime` instead of constructing new ones.
- For temporary calculations (e.g., collision rects), use module-scoped scratch objects instead of allocating new ones each frame.
- Keep string concatenation out of the hot path — pre-build score display strings only when the score changes, not every frame.
- Avoid `Array.prototype.filter` / `map` in the loop when possible — they allocate new arrays. Use in-place mutation or manual loops.
- Profile with browser DevTools' Performance tab to catch GC pauses. Target zero allocations per frame in steady state.
