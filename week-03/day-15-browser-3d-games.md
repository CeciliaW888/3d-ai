# Day 15: Browser 3D Games with Three.js

## Learning Objectives

- Understand the game loop and how `THREE.Clock` drives frame-rate-independent updates
- Handle keyboard input to move a player object in 3D space
- Implement basic axis-aligned bounding box (AABB) collision detection
- Understand why complex games need a physics engine and what Rapier.js provides
- Build a complete "collect the cubes" game from scratch with Three.js

## Core Concepts

### The Game Loop

Every real-time game runs one fundamental pattern: update the world, then draw it, repeat forever.
In a browser this is driven by `requestAnimationFrame`, which fires around 60 times per second.
The critical insight is that `requestAnimationFrame` does not guarantee a fixed time step.
A frame can take 8ms or 50ms depending on CPU load.
If you move a player by a fixed amount each frame, the game runs twice as fast on a 120Hz display as a 60Hz display.

The fix is to measure elapsed time and scale all movement by it.
`THREE.Clock` makes this easy:

```js
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta(); // seconds since last frame, e.g. 0.016
  update(delta);
  renderer.render(scene, camera);
}
```

Everything that moves is multiplied by `delta`:
`player.position.x += speed * delta` means "move `speed` units per second" regardless of frame rate.

### Game State Management

A game needs to track what is happening: playing, paused, game-over.
Use a simple state object to avoid writing spaghetti conditionals throughout your loop:

```js
const state = {
  phase: 'playing', // 'playing' | 'paused' | 'gameover'
  score: 0,
  timeLeft: 30,
};
```

Only run the `update` logic when `state.phase === 'playing'`.
The `render` call can run every frame so the screen does not freeze when paused.

### Keyboard Input

Polling is the simplest and most robust approach for game input.
Maintain a `keys` object that records which keys are currently held down.
The `update` function reads this object rather than reacting to individual events.

```js
const keys = {};
window.addEventListener('keydown', (e) => { keys[e.code] = true; });
window.addEventListener('keyup',   (e) => { keys[e.code] = false; });
```

Reading `keys.KeyW` in `update` means "is W currently held?"
This avoids missed inputs that happen when using only `keydown` events (which fire with OS key-repeat delay).

### Collision Detection: AABB

Axis-aligned bounding box collision is the simplest form of 3D collision.
Both objects are treated as boxes aligned with the world axes (no rotation).
Two AABB boxes are overlapping when they overlap on all three axes simultaneously.

```js
function aabbOverlap(a, b, aSize, bSize) {
  return (
    Math.abs(a.x - b.x) < (aSize + bSize) / 2 &&
    Math.abs(a.y - b.y) < (aSize + bSize) / 2 &&
    Math.abs(a.z - b.z) < (aSize + bSize) / 2
  );
}
```

AABB is fast enough for dozens of objects but breaks down for:
- Rotated objects (requires OBB - oriented bounding box)
- Complex mesh shapes (requires convex hull or triangle-mesh collision)
- Objects moving fast enough to pass through each other in one frame (requires continuous collision detection)

### Why Physics Engines Exist

Once you need gravity, friction, joints, stacking, or rolling, hand-written collision gets out of hand quickly.
Rapier.js (`@dimforge/rapier3d-compat`) is a WebAssembly physics engine designed for the browser.
It handles rigid body simulation, multiple collider shapes, forces, and joints.
You describe the physics world to Rapier, then copy the solved positions back to your Three.js meshes each frame.

A basic Rapier setup:

```js
import RAPIER from '@dimforge/rapier3d-compat';

await RAPIER.init(); // Must await - loads the WASM binary
const world = new RAPIER.World({ x: 0, y: -9.81, z: 0 });

// Create a rigid body
const bodyDesc = RAPIER.RigidBodyDesc.dynamic().setTranslation(0, 5, 0);
const body = world.createRigidBody(bodyDesc);

// Attach a collider shape
const colliderDesc = RAPIER.ColliderDesc.ball(0.5);
world.createCollider(colliderDesc, body);

// In the game loop:
world.step();
const pos = body.translation();
mesh.position.set(pos.x, pos.y, pos.z);
```

Today's game uses manual AABB rather than Rapier to keep the setup to zero dependencies.
Day 16 and 17 introduce Rapier for more complex interactions.

## Key Terminology

| Term | Definition |
|------|-----------|
| Game loop | The `requestAnimationFrame` cycle that repeatedly calls update and render at the display's refresh rate |
| `THREE.Clock` | Three.js utility that measures elapsed time between calls to `getDelta()`, enabling frame-rate-independent movement |
| `delta` | Elapsed time in seconds since the last frame, used as a multiplier to make movement speed hardware-independent |
| AABB | Axis-Aligned Bounding Box - a collision shape that is always axis-parallel, fast to compute but limited to box-shaped or approximated objects |
| Rigid body | A physics object treated as a solid, inflexible shape that responds to forces, gravity, and collisions |
| Polling | Reading input state (key held down or not) on each frame, as opposed to reacting to individual key events |
| `requestAnimationFrame` | Browser API that schedules a callback before the next display repaint, the foundation of all web animation and game loops |

