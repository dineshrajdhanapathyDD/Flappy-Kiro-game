# Canvas API & Collision Patterns

## Canvas API Patterns

- Get the 2D context once at init and pass it as a parameter — never call `getContext('2d')` per frame.
- Use `ctx.clearRect(0, 0, width, height)` at the start of each frame. Avoid `canvas.width = canvas.width` as a reset trick — it's slower and resets all state.
- Group draws by fill/stroke style. Set `ctx.fillStyle` once, then draw all objects of that color before switching.
- Use `ctx.save()` / `ctx.restore()` sparingly — only when you need transforms or clipping. They have a cost.
- For semi-transparent elements (clouds), set `ctx.globalAlpha` before drawing and restore it after. Prefer this over `rgba()` strings when opacity varies per entity.
- Draw images with `ctx.drawImage(img, x, y, w, h)` — always specify width and height to avoid layout recalculation.
- For text rendering (score HUD), set `ctx.font`, `ctx.fillStyle`, and `ctx.textAlign` once per frame, not per text call.
- Use `ctx.fillRect` for simple shapes (pipes, ground) — it's faster than building paths for rectangles.
- For rounded shapes (clouds), use `ctx.ellipse()` or `ctx.arc()` with `ctx.fill()`. Avoid `bezierCurveTo` unless you need custom shapes.

## Animation Frame Handling

- Use `requestAnimationFrame(callback)` as the sole loop driver. Store the returned ID for cancellation on game over.
- Compute delta time: `const dt = (timestamp - lastTimestamp) / 1000`. Store `lastTimestamp` in module scope.
- Cap delta time to prevent physics explosions after tab switches: `const dt = Math.min(rawDt, 0.05)`.
- Use delta time for all movement: `entity.x -= speed * dt`. This keeps behavior consistent across refresh rates.
- Cancel the animation frame on game over with `cancelAnimationFrame(rafId)`. Restart with a fresh `requestAnimationFrame` call on reset.
- On the start screen, run a lightweight animation loop for cosmetic elements only (cloud drift). Use the same `requestAnimationFrame` pattern.
- Avoid `setInterval` or `setTimeout` for game timing — they don't sync with the display refresh and cause visual jitter.

## Collision Detection

- Use axis-aligned bounding box (AABB) for all collision checks. It's sufficient for rectangular entities (pipes, ground, character sprite).
- AABB overlap test: two rects collide when `a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y`.
- For Kiro vs pipes: test against two rects per pipe pair (top pipe and bottom pipe). Derive pipe rects from `gapY` and `gapSize`.
- For Kiro vs ground: simple check — `kiro.y + kiro.height >= CONFIG.canvas.height - CONFIG.ground.height`.
- Early-exit optimization: skip collision check for pipes whose right edge is behind Kiro's left edge (already passed) or whose left edge is far ahead (not yet reachable).
- Keep collision logic as a pure function: `checkCollision(kiro, pipes, groundY) → boolean`. No side effects — state transitions happen in the caller.
- Use a small inset (1-2px) on the character's bounding box for a more forgiving feel. Define this as `CONFIG.kiro.hitboxInset`.
- Never use pixel-perfect collision in the game loop — `getImageData` is far too expensive at 60fps.
