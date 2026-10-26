# Day 4: Three.js Fundamentals

## Learning Objectives
- Set up a Vite + Three.js project from scratch using ES modules
- Understand the Scene, Camera, and Renderer triad that every Three.js project shares
- Create 3D geometry with different materials and animate it in a render loop
- Add interactive camera controls with OrbitControls
- Load a GLB model into a scene with GLTFLoader

## Core Concepts

### Why Three.js?

WebGL is the browser's raw 3D graphics API.
It gives you direct access to the GPU, but writing WebGL by hand is extremely verbose - a simple colored triangle takes over 100 lines of GLSL shader code and JavaScript boilerplate.
Three.js wraps all of that into a clean, intuitive API that feels like working with objects rather than GPU instructions.
You still get full GPU power, but you write a fraction of the code.

### The Three Core Objects

Every Three.js scene requires exactly three things to render anything on screen.

**Scene** - the container for everything visible.
Think of it as the 3D world itself.
You add objects, lights, and cameras to the scene.
The scene graph is hierarchical - objects can have children, forming a tree.

**Camera** - your viewing point into the scene.
The most common type is `PerspectiveCamera`, which works like a real camera with a field of view (FOV) and near/far clipping planes.
The FOV controls how wide the view is - 75 degrees is a natural human-like view.

**Renderer** - converts the scene and camera into pixels on screen.
`WebGLRenderer` is the standard choice.
It takes a `<canvas>` element and draws to it each frame.
The renderer needs to know the size of your output with `renderer.setSize(width, height)`.

### Geometry and Materials

In Three.js, a visible object called a `Mesh` is the combination of two things: a **Geometry** (the shape) and a **Material** (how it looks).

Built-in geometries include `BoxGeometry`, `SphereGeometry`, `PlaneGeometry`, `CylinderGeometry`, and more.
Each takes dimension arguments - `BoxGeometry(1, 1, 1)` creates a 1x1x1 unit cube.

Two essential materials to know:

- **MeshBasicMaterial** - flat color, no lighting.
Useful for debugging or UI elements.
It ignores all lights in the scene.

- **MeshStandardMaterial** - physically-based (PBR) material that responds to lights.
This is what you use for realistic objects.
Set `color`, `roughness` (0 = mirror, 1 = chalk), and `metalness` (0 = plastic, 1 = metal).

### The Animation Loop

Three.js renders a still frame each time you call `renderer.render(scene, camera)`.
To animate, you call it repeatedly using `requestAnimationFrame`, the browser's built-in animation API.

```js
function animate() {
  requestAnimationFrame(animate);
  mesh.rotation.y += 0.01;
  renderer.render(scene, camera);
}
animate();
```

This pattern calls itself ~60 times per second, synchronized with the display refresh rate.
The mesh rotation updates each frame, creating smooth animation.

### OrbitControls and GLTFLoader

**OrbitControls** lets users orbit, pan, and zoom the camera using mouse or touch.
It is not included in the core Three.js build - import it from `three/addons/controls/OrbitControls.js`.

**GLTFLoader** loads `.glb` and `.gltf` files, the standard format for 3D models on the web.
It is also an addon: `three/addons/loaders/GLTFLoader.js`.
The loader is asynchronous - it takes a callback or returns a Promise.

## Key Terminology

| Term | Definition |
|------|-----------|
| WebGL | Browser's low-level 3D graphics API; Three.js builds on top of it |
| Scene | The root container holding all 3D objects, lights, and cameras |
| Mesh | A visible 3D object = Geometry (shape) + Material (appearance) |
| Geometry | Defines the shape of a mesh - its vertices and faces |
| Material | Defines how a surface looks - color, shininess, texture maps |
| PerspectiveCamera | Camera that mimics real-world perspective with a field of view |
| Renderer | Converts the 3D scene into 2D pixels on a canvas element |
| GLB | Binary GLTF format - the preferred format for web 3D models |
| OrbitControls | Mouse/touch addon that lets users orbit and zoom the camera |
| requestAnimationFrame | Browser API for running a function before each screen repaint |

## Code Examples

### Project Setup

```bash
npm create vite@latest my-3d-scene -- --template vanilla
cd my-3d-scene
npm install three
npm run dev
```

### Complete Spinning Cube Scene

Replace the contents of `src/main.js` with:

```js
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// 1. Scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a2e);

// 2. Camera
const camera = new THREE.PerspectiveCamera(
  75,                                      // field of view in degrees
  window.innerWidth / window.innerHeight,  // aspect ratio
  0.1,                                     // near clipping plane
  100                                      // far clipping plane
);
camera.position.set(0, 1, 3);

// 3. Renderer
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// Geometry + Material = Mesh
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshStandardMaterial({
  color: 0x0d9488,
  roughness: 0.4,
  metalness: 0.2,
});
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// Lighting (required for MeshStandardMaterial)
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1.5);
directionalLight.position.set(5, 8, 5);
scene.add(directionalLight);

// Interactive controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// Responsive resize
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// Animation loop
function animate() {
  requestAnimationFrame(animate);
  cube.rotation.x += 0.005;
  cube.rotation.y += 0.01;
  controls.update(); // needed when damping is on
  renderer.render(scene, camera);
}
animate();
```

### Loading a GLB Model

```js
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();
loader.load(
  '/model.glb',
  (gltf) => {
    scene.add(gltf.scene);
  },
  undefined, // progress callback (optional)
  (error) => console.error('Load error:', error)
);
```

## Hands-On Exercises

1. Run the spinning cube setup above.
Observe how removing the `AmbientLight` makes the cube completely black, confirming that `MeshStandardMaterial` requires light.

2. Add a second mesh to the scene - try a `SphereGeometry(0.5, 32, 32)` - and position it at `(2, 0, 0)`.
Experiment with different material colors.

3. Change `MeshStandardMaterial` to `MeshBasicMaterial` on the cube.
Notice the material no longer responds to lights and always shows its full color.

4. Download a free GLB from Sketchfab (sketchfab.com) and load it using `GLTFLoader`.
Call `gltf.scene.scale.setScalar(0.1)` if the model appears too large.

5. Add a `PlaneGeometry(10, 10)` as a ground plane.
Rotate it with `plane.rotation.x = -Math.PI / 2` to lay it flat.

## Resources

- [Three.js Official Docs](https://threejs.org/docs/) - API reference and live examples
- [Three.js Examples](https://threejs.org/examples/) - hundreds of working demos with source code
- [Three.js Journey](https://threejs-journey.com/) - Bruno Simon's comprehensive course, the gold standard for learning Three.js
- [GLTF Viewer](https://gltf-viewer.donmccurdy.com/) - inspect GLB files in the browser before loading them
- [Sketchfab Free Models](https://sketchfab.com/features/free-3d-models) - free GLB downloads for practice

## Reflection Questions

1. Why does `MeshStandardMaterial` require lights in the scene while `MeshBasicMaterial` does not?
2. What would happen if you called `renderer.render(scene, camera)` only once at startup, with no animation loop?
3. Why is `camera.updateProjectionMatrix()` needed after changing the aspect ratio on resize?
4. What is the difference between a Geometry and a Mesh in Three.js?
5. Why do we use the `three/addons/` path for OrbitControls instead of importing directly from `three`?

## Next Steps

Tomorrow you will rebuild this same scene using React Three Fiber - a declarative React renderer for Three.js that replaces imperative scene setup with JSX components and hooks.

---

[← Day 3: Text-to-3D AI](./day-03-text-to-3d.md) | [Day 5: React Three Fiber →](./day-05-react-three-fiber.md)