## Code Examples

### Complete "Collect the Cubes" Game

Create `game.js` and link it from an HTML file with a `<canvas id="game">` element:

```js
import * as THREE from 'three';

/* ------------------------------------------------------------------
   SETUP
   ------------------------------------------------------------------ */
const canvas = document.getElementById('game');
const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a2e);
scene.fog = new THREE.Fog(0x1a1a2e, 10, 40);

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  100
);
camera.position.set(0, 6, 10);
camera.lookAt(0, 0, 0);

/* ------------------------------------------------------------------
   LIGHTING
   ------------------------------------------------------------------ */
const ambient = new THREE.AmbientLight(0x404060, 0.6);
scene.add(ambient);

const sun = new THREE.DirectionalLight(0xffffff, 1.2);
sun.position.set(5, 10, 5);
sun.castShadow = true;
sun.shadow.mapSize.set(1024, 1024);
scene.add(sun);

/* ------------------------------------------------------------------
   GROUND
   ------------------------------------------------------------------ */
const groundGeo = new THREE.PlaneGeometry(30, 30);
const groundMat = new THREE.MeshStandardMaterial({ color: 0x16213e });
const ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

/* ------------------------------------------------------------------
   PLAYER
   ------------------------------------------------------------------ */
const playerGeo = new THREE.BoxGeometry(0.8, 0.8, 0.8);
const playerMat = new THREE.MeshStandardMaterial({ color: 0x00d4ff });
const player = new THREE.Mesh(playerGeo, playerMat);
player.position.set(0, 0.4, 0);
player.castShadow = true;
scene.add(player);

const PLAYER_SPEED = 5; // units per second
const PLAYER_SIZE  = 0.8;

/* ------------------------------------------------------------------
   COLLECTIBLE CUBES
   ------------------------------------------------------------------ */
const CUBE_COUNT = 12;
const CUBE_SIZE  = 0.6;
const cubes      = [];

const cubeMat = new THREE.MeshStandardMaterial({
  color: 0xffd700,
  emissive: 0x554400,
  roughness: 0.3,
  metalness: 0.7,
});

function spawnCube() {
  const geo  = new THREE.BoxGeometry(CUBE_SIZE, CUBE_SIZE, CUBE_SIZE);
  const mesh = new THREE.Mesh(geo, cubeMat.clone());
  mesh.castShadow = true;

  // Random position on the ground plane
  mesh.position.set(
    (Math.random() - 0.5) * 24,
    CUBE_SIZE / 2,
    (Math.random() - 0.5) * 24
  );
  scene.add(mesh);
  cubes.push(mesh);
}

for (let i = 0; i < CUBE_COUNT; i++) spawnCube();

/* ------------------------------------------------------------------
   INPUT
   ------------------------------------------------------------------ */
const keys = {};
window.addEventListener('keydown', (e) => { keys[e.code] = true; });
window.addEventListener('keyup',   (e) => { keys[e.code] = false; });

/* ------------------------------------------------------------------
   GAME STATE
   ------------------------------------------------------------------ */
const state = {
  phase:    'playing',  // 'playing' | 'gameover'
  score:    0,
  timeLeft: 30,
};

/* ------------------------------------------------------------------
   UI
   ------------------------------------------------------------------ */
const uiScore    = document.getElementById('score');
const uiTimer    = document.getElementById('timer');
const uiMessage  = document.getElementById('message');

function updateUI() {
  if (uiScore)   uiScore.textContent   = `Score: ${state.score}`;
  if (uiTimer)   uiTimer.textContent   = `Time: ${Math.ceil(state.timeLeft)}s`;
  if (uiMessage) uiMessage.textContent = state.phase === 'gameover'
    ? `Game Over! Final score: ${state.score}`
    : '';
}

/* ------------------------------------------------------------------
   AABB COLLISION
   ------------------------------------------------------------------ */
function aabbOverlap(posA, posB, sizeA, sizeB) {
  const half = (sizeA + sizeB) / 2;
  return (
    Math.abs(posA.x - posB.x) < half &&
    Math.abs(posA.y - posB.y) < half &&
    Math.abs(posA.z - posB.z) < half
  );
}

/* ------------------------------------------------------------------
   UPDATE (called every frame with delta in seconds)
   ------------------------------------------------------------------ */
function update(delta) {
  if (state.phase !== 'playing') return;

  // --- Timer ---
  state.timeLeft -= delta;
  if (state.timeLeft <= 0) {
    state.timeLeft = 0;
    state.phase = 'gameover';
    updateUI();
    return;
  }

  // --- Player movement ---
  const move = new THREE.Vector3();
  if (keys['KeyW'] || keys['ArrowUp'])    move.z -= 1;
  if (keys['KeyS'] || keys['ArrowDown'])  move.z += 1;
  if (keys['KeyA'] || keys['ArrowLeft'])  move.x -= 1;
  if (keys['KeyD'] || keys['ArrowRight']) move.x += 1;

  if (move.lengthSq() > 0) {
    move.normalize().multiplyScalar(PLAYER_SPEED * delta);
    player.position.add(move);
    // Clamp to ground plane bounds
    player.position.x = THREE.MathUtils.clamp(player.position.x, -14, 14);
    player.position.z = THREE.MathUtils.clamp(player.position.z, -14, 14);
  }

  // Rotate player in movement direction (visual feedback)
  if (move.lengthSq() > 0) {
    player.rotation.y = Math.atan2(move.x, move.z);
  }

  // --- Collectible collision ---
  for (let i = cubes.length - 1; i >= 0; i--) {
    const cube = cubes[i];
    if (aabbOverlap(player.position, cube.position, PLAYER_SIZE, CUBE_SIZE)) {
      scene.remove(cube);
      cube.geometry.dispose();
      cubes.splice(i, 1);
      state.score += 10;

      // Spawn a new cube to keep the count up
      if (cubes.length < CUBE_COUNT) spawnCube();
    }
  }

  // Animate collectibles (spin + bob)
  const t = performance.now() * 0.001;
  cubes.forEach((cube, i) => {
    cube.rotation.y += delta * 1.5;
    cube.position.y = CUBE_SIZE / 2 + Math.sin(t * 2 + i) * 0.1;
  });

  // Camera follows player
  camera.position.x = player.position.x;
  camera.position.z = player.position.z + 10;
  camera.lookAt(player.position);

  updateUI();
}

/* ------------------------------------------------------------------
   GAME LOOP
   ------------------------------------------------------------------ */
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta(); // seconds since last call
  update(delta);
  renderer.render(scene, camera);
}

/* ------------------------------------------------------------------
   RESIZE
   ------------------------------------------------------------------ */
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// Start
animate();
```

