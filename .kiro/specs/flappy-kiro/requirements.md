# Requirements Document

## Introduction

Flappy Kiro is a retro browser-based endless side-scroller game inspired by Flappy Bird. The player controls a ghost character ("Kiro") that must navigate through gaps between pairs of green pipes by tapping or clicking to jump. The game features a hand-drawn sketchy aesthetic with a light blue sky, floating clouds, and a dark ground area. The game tracks the player's current score and high score across sessions.

## Glossary

- **Game**: The Flappy Kiro browser application, implemented as a single HTML page with CSS and JavaScript
- **Player**: The human user interacting with the Game
- **Kiro**: The ghost character sprite controlled by the Player, rendered from the `assets/ghosty.png` image
- **Pipe_Pair**: A pair of vertical green obstacles (one extending from the top, one from the bottom) with a gap between them
- **Gap**: The vertical opening between the top and bottom pipes of a Pipe_Pair that Kiro must pass through
- **Score**: The number of Pipe_Pairs that Kiro has successfully passed through in the current run
- **High_Score**: The highest Score achieved across all game sessions, persisted in browser local storage
- **Ground**: The dark-colored floor area at the bottom of the game screen that acts as a collision boundary
- **Game_Canvas**: The HTML5 Canvas element where all game graphics are rendered
- **Game_Loop**: The continuous animation cycle that updates game state and renders frames

## Requirements

### Requirement 1: Game Initialization

**User Story:** As a Player, I want the game to load in my browser and display a start screen, so that I can begin playing when ready.

#### Acceptance Criteria

1. THE Game SHALL render on an HTML5 Canvas element within a single HTML page using CSS and JavaScript
2. WHEN the Game is loaded, THE Game SHALL display Kiro at a default starting position on the left side of the Game_Canvas
3. WHEN the Game is loaded, THE Game SHALL display a light blue sky background with floating white cloud elements and a dark Ground area at the bottom
4. WHEN the Game is loaded, THE Game SHALL wait for Player input before starting the Game_Loop

### Requirement 2: Player Input and Jump Mechanic

**User Story:** As a Player, I want to make Kiro jump by clicking or pressing a key, so that I can navigate through obstacles.

#### Acceptance Criteria

1. WHEN the Player clicks the mouse or presses the spacebar, THE Game SHALL apply an upward velocity to Kiro
2. WHEN Kiro jumps, THE Game SHALL play the `assets/jump.wav` sound effect
3. WHILE the Game_Loop is running, THE Game SHALL apply gravity to Kiro, causing downward acceleration each frame
4. THE Game SHALL constrain Kiro's vertical position so that Kiro does not move above the top edge of the Game_Canvas

### Requirement 3: Pipe Obstacle Generation

**User Story:** As a Player, I want pipes to appear as obstacles, so that the game provides a continuous challenge.

#### Acceptance Criteria

1. WHILE the Game_Loop is running, THE Game SHALL generate Pipe_Pairs at regular horizontal intervals that scroll from right to left
2. THE Game SHALL render each Pipe_Pair as two green rectangular pipes (top and bottom) with a Gap between them
3. THE Game SHALL randomize the vertical position of the Gap for each Pipe_Pair within a range that keeps the Gap fully visible on screen
4. THE Game SHALL maintain a consistent Gap size across all Pipe_Pairs that allows Kiro to pass through
5. WHEN a Pipe_Pair scrolls completely off the left edge of the Game_Canvas, THE Game SHALL remove the Pipe_Pair from memory

### Requirement 4: Collision Detection

**User Story:** As a Player, I want the game to detect when Kiro hits an obstacle, so that the game ends fairly.

#### Acceptance Criteria

1. WHILE the Game_Loop is running, THE Game SHALL check for collisions between Kiro and each Pipe_Pair every frame
2. WHILE the Game_Loop is running, THE Game SHALL check for collisions between Kiro and the Ground every frame
3. WHEN Kiro collides with a Pipe_Pair or the Ground, THE Game SHALL transition to the game over state

### Requirement 5: Scoring

**User Story:** As a Player, I want to see my score increase as I pass pipes, so that I can track my progress.

#### Acceptance Criteria

1. WHEN Kiro passes through a Pipe_Pair Gap successfully, THE Game SHALL increment the Score by one
2. THE Game SHALL display the current Score and the High_Score at the bottom of the Game_Canvas in the format "Score: [value] | High: [value]"
3. WHEN the Score exceeds the High_Score, THE Game SHALL update the High_Score to match the Score
4. THE Game SHALL persist the High_Score to browser local storage so that the High_Score is retained across browser sessions

### Requirement 6: Game Over State

**User Story:** As a Player, I want to know when the game is over and be able to restart, so that I can try again.

#### Acceptance Criteria

1. WHEN the game over state is triggered, THE Game SHALL play the `assets/game_over.wav` sound effect
2. WHEN the game over state is triggered, THE Game SHALL stop the Game_Loop and display a game over message on the Game_Canvas
3. WHEN the game over state is triggered, THE Game SHALL save the High_Score to local storage
4. WHEN the Player clicks or presses the spacebar during the game over state, THE Game SHALL reset the Score to zero, reposition Kiro, clear all Pipe_Pairs, and restart the Game_Loop

### Requirement 7: Visual Style and Background

**User Story:** As a Player, I want the game to have a retro hand-drawn aesthetic, so that the experience feels charming and distinctive.

#### Acceptance Criteria

1. THE Game SHALL render the background with a light blue sky color
2. THE Game SHALL render floating cloud shapes in the background with varying opacity levels (semi-transparent) so that clouds appear at different visual depths
3. THE Game SHALL scroll each cloud shape from right to left at a different speed, with more opaque clouds moving faster and more transparent clouds moving slower, to create a parallax perspective effect
4. THE Game SHALL render the Ground as a dark-colored strip at the bottom of the Game_Canvas
5. THE Game SHALL render Kiro using the `assets/ghosty.png` sprite image
6. THE Game SHALL render Pipe_Pairs in a green color with a retro sketchy visual style

### Requirement 8: Endless Scrolling

**User Story:** As a Player, I want the game world to scroll continuously, so that the game feels like an endless journey.

#### Acceptance Criteria

1. WHILE the Game_Loop is running, THE Game SHALL scroll the background, Ground, and clouds continuously from right to left, with each cloud layer moving at its own designated speed
2. WHILE the Game_Loop is running, THE Game SHALL maintain a constant horizontal scroll speed for Pipe_Pairs
3. THE Game SHALL seamlessly loop the background and Ground elements so that no visual gaps appear during scrolling
