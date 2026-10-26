# Day 21: Final Capstone: Your 3D Web Experience

## Learning Objectives

- Select and complete one of four structured 3D web projects based on your interests and skill level.
- Apply the full pipeline: AI model generation, optimization, and rendering in Three.js or R3F.
- Deploy a live, public-facing 3D web experience to GitHub Pages or Vercel.
- Reflect on everything covered in 21 days and identify clear next steps for continued learning.

## Core Concepts

### Choosing Your Project

Today is your day to build and ship.
Choose one of four project options below based on what excites you most.
Each option has a step-by-step guide and starter code.
They are all deployable in a 30-minute focused session.

**Option A** is for learners who want to combine everything into a real portfolio asset.
**Option B** is for learners interested in product design and interactivity.
**Option C** is for learners drawn to games and physics.
**Option D** is for learners who love visual art and shaders.

All four options use the tools and patterns you have practiced.
None require more than what you have already learned.

### Option A: 3D Portfolio Hero Section

Build an animated 3D model in a portfolio hero section.
The model loads, plays a welcome animation, then idles.
Add scroll-triggered zoom for a polished feel.

**Core tools**: Meshy.ai + Mixamo + R3F + Vercel

Steps:
1. Generate a character or mascot in Meshy.ai.
2. Export and optimize with gltf-transform.
3. Rig and animate in Mixamo (wave + idle).
4. Load in R3F with useAnimations.
5. Add a subtle auto-rotate and a clean background color.
6. Deploy to Vercel.

### Option B: Product Configurator

Build a 3D product viewer where the user can change colors and materials.
A minimal UI panel on the side drives the changes via Three.js material updates.

**Core tools**: Meshy.ai or a free model from Sketchfab + Three.js + GitHub Pages

Steps:
1. Find or generate a simple product model (shoe, bottle, headphones).
2. Load with GLTFLoader.
3. Traverse the scene to identify material names.
4. Wire HTML color inputs to material.color.set().
5. Add OrbitControls for inspection.
6. Deploy to GitHub Pages.

### Option C: Mini Browser Game

Build a simple 3D collectible game.
A player sphere moves with arrow keys.
Collect floating cubes for points.
Display a score counter in HTML.

**Core tools**: Three.js (no external physics needed for this one) + GitHub Pages

Steps:
1. Create a flat plane as the ground.
2. Add a sphere as the player - move with WASD/arrow keys.
3. Spawn 10 rotating collectibles at random positions.
4. On collision (distance check), remove the collectible and increment score.
5. Add a simple ambient + directional light.
6. Deploy to GitHub Pages.

### Option D: Interactive Art Piece

Build a fullscreen GLSL shader scene that responds to mouse movement.
The shader creates flowing color waves that follow the cursor.

**Core tools**: Three.js GLSL shaders (Day 17) + Vercel or GitHub Pages

Steps:
1. Create a fullscreen PlaneGeometry.
2. Write a vertex shader and a fragment shader.
3. Pass mouse position as a uniform.
4. Create a flowing noise-based color pattern with mouse influence.
5. Add a subtle time-based animation.
6. Deploy.

### Deploying in 5 Minutes

**GitHub Pages (simplest for static sites):**
No build step required for vanilla Three.js projects.
Push your files to a repo, go to Settings > Pages, set source to main branch.
Your site is live at `username.github.io/repo-name`.

**Vercel (best for React/Vite projects):**
Connect your GitHub repository at vercel.com.
Vercel auto-detects Vite and runs `npm run build`.
Your site gets a live URL in under 2 minutes.

## Key Terminology

| Term | Definition |
|------|-----------|
| Capstone | A final project that applies all skills learned in a course |
| Deploy | Publish a project to a live hosting server accessible via URL |
| GitHub Pages | Free static hosting from GitHub; ideal for portfolios and demos |
| Vercel | Cloud platform for frontend deployments; auto-deploys from Git commits |
| OrbitControls | Three.js helper that adds mouse-controlled camera rotation/zoom |
| Traversal | Iterating through every child in a Three.js scene graph to find or modify objects |
| Collectible | In games, an object the player can pick up to earn points or power-ups |
| Uniform | A variable passed from JavaScript to a GLSL shader, updated each frame |

## Code Examples

### Option A: R3F Animated Character Starter

