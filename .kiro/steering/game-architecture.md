# Game Architecture Standards

## Modular Systems

- Organize code into logical system functions, each responsible for one concern: physics, pipes, clouds, collision, scoring, rendering, audio.
- Each system function operates on the shared `gameState` object. Systems read and mutate state — they don't own it.
- Keep system functions in a predictable call order within the game loop. Document the order at the top of the loop function.
- Factory functions (`spawnPipe`, `initClouds`) create entities. Update functions (`updatePipes`, `updateClouds`) process them. Render functions (`renderPipes`, `renderClouds`) draw them. Keep these three concerns separate.
- If the single-file approach grows unwieldy, extract systems into clearly commented sections with banner comments (e.g., `// === PIPE SYSTEM ===`).
- Avoid circular dependencies between systems. Collision detection reads Kiro and pipe state but never modifies them — the game loop handles state transitions based on collision results.

## Event Handling Patterns

- Register all event listeners once at initialization. Never add or remove listeners during gameplay.
- Use a single input handler function that checks `gameState.status` to determine behavior (jump, start, restart).
- Listen on `document` for keyboard events and on the `canvas` for click/touch events.
- For keyboard input, check `event.code === 'Space'` (not `event.key`) for consistent cross-layout behavior. Call `event.preventDefault()` to stop page scrolling.
- For touch support, listen to `touchstart` and call `event.preventDefault()` to avoid double-firing with click on mobile.
- Queue input rather than acting immediately — set a flag (e.g., `gameState.jumpQueued = true`) that the update phase consumes. This prevents input from being processed at inconsistent points in the frame.
- Debounce restart input in the game over state — ignore clicks/presses for a short delay (e.g., 300ms) after game over triggers to prevent accidental instant restarts.

## State Management

- Use a single `gameState` object as the source of truth. All systems read from and write to this object.
- Game status is a string field: `'start'`, `'playing'`, or `'gameOver'`. Use simple `if`/`switch` checks — no state machine library needed.
- The `resetGame(gameState)` function is the only way to transition from `gameOver` to `playing`. It zeroes the score, repositions Kiro, clears pipes, and sets status.
- Never mutate `CONFIG` during gameplay — it's read-only after initialization (dev console tweaks are the exception).
- Track `frameCount` on `gameState` for interval-based logic (pipe spawning). Reset it in `resetGame`.
- Persist only `highScore` to `localStorage`. All other state is ephemeral and reconstructed on reset.
- Keep derived values (ground Y position, playable height) as computed properties or local variables — don't store them on `gameState` if they can be calculated from `CONFIG`.
