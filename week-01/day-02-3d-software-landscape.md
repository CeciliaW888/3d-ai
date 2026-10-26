# Day 2: The 3D Software Landscape

## Learning Objectives

- Map the major 3D creation, rendering, and delivery tools to their roles in the pipeline
- Explain what Three.js is and why it exists on top of raw WebGL
- Describe the relationship between Three.js and React Three Fiber
- Explain why GLB is the standard format for web 3D asset delivery
- Choose the appropriate tool for a given web 3D use case

## Core Concepts

### The Web 3D Ecosystem at a Glance

The 3D world for the web can feel overwhelming at first - dozens of tools, all with their own names and specialties.
The good news: for web 3D, the ecosystem has converged around a small set of tools that work well together.
Understanding where each one sits in the pipeline prevents costly tool-mismatch mistakes.

Think of the pipeline in three layers:

**Creation** - where you build or generate 3D assets:
Blender (manual modeling), Meshy.ai and Tripo3D (AI generation), Spline (visual web 3D)

**Delivery Format** - the lingua franca of web 3D:
GLTF and GLB

**Rendering in the Browser** - what makes it appear on screen:
Three.js, React Three Fiber

**Asset Libraries** - where you find ready-made models:
Sketchfab, Poly Pizza, Kenney.nl

### Blender: The Free 3D Powerhouse

Blender is the open-source 3D application that does everything: modeling, sculpting, rigging, animation, simulation, rendering, compositing, and video editing.
It is completely free, released under the GNU GPL, and is used by everyone from solo indie developers to large studios.

Blender's interface takes time to learn - it has a dense keyboard-shortcut-driven workflow.
For web 3D purposes, you need to be comfortable enough to:
- Open a model and inspect it for problems
- Adjust or fix materials and UV maps
- Use the remesh tools to reduce polygon count
- Export to GLB format

You do not need to become a Blender expert.
AI-generated models often need a cleanup pass before they are web-ready, and Blender is where that happens.

### Three.js: WebGL Without the Pain

Raw WebGL - the browser's low-level GPU API - requires hundreds of lines of boilerplate to render even a single triangle.
You have to manually write vertex shaders, fragment shaders, set up buffer objects, bind textures, and manage draw calls.

Three.js is a JavaScript library that wraps all of that into a clean, object-oriented scene graph API.
You work with familiar objects: `Scene`, `Camera`, `Light`, `Mesh`, `Geometry`, `Material`.
The library handles all the GPU setup, shader compilation, and draw call management behind the scenes.

Three.js is the foundation under almost all serious web 3D.
Even if you primarily use higher-level tools like React Three Fiber, learning Three.js concepts makes you a stronger developer because you understand exactly what each layer is doing.

### React Three Fiber: Three.js as React Components

React Three Fiber (R3F) is a React renderer for Three.js.
Instead of writing imperative code like `scene.add(mesh)`, you write declarative JSX like `<mesh>`.
R3F lets you use React's full component model - state, effects, context, refs - to drive your 3D scene.

This is the right choice when:
- You are building a React or Next.js application
- You want to mix 3D elements with standard React UI components
- Your scene's state is driven by application data

The companion library **Drei** (German for "three") provides a large collection of pre-built R3F components for common patterns: orbit controls, environment maps, model loaders, HTML overlays, postprocessing effects, and much more.
Drei dramatically reduces the boilerplate of common 3D tasks.

### Spline: No-Code 3D for the Web

Spline is a browser-based 3D design tool built specifically for creating interactive scenes that embed directly in websites.
You build your scene visually in the Spline editor, then drop it into your site as an iframe or a React component.

Spline is ideal when:
- You need a polished 3D hero section without writing Three.js code
- A designer (rather than a developer) is building the 3D content
- You want built-in scroll animations, hover effects, and physics

Its limitation is flexibility.
Complex data-driven 3D or programmatically generated content is harder to achieve in Spline than in Three.js or R3F.

### Meshy.ai and Tripo3D: AI-Generated 3D Assets

These tools generate textured 3D models from text prompts or reference images in under two minutes.
They live in the "Creation" layer of the pipeline and produce standard GLB output that drops directly into Three.js, R3F, or any other 3D renderer.

Day 3 covers these tools in depth.
The key point for today: they are asset creation tools, not rendering tools.
You generate a model here and then load it into Three.js or R3F to display it.

### Sketchfab: The YouTube of 3D

Sketchfab is the largest online community and marketplace for 3D models.
It has a huge library of free CC-licensed models, an in-browser 3D viewer, and an API for embedding models directly in your site.
When you need a ready-made asset and do not want to generate or model it from scratch, Sketchfab is your first stop.

### GLTF/GLB: The "JPEG of 3D"

GLTF (GL Transmission Format) was designed by the Khronos Group specifically for efficient real-time delivery of 3D assets to WebGL renderers.
Think of it as what JPEG is for images, but for 3D - a standard, compact format understood by everything in the ecosystem.

**GLTF** stores data as a JSON file with references to external .bin geometry files and image textures.
**GLB** is the binary-packed version: everything in a single file.

