# Implementation Plan: Flappy Kiro

## Overview

Build a single-file browser game (index.html) with embedded CSS and JavaScript using the HTML5 Canvas API. Implementation proceeds bottom-up: CONFIG and data models first, then pure logic functions (physics, pipes, clouds, collision, scoring), then rendering and audio, and finally wiring everything into the game loop. Property-based tests use fast-check and run via Node.js.

## Tasks

- [x] 1. Scaffold HTML file with CONFIG and data models
  - [x] 1.1 Create `index.html` with HTML boilerplate, canvas element, embedded CSS for centering, and the `CONFIG` object
    - Define the full `CONFIG` object as specified in the design (canvas, gravity, jumpVelocity, pipe, ground, kiro, cloud)
    - Create the canvas element (480×640) and obtain the 2D rendering context
    - Preload `assets/ghosty.png` as an Image, `assets/jump.wav` and `assets/game_over.wav` as Audio objects with `onerror` fallbacks
    - Initialize the `gameState` object with status `'start'`, score 0, highScore from localStorage, default KiroState, empty pipes array, and initial clouds
    - _Requirements: 1.1, 1.2, 1.3, 7.5_

- [x] 2. Implement core game logic functions
  - [x] 2.1 Implement Physics Engine — `updateKiro(kiro)` function
    - Apply `CONFIG.gravity` to `kiro.velocity`, update `kiro.y`, clamp `kiro.y` to >= 0
    - Implement jump by setting `kiro.velocity = CONFIG.jumpVelocity`
    - _Requirements: 2.1, 2.3, 2.4_

  - [ ]* 2.2 Write property test: Jump sets upward velocity
    - **Property 1: Jump sets upward velocity**
    - **Validates: Requirements 2.1**

  - [ ]* 2.3 Write property test: Gravity accumulates downward velocity
    - **Property 2: Gravity accumulates downward velocity**
    - **Validates: Requirements 2.3**

  - [ ]* 2.4 Write property test: Kiro is clamped to canvas top
    - **Property 3: Kiro is clamped to canvas top**
    - **Validates: Requirements 2.4**

  - [x] 2.5 Implement Pipe Manager — `spawnPipe()`, `updatePipes(pipes)`, `checkPipePass(kiro, pipes)`
    - `spawnPipe()` creates a PipeState with randomized gapY within valid range, gapSize = CONFIG.pipe.gap, width = CONFIG.pipe.width
    - `updatePipes(pipes)` moves each pipe left by CONFIG.pipe.speed and removes pipes whose right edge < 0
    - `checkPipePass(kiro, pipes)` detects when kiro.x passes a pipe's right edge, marks pipe as passed, returns newly passed count
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 5.1_

  - [ ]* 2.6 Write property test: Pipe generation produces valid gaps
    - **Property 4: Pipe generation produces valid gaps**
    - **Validates: Requirements 3.3, 3.4**

  - [ ]* 2.7 Write property test: Pipes move left at constant speed
    - **Property 5: Pipes move left at constant speed**
    - **Validates: Requirements 3.1, 8.2**

  - [ ]* 2.8 Write property test: Off-screen pipes are removed
    - **Property 6: Off-screen pipes are removed**
    - **Validates: Requirements 3.5**

  - [x] 2.9 Implement Collision Detector — `checkCollision(kiro, pipes)`
    - Return true if Kiro's bounding box overlaps any pipe rectangle (top pipe or bottom pipe) or if Kiro's bottom edge >= ground y
    - Ground y = CONFIG.canvas.height - CONFIG.ground.height
    - _Requirements: 4.1, 4.2, 4.3_

  - [ ]* 2.10 Write property test: Collision detection correctness
    - **Property 7: Collision detection correctness**
    - **Validates: Requirements 4.1, 4.2**