```javascript
// src/App.jsx
import { Canvas } from '@react-three/fiber';
import { useGLTF, useAnimations, OrbitControls, Environment } from '@react-three/drei';
import { useEffect } from 'react';

useGLTF.preload('/model.glb');

function Character() {
  const { scene, animations } = useGLTF('/model.glb');
  const { actions, names } = useAnimations(animations, scene);

  useEffect(() => {
    // Play the first available animation
    if (names[0]) actions[names[0]]?.reset().play();
  }, [actions, names]);

  return <primitive object={scene} position={[0, -1.5, 0]} />;
}

export default function App() {
  return (
    <div style={{ width: '100vw', height: '100vh', background: '#0a0a0a' }}>
      <Canvas camera={{ position: [0, 0, 4], fov: 50 }}>
        <ambientLight intensity={0.4} />
        <directionalLight position={[3, 5, 3]} intensity={1.5} />
        <Character />
        <OrbitControls enableZoom={false} autoRotate autoRotateSpeed={1} />
        <Environment preset="city" />
      </Canvas>
    </div>
  );
}
```

### Option B: Product Configurator Starter (Three.js)

```javascript
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(50, innerWidth / innerHeight, 0.1, 100);
camera.position.set(0, 1, 3);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(devicePixelRatio);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 0.5));
scene.add(Object.assign(new THREE.DirectionalLight(0xffffff, 1), { position: new THREE.Vector3(3, 5, 3) }));

const controls = new OrbitControls(camera, renderer.domElement);

const loader = new GLTFLoader();
let productMesh = null;

loader.load('/product.glb', (gltf) => {
  scene.add(gltf.scene);
  // Find the first mesh with a material
  gltf.scene.traverse((child) => {
    if (child.isMesh) productMesh = child;
  });
});

// Wire to color picker input in HTML: <input type="color" id="colorPicker" value="#ffffff">
document.getElementById('colorPicker').addEventListener('input', (e) => {
  if (productMesh) productMesh.material.color.set(e.target.value);
});

(function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
})();
```

### Option C: Mini Game Starter (Three.js)

```javascript
import * as THREE from 'three';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a2e);
const camera = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 100);
camera.position.set(0, 8, 10);
camera.lookAt(0, 0, 0);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 0.6));
scene.add(Object.assign(new THREE.DirectionalLight(0xffffff, 1), { position: new THREE.Vector3(5, 10, 5) }));

// Ground
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(20, 20),
  new THREE.MeshStandardMaterial({ color: 0x2a2a4a })
);
ground.rotation.x = -Math.PI / 2;
scene.add(ground);

// Player
const player = new THREE.Mesh(
  new THREE.SphereGeometry(0.5, 16, 16),
  new THREE.MeshStandardMaterial({ color: 0x0d9488 })
);
player.position.y = 0.5;
scene.add(player);

// Collectibles
const collectibles = [];
for (let i = 0; i < 10; i++) {
  const mesh = new THREE.Mesh(
    new THREE.BoxGeometry(0.4, 0.4, 0.4),
    new THREE.MeshStandardMaterial({ color: 0xf4d35e })
  );
  mesh.position.set((Math.random() - 0.5) * 16, 0.2, (Math.random() - 0.5) * 16);
  scene.add(mesh);
  collectibles.push(mesh);
}

const keys = {};
document.addEventListener('keydown', (e) => { keys[e.key] = true; });
document.addEventListener('keyup',   (e) => { keys[e.key] = false; });

let score = 0;
const scoreEl = document.getElementById('score');

const clock = new THREE.Clock();
(function animate() {
  requestAnimationFrame(animate);
  const dt = clock.getDelta();
  const speed = 5;

  if (keys['ArrowLeft']  || keys['a']) player.position.x -= speed * dt;
  if (keys['ArrowRight'] || keys['d']) player.position.x += speed * dt;
  if (keys['ArrowUp']    || keys['w']) player.position.z -= speed * dt;
  if (keys['ArrowDown']  || keys['s']) player.position.z += speed * dt;

  collectibles.forEach((c, i) => {
    c.rotation.y += dt;
    if (c.visible && player.position.distanceTo(c.position) < 0.8) {
      c.visible = false;
      score++;
      if (scoreEl) scoreEl.textContent = 'Score: ' + score;
    }
  });

  renderer.render(scene, camera);
})();
```

### Option D: Interactive Shader Art Starter (Three.js)

