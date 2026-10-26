# Day 3: Text-to-3D AI - Meshy & Tripo3D

## Learning Objectives

- Explain how diffusion-based text-to-3D models generate geometry and textures at a high level
- Generate a 3D model using Meshy.ai from a text prompt
- Apply prompt engineering techniques specific to 3D asset generation
- Compare Meshy.ai and Tripo3D for different asset types
- Export an AI-generated model as GLB and load it in a basic Three.js scene

## Core Concepts

### How Text-to-3D AI Actually Works

Text-to-3D tools do not think in terms of triangles and vertices.
They think in terms of images and neural representations.
Understanding this helps you predict what they excel at - and where they will struggle.

Modern text-to-3D tools like Meshy and Tripo3D run a two-stage pipeline:

**Stage 1 - Multi-view image synthesis.**
A diffusion model generates a set of images showing the object from multiple fixed angles simultaneously - typically front, back, left, right, and top.
This model was trained specifically to produce geometrically consistent views: if the front view shows a handle, the side view will account for it in the same position.
This is the same diffusion technology behind image generators like Stable Diffusion, but fine-tuned for multi-view consistency.

**Stage 2 - 3D reconstruction.**
The multi-view images are fed into a 3D reconstruction network that builds a textured mesh from them.
This is conceptually similar to photogrammetry (building 3D models from photographs), but done entirely inside a neural network in seconds rather than hours.

The output is a textured GLB mesh ready for Three.js, React Three Fiber, or any 3D viewer.
Quality has improved dramatically through 2024-2025, but current models still struggle with fine details (fingers, text, intricate mechanical parts), transparent materials, and objects that look very different from every angle.

### Meshy.ai Walkthrough