- [x] 3. Implement scoring, high score, and cloud systems
  - [x] 3.1 Implement scoring and high score functions
    - `formatScore(score, highScore)` returns `"Score: {score} | High: {highScore}"`
    - `updateHighScore(score, highScore)` returns `Math.max(score, highScore)`
    - `saveHighScore(score)` writes to localStorage key `flappyKiroHigh` with try-catch
    - `loadHighScore()` reads from localStorage, parses with parseInt, falls back to 0
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 6.3_

  - [ ]* 3.2 Write property test: Score increments on pipe pass
    - **Property 8: Score increments on pipe pass**
    - **Validates: Requirements 5.1**

  - [ ]* 3.3 Write property test: Score display format
    - **Property 9: Score display format**
    - **Validates: Requirements 5.2**

  - [ ]* 3.4 Write property test: High score is monotonically non-decreasing
    - **Property 10: High score is monotonically non-decreasing**
    - **Validates: Requirements 5.3**

  - [ ]* 3.5 Write property test: High score persistence round trip
    - **Property 11: High score persistence round trip**
    - **Validates: Requirements 5.4**

  - [x] 3.6 Implement Cloud Manager — `initClouds()`, `updateClouds(clouds)`
    - `initClouds()` generates CONFIG.cloud.count clouds with random positions, opacity in [minOpacity, maxOpacity], speed derived from opacity (higher opacity = faster speed)
    - `updateClouds(clouds)` moves each cloud left by its speed, wraps to right side when off-screen
    - _Requirements: 7.2, 7.3, 8.1, 8.3_

  - [ ]* 3.7 Write property test: Cloud parallax — opacity correlates with speed
    - **Property 13: Cloud parallax — opacity correlates with speed**
    - **Validates: Requirements 7.2, 7.3, 8.1**

  - [ ]* 3.8 Write property test: Cloud wrapping prevents off-screen drift
    - **Property 14: Cloud wrapping prevents off-screen drift**
    - **Validates: Requirements 8.3**

  - [x] 3.9 Implement `resetGame(gameState)` function
    - Reset score to 0, reposition Kiro to default, clear pipes array, set status to `'playing'`
    - _Requirements: 6.4_

  - [ ]* 3.10 Write property test: Game reset produces clean initial state
    - **Property 12: Game reset produces clean initial state**
    - **Validates: Requirements 6.4**

- [ ] 4. Checkpoint — Verify all logic functions
  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implement renderer and audio
  - [x] 5.1 Implement the `render(ctx, gameState, assets)` function
    - Draw layers in order: sky background (light blue fill), clouds (semi-transparent ellipses), pipes (green rectangles with sketchy style), ground (dark rectangle), Kiro sprite (ghosty.png or fallback rectangle), HUD score text, overlay messages (start prompt or game over message)
    - _Requirements: 1.2, 1.3, 5.2, 7.1, 7.2, 7.4, 7.5, 7.6_

  - [x] 5.2 Implement Audio Manager — `playSound(audioElement)` function
    - Rewind audio to start and play; handle cases where audio failed to load
    - _Requirements: 2.2, 6.1_

- [x] 6. Wire game loop and input handling
  - [x] 6.1 Implement Input Handler
    - Add event listeners for `click`, `keydown` (Space), and `touchstart`
    - In `start` state: call `resetGame()`, start game loop
    - In `playing` state: apply jump velocity, play jump.wav
    - In `gameOver` state: call `resetGame()`, restart game loop
    - _Requirements: 1.4, 2.1, 2.2, 6.4_

  - [x] 6.2 Implement the main game loop using `requestAnimationFrame`
    - Each frame: update Kiro physics, spawn pipes on interval (using frameCount), update pipes, update clouds, check collisions (transition to gameOver on hit, play game_over.wav, save high score), check pipe passes (increment score, update high score), render frame
    - On game over: stop the loop, display game over overlay
    - _Requirements: 1.4, 2.3, 3.1, 4.1, 4.2, 4.3, 5.1, 5.3, 6.1, 6.2, 6.3, 8.1, 8.2, 8.3_

  - [x] 6.3 Implement start screen rendering
    - On load: render initial scene with Kiro at default position, clouds drifting, start prompt text
    - Animate clouds on the start screen using a separate animation loop or the same loop in idle mode
    - _Requirements: 1.2, 1.3, 1.4_

- [ ] 7. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Property tests use fast-check and validate universal correctness properties from the design
- Unit tests validate specific examples and edge cases
- All code lives in a single `index.html` file; test file is separate (e.g., `tests/flappy-kiro.test.js`)