For web delivery, always use **GLB**.
One file, faster loading, and natively supported by Three.js's `GLTFLoader`.

A GLB file can contain:
- Mesh geometry with normals, UV coordinates, and vertex data
- PBR material properties (albedo, roughness, metalness) and all texture maps
- Skeletal animations and morph target animations
- Scene hierarchy with object positions, rotations, and scales

Every tool in this course - Blender, Meshy.ai, Tripo3D, Spline - exports to GLB.
It is the single format that connects them all.

## Key Terminology

| Term | Definition |
|------|-----------|
| Three.js | A JavaScript library wrapping WebGL into a high-level scene graph API |
| React Three Fiber (R3F) | A React renderer for Three.js - write Three.js scenes as declarative JSX |
| Drei | R3F's companion utility library with pre-built components and helpers |
| WebGL | The browser-level GPU API that Three.js sits on top of |
| Blender | Free, open-source 3D suite for modeling, rigging, animation, and rendering |
| Spline | Browser-based no-code tool for building interactive 3D web scenes |
| GLTF | GL Transmission Format - the Khronos standard for web 3D asset delivery |
| GLB | Binary-packed single-file version of GLTF - preferred for web delivery |
| Sketchfab | Online platform and marketplace for discovering and sharing 3D models |
| Scene Graph | A hierarchical tree data structure representing all objects and their relationships in a scene |

## Code Examples

```javascript
// Three.js: Load a GLB model from a URL
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();

loader.load(
  '/models/my-model.glb',
  (gltf) => {
    // gltf.scene is a THREE.Group containing the full loaded model
    scene.add(gltf.scene);

    // If the GLB contains animations, play the first one
    if (gltf.animations.length > 0) {
      const mixer = new THREE.AnimationMixer(gltf.scene);
      mixer.clipAction(gltf.animations[0]).play();
    }
  },
  (progress) => {
    const pct = (progress.loaded / progress.total * 100).toFixed(0);
    console.log(`Loading: ${pct}%`);
  },
  (error) => console.error('Load failed:', error)
);
```

```jsx
// React Three Fiber: Load the same GLB using Drei's useGLTF hook
// Drei handles loading, caching, and gives you a React-compatible result
import { Canvas } from '@react-three/fiber';
import { useGLTF, OrbitControls, Environment } from '@react-three/drei';

function Model({ url }) {
  const { scene } = useGLTF(url);
  return <primitive object={scene} />;
}

export default function App() {
  return (
    <Canvas camera={{ position: [0, 1, 3] }}>
      <ambientLight intensity={0.5} />
      <directionalLight position={[5, 10, 5]} intensity={1.2} />
      <Model url="/models/my-model.glb" />
      <OrbitControls />
      <Environment preset="sunset" />
    </Canvas>
  );
}
```

Notice how the R3F version is more concise for a React context, while the Three.js version gives you explicit control over the loading lifecycle.
Both approaches load the same GLB file.

## Hands-On Exercises

1. **Map the ecosystem**: Draw (or sketch in a notes app) the three-layer pipeline - Creation, Format, Rendering.
   Place each tool from today's lesson in its correct layer.
2. **Explore Three.js examples**: Browse [threejs.org/examples](https://threejs.org/examples) and open the source code of one example.
   Identify the geometry type, material type, and light types being used.
3. **Try Spline**: Go to [spline.design](https://spline.design), create a free account, and open one of the template scenes.
   Export it as an iframe embed code and paste it into a plain HTML file.
4. **Download a Sketchfab model**: Find a free model on [sketchfab.com](https://sketchfab.com) (filter by "Downloadable"), download it as GLB, and note the file size.
5. **Bookmark the docs**: Bookmark the Three.js docs, the R3F docs, and the Drei docs.
   Spend five minutes scanning each to get a feel for the scope of each library.

## Resources

- [Three.js Documentation](https://threejs.org/docs/)
- [React Three Fiber Documentation](https://docs.pmnd.rs/react-three-fiber)
- [Drei Documentation](https://drei.docs.pmnd.rs)
- [Spline Design](https://spline.design)
- [Sketchfab Free Models](https://sketchfab.com/features/free-3d-models)

## Reflection Questions

1. If you are building a Next.js marketing site that needs a spinning product model, which rendering tool would you choose and why?
2. Describe the relationship between WebGL, Three.js, and React Three Fiber as a layered stack - what does each layer add?
3. Why is GLB preferred over OBJ for web 3D projects?
4. What is the main tradeoff between using Spline versus Three.js for a web project?
5. When would you use a model from Sketchfab versus generating one with Meshy.ai?

## Next Steps

Tomorrow you go hands-on with AI-powered 3D generation - you will use Meshy.ai and Tripo3D to generate your first 3D models from text prompts, apply prompt engineering techniques, and export them as GLB files ready for the web.

[← How 3D Works](./day-01-how-3d-works.md) | [Text-to-3D AI →](./day-03-text-to-3d-ai.md)