Meshy ([meshy.ai](https://meshy.ai)) is one of the most capable text-to-3D tools available today.
It has a generous free tier with monthly credits and a clean browser-based interface.

The standard workflow:

1. Create a free account and log in to the dashboard
2. Choose **Text to 3D** from the sidebar
3. Enter your prompt - be specific (more on this below)
4. Choose art style: **Realistic**, **Cartoon**, or **Low Poly**
5. Choose quality: **Draft** (about 30 seconds) for iteration, **Refine** (2-3 minutes) for final output
6. Wait for generation and preview the result in the browser viewer
7. Use the polygon reduction slider to bring the mesh to a web-appropriate count
8. Click **Download** and select **GLB** format

Meshy also offers **Image to 3D** (use a reference photo as the starting point) and **Text to Texture** (apply AI-generated textures to an existing mesh you upload).

### Tripo3D Walkthrough

Tripo3D ([tripo3d.ai](https://tripo3d.ai)) is Meshy's closest peer.
The interface is similar: enter a prompt, choose a style, wait, download.

Tripo3D tends to produce:
- Lower polygon counts by default (better for web performance without extra remeshing)
- Cleaner edge flow on hard-surface and architectural objects
- Slightly less surface texture detail compared to Meshy

For any important asset, generate it in both tools and pick the better result.
With sub-2-minute generation times, this costs almost no extra time.

### Luma AI Genie

Luma AI's Genie ([lumalabs.ai/genie](https://lumalabs.ai/genie)) is a third option particularly strong for organic shapes, creatures, and objects with complex surface texture like wood grain, woven fabric, or rock.
The free tier is rate-limited but worth experimenting with.

### Prompt Engineering for 3D

The same intuitions you have for image generation prompts apply here, with some 3D-specific additions.

**Be specific about art style:**
- "low-poly wooden treasure chest, game asset style" vs "photorealistic oak chest with iron bands"
- "cartoon character, simplified geometry, vibrant colors" vs "hyper-realistic human with detailed pores"

**Describe the object, not a scene:**
- Good: "ceramic coffee mug, white glaze with a blue stripe, simple handle"
- Avoid: "a cozy coffee shop corner with a mug on the counter" - too much scene context confuses the model

**Specify presentation and isolation:**
- "single object, centered, no background, no shadow plane"
- "isolated prop, front-facing, game-ready asset"

**Use style references:**
- "Studio Ghibli inspired organic shapes"
- "hard-surface sci-fi aesthetic, machined metal"
- "Pixar character style, expressive proportions"

**What to avoid:**
- Complex articulated poses (a running character is much harder than a T-pose)
- Transparent or glass-like materials - current models render these poorly
- Very small text or lettering
- Multiple separate objects in one prompt

### Output Formats for Web

| Format | Description | When to use |
|--------|-------------|-------------|
| GLB | Binary GLTF - mesh, materials, textures in one file | Web projects - Three.js, R3F, Spline |
| OBJ + MTL | Classic format with separate texture files | Importing into Blender for cleanup |
| FBX | Autodesk format, strong animation support | Game engines, Mixamo rigging pipeline |

Always download **GLB** for web work.
One file, faster loading, and natively supported by `GLTFLoader`.

### Polygon Count and Retopology

AI-generated meshes frequently have messy topology: uneven triangle density, floating geometry, artifacts around complex features, and more polygons than any web renderer needs.

**Retopology** is the process of rebuilding a cleaner mesh over an existing one.
For web 3D, the practical question is how many triangles your target device can handle:

- Background decorative prop: under 2,000 triangles
- Hero product model in a presentation: 5,000-10,000 triangles
- Interactive game character: 5,000-15,000 triangles

Meshy and Tripo3D both include a polygon reduction slider before export.
Always use it.
If a model still has topology problems after remeshing, open it in Blender for cleanup.
This is the "Creation -> Quality Check" step before the model reaches your web renderer.

## Key Terminology

| Term | Definition |
|------|-----------|
| Diffusion Model | An AI model trained to generate data by learning to reverse a noise-addition process |
| Multi-view Synthesis | Generating geometrically consistent images of an object from multiple angles at once |
| 3D Reconstruction | Converting a set of 2D image observations into a 3D mesh and texture maps |
| NeRF | Neural Radiance Field - a neural function that implicitly represents a 3D scene; used in research and earlier generation tools |
| Retopology | Rebuilding a mesh with cleaner, more efficient edge flow and polygon distribution |
| Remeshing | Automatically redistributing a mesh's polygons to hit a target count |
| Polygon Count | Total number of triangles in a mesh - directly affects rendering performance |
| Photogrammetry | Technique for building 3D models from multiple photographs of a real physical object |

## Code Examples

```javascript
// Loading an AI-generated GLB with orbit controls and auto-centering
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a2e);

const camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 100);
camera.position.set(0, 1.5, 3);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// Orbit controls let the user spin and inspect the model
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// Lighting setup that works well with PBR materials from AI generators
scene.add(new THREE.AmbientLight(0xffffff, 0.5));
const keyLight = new THREE.DirectionalLight(0xffffff, 1.2);
keyLight.position.set(5, 10, 5);
scene.add(keyLight);

// Load the AI-generated GLB
const loader = new GLTFLoader();
loader.load('/models/ai-generated.glb', (gltf) => {
  const model = gltf.scene;

  // Auto-center: AI models often have off-center origins
  const box = new THREE.Box3().setFromObject(model);
  const center = box.getCenter(new THREE.Vector3());
  model.position.sub(center);

  // Auto-scale: fit the model inside a 2-unit bounding box
  const size = box.getSize(new THREE.Vector3()).length();
  model.scale.setScalar(2 / size);

  scene.add(model);
});

function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}
animate();
```

```javascript
// Meshy API: Programmatic text-to-3D generation (requires a paid API plan)
// Useful for building automated asset generation pipelines

async function generateMeshyModel(prompt, apiKey) {
  // Step 1: Submit a generation task
  const createRes = await fetch('https://api.meshy.ai/v2/text-to-3d', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      mode: 'preview',          // 'preview' is fast; 'refine' is higher quality
      prompt: prompt,
      art_style: 'realistic',   // 'realistic', 'cartoon', or 'low-poly'
      negative_prompt: 'low quality, blurry, distorted geometry',
    }),
  });
  const { result: taskId } = await createRes.json();

  // Step 2: Poll until the task completes
  while (true) {
    await new Promise(r => setTimeout(r, 5000)); // Wait 5 seconds between polls

    const statusRes = await fetch(`https://api.meshy.ai/v2/text-to-3d/${taskId}`, {
      headers: { 'Authorization': `Bearer ${apiKey}` },
    });
    const task = await statusRes.json();

    if (task.status === 'SUCCEEDED') {
      // task.model_urls.glb is the download URL for the GLB file
      return task.model_urls.glb;
    }
    if (task.status === 'FAILED') {
      throw new Error('Generation failed: ' + task.message);
    }
    console.log(`Progress: ${task.progress}%`);
  }
}
```

## Hands-On Exercises

1. **Your first AI model**: Go to [meshy.ai](https://meshy.ai), create a free account, and generate a simple object in Draft mode.
   Start with something clear and bounded like "ceramic coffee mug, white, photorealistic" or "wooden treasure chest, game asset."
2. **Prompt comparison**: Generate the same object concept (such as "wooden chair") with three different style settings: Realistic, Cartoon, and Low Poly.
   Compare the results side by side and note which style generates most consistently.
3. **Cross-tool comparison**: Take your best prompt and generate it in both Meshy.ai and Tripo3D.
   Compare mesh density, texture quality, and any artifacts in the preview viewer.
4. **Download and load**: Download your best model as GLB.
   Set up a minimal Three.js scene using the code example above and load the model.
   Verify it centers and scales correctly in the browser.
5. **Prompt iteration log**: Take a model with obvious quality issues.
   Iterate on the prompt at least three times and document what changes to the prompt improved or degraded the result.

## Resources

- [Meshy.ai](https://meshy.ai) - primary text-to-3D tool with free tier
- [Tripo3D](https://tripo3d.ai) - alternative generator, strong on hard-surface topology
- [Luma AI Genie](https://lumalabs.ai/genie) - good for organic shapes and creatures
- [Three.js GLTFLoader Documentation](https://threejs.org/docs/#examples/en/loaders/GLTFLoader)
- [Khronos GLTF Specification](https://www.khronos.org/gltf/)

## Reflection Questions

1. Why do text-to-3D models generate multiple 2D views first instead of directly outputting vertex data?
2. What types of objects tend to generate well with current AI tools, and what types consistently fail?
3. If a client needs a specific real-world branded product in a 3D visualization, why might AI generation be insufficient?
4. What is retopology and when should you do it on an AI-generated model?
5. How would you decide between using a model from Sketchfab versus generating one with Meshy.ai for a client project?

## Next Steps

Tomorrow you will write your first complete Three.js scene from scratch - setting up the renderer, camera, and render loop, adding geometries and materials, and building interactivity.

[← The 3D Software Landscape](./day-02-3d-software-landscape.md) | [Three.js Basics →](./day-04-threejs-basics.md)
