# Day 18: Performance Optimization for Web 3D

## Learning Objectives

- Identify the four main causes of slow web 3D: draw calls, polygon count, texture memory, and JS overhead.
- Use Chrome DevTools, the Stats panel, and Spector.js to profile a scene.
- Reduce draw calls dramatically with `THREE.InstancedMesh`.
- Compress assets with Draco (geometry) and KTX2/Basis (textures) before deploying.
- Apply frustum culling and lazy loading to avoid rendering what the user cannot see.

## Core Concepts

### Why Web 3D Gets Slow

The GPU is fast - but getting data to the GPU is not.
Every time the CPU tells the GPU "draw this object", that is one **draw call**.
Draw calls have overhead: the CPU must prepare a state change, send geometry and material data through the WebGL API, and the GPU pipeline must restart.
On a typical device, more than 200-500 draw calls per frame will cause noticeable stutter.

The four main bottlenecks are:

1. **Too many draw calls** - each separate mesh with a separate material is a draw call.
2. **Too many polygons** - the GPU must process every triangle, every frame, for every visible object.
3. **Too much texture memory** - uncompressed 2048x2048 RGBA textures consume 16 MB of VRAM each.
4. **JavaScript overhead** - your `requestAnimationFrame` loop, physics updates, and state management run on the main thread alongside rendering.

### Profiling Tools

Before optimizing, always measure first.
Guess-and-fix wastes time; data-driven fixes solve the real bottleneck.

**Three.js Stats panel:**

```js
import Stats from 'three/addons/libs/stats.module.js';

const stats = new Stats();
document.body.appendChild(stats.dom);

function animate() {
  requestAnimationFrame(animate);
  stats.begin();
  renderer.render(scene, camera);
  stats.end();
}
```

The Stats panel shows FPS (frames per second), frame time in ms, and memory usage.
Target: 60 FPS (16.6 ms per frame). Below 30 FPS needs fixing.

**Chrome DevTools Performance tab:** Press F12 > Performance > Record a few seconds.
Look for long JS frames (yellow bars), long GPU frames (green bars), and excessive garbage collection (white bars).

**Spector.js:** A browser extension that captures a single WebGL frame and shows every draw call, every shader, and every texture.
Install from the Chrome Web Store.
The draw call count in the bottom-left corner after capture is your baseline to beat.

**renderer.info:** Three.js exposes real-time stats without any extra library:

```js
console.log(renderer.info.render);
// { calls: 47, triangles: 124500, points: 0, lines: 0 }
```

Log this every 60 frames and watch the `calls` number.

### InstancedMesh: One Draw Call for 1,000 Objects

`THREE.InstancedMesh` renders multiple copies of the same geometry+material in a single draw call.
Each instance gets its own transformation matrix (position, rotation, scale) stored in a GPU buffer.

```js
import * as THREE from 'three';

const count = 1000;
const geometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);
const material = new THREE.MeshStandardMaterial({ color: 0x0d9488 });

// One mesh, 1000 instances - costs exactly 1 draw call
const instancedMesh = new THREE.InstancedMesh(geometry, material, count);

const matrix = new THREE.Matrix4();

for (let i = 0; i < count; i++) {
  // Position each instance randomly in a 40x40x40 cube
  matrix.setPosition(
    (Math.random() - 0.5) * 40,
    (Math.random() - 0.5) * 40,
    (Math.random() - 0.5) * 40
  );
  instancedMesh.setMatrixAt(i, matrix);
}

// Tell Three.js to upload the matrix buffer to the GPU
instancedMesh.instanceMatrix.needsUpdate = true;

scene.add(instancedMesh);
```

Without instancing: 1,000 separate `Mesh` objects = 1,000 draw calls.
With instancing: 1 `InstancedMesh` = 1 draw call.
The difference on a mid-range device can be 60 FPS vs 8 FPS.

### Geometry Merging

When you have many static objects that share a material but are not identical geometry, merge them into one `BufferGeometry`:

```js
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js';

const geometries = [];
for (let i = 0; i < 200; i++) {
  const geo = new THREE.BoxGeometry(1, 1, 1);
  const matrix = new THREE.Matrix4().setPosition(
    Math.random() * 50 - 25,
    0,
    Math.random() * 50 - 25
  );
  geo.applyMatrix4(matrix);
  geometries.push(geo);
}

const merged = mergeGeometries(geometries);
const mesh = new THREE.Mesh(merged, new THREE.MeshStandardMaterial({ color: 0x888888 }));
scene.add(mesh);
// Result: 1 draw call for 200 objects
```

Note: merged objects can no longer be moved individually.
Use this only for static scene elements like floors, walls, and terrain chunks.

### Texture and Asset Compression

**Draco compression for GLTF geometry:**
Reduces file size 5-10x with minimal quality loss.