```javascript
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 1);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

const uniforms = {
  uTime:  { value: 0 },
  uMouse: { value: new THREE.Vector2(0.5, 0.5) },
  uRes:   { value: new THREE.Vector2(innerWidth, innerHeight) }
};

const material = new THREE.ShaderMaterial({
  uniforms,
  vertexShader: `
    varying vec2 vUv;
    void main() { vUv = uv; gl_Position = vec4(position, 1.0); }
  `,
  fragmentShader: `
    uniform float uTime;
    uniform vec2  uMouse;
    varying vec2  vUv;

    float wave(vec2 p, vec2 center, float freq) {
      float d = distance(p, center);
      return sin(d * freq - uTime * 2.0) * 0.5 + 0.5;
    }

    void main() {
      float w = wave(vUv, uMouse, 12.0);
      vec3 colorA = vec3(0.05, 0.58, 0.53);
      vec3 colorB = vec3(0.06, 0.06, 0.12);
      vec3 color = mix(colorB, colorA, w);
      gl_FragColor = vec4(color, 1.0);
    }
  `
});

scene.add(new THREE.Mesh(new THREE.PlaneGeometry(2, 2), material));

document.addEventListener('mousemove', (e) => {
  uniforms.uMouse.value.set(e.clientX / innerWidth, 1 - e.clientY / innerHeight);
});

const clock = new THREE.Clock();
(function animate() {
  requestAnimationFrame(animate);
  uniforms.uTime.value = clock.getElapsedTime();
  renderer.render(scene, camera);
})();
```

## Hands-On Exercises

1. Choose your project option and set a 30-minute timer.
   Build the starter code locally and verify it runs in the browser before adding your own changes.

2. Customize your project - change at least two things from the starter code.
   For Option A, swap in your own model.
   For Option B, add a second configurable property (roughness or metalness).
   For Option C, add a win condition (collect all 10) and a reset button.
   For Option D, add a second wave center that responds to click events.

3. Run gltf-transform on your model (if using Options A or B) before deploying.
   Record the before and after file sizes.

4. Deploy your project using GitHub Pages or Vercel.
   Share the live URL with someone.

5. Write 3 sentences describing what you built, one challenge you solved, and what you would add next.

## Resources

- [Three.js Journey](https://threejs-journey.com/) - the best comprehensive Three.js course by Bruno Simon; covers everything from basics to advanced shaders
- [React Three Fiber documentation](https://r3f.docs.pmnd.rs/) - official docs, especially the @react-three/drei helpers
- [Shadertoy](https://www.shadertoy.com/) - community gallery of GLSL shaders; incredible learning resource and inspiration for Option D
- [Blender Guru on YouTube](https://www.blenderartists.org/) - Andrew Price's tutorials are the standard starting point for Blender; the Blender Artists forum is a friendly community
- [Sketchfab](https://sketchfab.com/) - thousands of free 3D models with download in GLB format; great for extending what AI tools generate

## Reflection Questions

1. Which of the six pipeline stages felt most natural to you?
   Which felt hardest?

2. AI tools did a lot of heavy lifting in this course.
   What did you still need human judgment and taste to decide?

3. What would you build next if you had one more week?
   Which tools from this course would you use?

4. Deploying to a live URL changed how the project felt.
   Why does having a public URL matter for creative and professional work?

5. What is one thing from this course that surprised you - a capability, a workflow, or a constraint you did not expect?

## Next Steps

### Congratulations - You Finished the Course!

You have covered Three.js, React Three Fiber, Spline, Meshy.ai, Tripo3D, Mixamo, GLSL shaders, performance optimization, the Blender export pipeline, and a full AI-assisted workflow.
In 21 days at 30 minutes per day, you went from zero 3D experience to shipping live, interactive 3D experiences on the web.
That is a real skill set.

**Where to go from here:**

- **Deepen Three.js**: Bruno Simon's [Three.js Journey](https://threejs-journey.com/) is the gold standard.
  It picks up exactly where this course leaves off.

- **Master React Three Fiber**: The [R3F documentation](https://r3f.docs.pmnd.rs/) and the @react-three/drei helper library have dozens of patterns you have not touched yet.

- **Level up your shaders**: Spend time on [Shadertoy](https://www.shadertoy.com/).
  Read other people's shaders.
  Try to recreate one effect per week.

- **Improve your Blender skills**: [Blender Guru](https://www.blenderguru.com/) has excellent modeling tutorials.
  Even moderate Blender skill unlocks a lot when combined with AI generation.

- **Build something real**: The best next step is a project that matters to you - a portfolio, a game, a tool for your work.
  Real constraints teach faster than tutorials.

You now have the foundation.
Everything else is practice, iteration, and curiosity.

[Back to Day 20: Full AI-Assisted Workflow](./day-20-full-ai-workflow.md)