### Minimal HTML wrapper

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Collect the Cubes</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { overflow: hidden; background: #000; }
    canvas { display: block; }
    #hud {
      position: fixed;
      top: 1rem;
      left: 1rem;
      color: #fff;
      font-family: monospace;
      font-size: 1.2rem;
      pointer-events: none;
    }
    #message {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: #ffd700;
      font-family: monospace;
      font-size: 2rem;
      font-weight: bold;
      pointer-events: none;
    }
  </style>
</head>
<body>
  <canvas id="game"></canvas>
  <div id="hud">
    <div id="score">Score: 0</div>
    <div id="timer">Time: 30s</div>
  </div>
  <div id="message"></div>
  <script type="module" src="game.js"></script>
</body>
</html>
```

## Hands-On Exercises

1. Get the game running in a Vite project.
   Add `three` as a dependency and test that WASD movement and collection both work.
2. Increase `PLAYER_SPEED` from 5 to 10 and observe how `delta` scaling keeps the movement feeling the same on different frame rates (open DevTools throttling to verify).
3. Add a "collectible counter" that counts how many cubes remain in the scene, displayed in the HUD.
4. Add a power-up cube (a different color) that appears every 10 seconds and grants 3 seconds of double-speed movement when collected.
5. Read the Rapier.js quickstart guide and add a falling sphere to the scene that bounces on the ground using the Rapier physics world instead of manual positioning.

## Resources

- [Three.js Clock Docs](https://threejs.org/docs/#api/en/core/Clock) - `getDelta`, `getElapsedTime`, and `autoStart`
- [Rapier.js User Guide](https://rapier.rs/docs/user_guides/javascript/getting_started_js) - physics engine quickstart
- [MDN KeyboardEvent.code Reference](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code/code_values) - all key code strings for input mapping
- [Three.js MathUtils](https://threejs.org/docs/#api/en/math/MathUtils) - `clamp`, `lerp`, `randFloat`, and other game math helpers
- [Game Programming Patterns (free book)](https://gameprogrammingpatterns.com/) - timeless patterns for game architecture, all applicable to Three.js games

## Reflection Questions

1. Why does multiplying movement by `delta` make a game frame-rate independent?
   What would happen if you forgot to multiply?
2. What is the difference between a `keydown` event listener and the polling approach used in this game?
   When does the polling approach give better results?
3. Describe one scenario where AABB collision would give the wrong result, and explain what collision method would fix it.
4. What problem does a physics engine like Rapier solve that manual AABB collision cannot?
5. How does the camera-follows-player code work, and what would you change to add a smooth lag effect?

## Next Steps

Day 16 covers AI game asset creation - generating 3D characters, textures, and environments with AI tools to populate your game world without modeling by hand.

- Previous: [Day 14 - Week 2 Capstone](../week-02/day-14-week2-capstone.md)
- Next: Day 16 - AI Game Asset Creation