```bash
npx @gltf-transform/cli optimize scene.glb scene.optimized.glb \
  --compress draco \
  --texture-compress webp
```

Load Draco-compressed files with `DRACOLoader`:

```js
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const draco = new DRACOLoader();
draco.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');
const loader = new GLTFLoader();
loader.setDRACOLoader(draco);
```

**KTX2 / Basis textures:**
GPU-native compressed textures stay compressed in VRAM (regular PNG/JPG are decompressed at upload time, consuming full RGBA memory).
A 2048x2048 PNG = 16 MB VRAM.
The same image as KTX2 ETC1S = ~0.5 MB VRAM.
Generate KTX2 files with: `npx @gltf-transform/cli optimize --texture-compress ktx2`.

### Frustum Culling and Lazy Loading

**Frustum culling** is automatic in Three.js: objects outside the camera's view frustum are skipped at the CPU level.
Make sure you do not disable it: `mesh.frustumCulled = true` (the default).
For large outdoor scenes, divide the world into chunks and only add chunks near the player to the scene.

**Lazy loading:** do not load all GLB files on page load.
Load models when the user navigates near them:

```js
const loader = new GLTFLoader();
let treeLoaded = false;

function checkProximity(playerPos, treePos) {
  if (!treeLoaded && playerPos.distanceTo(treePos) < 30) {
    treeLoaded = true;
    loader.load('tree.glb', gltf => scene.add(gltf.scene));
  }
}
```

## Key Terminology

| Term | Definition |
|------|-----------|
| Draw call | A single GPU instruction to render a mesh - minimizing these is the #1 optimization |
| InstancedMesh | Three.js class that renders N copies of one geometry+material in 1 draw call |
| Draco compression | Lossless compression for GLTF geometry data, reducing file size 5-10x |
| KTX2 | GPU-native compressed texture format; stays compressed in VRAM |
| Frustum culling | Skipping objects outside the camera's visible cone before sending to GPU |
| Geometry merging | Combining multiple static geometries into one BufferGeometry |
| Spector.js | Browser extension that captures and inspects a single WebGL frame |

## Code Examples

See `InstancedMesh` example in Core Concepts above.
The geometry merging snippet and Draco loading code are also in Core Concepts and are copy-paste ready.

## Hands-On Exercises

1. **Measure your baseline.** Add `renderer.info` logging to any Three.js scene.
   Record the draw call count before and after adding 50 separate cube meshes.

2. **Implement InstancedMesh.** Replace those 50 separate cubes with one `THREE.InstancedMesh` with 50 instances.
   Log `renderer.info.render.calls` again and compare.

3. **Profile with Stats.** Add the Stats panel to your scene.
   Load a complex GLB model and observe the FPS drop.
   Enable `DRACOLoader` and compare load time.

4. **Install Spector.js.** Install it from the Chrome Web Store (search "Spector.js").
   Open your Three.js project, click the extension icon, and capture a frame.
   Find the total draw call count in the capture summary.

5. **Compress a GLB.** Take any GLB file and run `npx @gltf-transform/cli optimize input.glb output.glb --compress draco`.
   Compare the file sizes before and after.
   Check the compressed version still loads correctly in your scene.

## Resources

- [Three.js InstancedMesh documentation](https://threejs.org/docs/#api/en/objects/InstancedMesh)
- [glTF-Transform CLI - optimization toolchain](https://gltf-transform.dev/)
- [gltf.report - browser-based GLTF analyzer](https://gltf.report/)
- [Spector.js WebGL debugger](https://spector.babylonjs.com/)
- [Three.js renderer.info reference](https://threejs.org/docs/#api/en/renderers/WebGLRenderer.info)

## Reflection Questions

1. You have a forest scene with 500 identical trees scattered at different positions.
   What is the most effective single optimization you can apply, and why?
2. Why does geometry merging help performance, but also limit interactivity?
   Give a concrete example of where you would and would not use it.
3. A scene runs at 60 FPS on desktop but 10 FPS on mobile.
   List three different techniques you would try first, in order of expected impact.
4. What is the difference between a file being compressed on disk (Draco) versus a texture being compressed in VRAM (KTX2)?
   Why does the VRAM compression matter even when loading times are fast?
5. `mesh.frustumCulled = false` disables frustum culling for that mesh.
   When might you intentionally set this, and what would be the performance consequence?

## Next Steps

Day 19 covers the Blender-to-web export pipeline.
You will learn to prepare assets in Blender specifically for the web: applying modifiers, baking textures, exporting with proper scale, and handling materials that look correct in Three.js.

- Previous: [Day 17 - GLSL Shaders: Visual Magic](day-17-glsl-shaders.md)
- Next: [Day 19 - Blender to Web: Export Pipeline](day-19-blender-to-web.md)
