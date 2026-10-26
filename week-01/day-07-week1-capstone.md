# Day 7: Week 1 Capstone - AI 3D Hero Section

## Learning Objectives
- Combine an AI-generated GLB model with Three.js in a real landing page hero section.
- Use GLTFLoader to load an external 3D model into a Three.js scene.
- Apply responsive CSS so the canvas scales correctly on mobile and desktop.
- Add a smooth floating or rotation animation using requestAnimationFrame.
- Handle the loading state and add a mobile fallback gracefully.

## Core Concepts

### Project Overview

This capstone ties together everything from Week 1.
You will build a complete, production-ready hero section - the first thing a visitor sees when they land on a page.
The 3D object at the center comes from an AI generator such as Meshy.ai, and you will drop it into a Three.js canvas embedded in HTML.

### Architecture Decisions

**Three.js via CDN vs React Three Fiber**
For a standalone HTML file, Three.js via CDN is the simplest path.
You import Three.js from a CDN, add a `<canvas>` element, and write a small scene setup script.
React Three Fiber would require a build step, so it is not the right choice for a quick, self-contained hero file.

**Spline as an alternative**
If you generated your model inside Spline and exported it as a URL embed, you could use a `<script>` tag plus the `<spline-viewer>` web component instead of writing any Three.js.
That approach trades control for convenience.
Today we use GLTFLoader so you learn the fundamentals.

**GLB vs GLTF**
GLB is a single binary bundle that includes geometry, materials, and textures.
GLTF is the text-based version that references separate texture files.
Always use GLB for web delivery - one HTTP request is better than many.

### The Workflow

1. **Generate the model** - Use Meshy.ai's text-to-3D or image-to-3D workflow.
Choose a simple subject: a gem, a crystal, a piece of hardware, a product.
Simple shapes look great with Three.js lighting and require less polygon cleanup.

2. **Download the GLB** - Meshy exports GLB directly.
Place it next to your HTML file or host it on a CDN.

3. **Set up the scene** - Create a `WebGLRenderer`, a `PerspectiveCamera`, a `Scene`, and ambient + directional lights.
Two lights are enough: an `AmbientLight` for overall brightness and a `DirectionalLight` to create shading.

4. **Load the model** - Use `THREE.GLTFLoader` (from the three/examples/jsm addons) to load the GLB.
Center and scale the model using a bounding box helper.

5. **Add animation** - A gentle Y-axis rotation (about 0.003 radians per frame) and a sine-wave float on the Y position create a lively but not distracting motion.

6. **Responsive canvas** - Listen for `window.resize` and update both the renderer size and the camera aspect ratio.

### Performance Tips

- Set `antialias: true` only if the device pixel ratio is 1.
Higher DPR screens already have enough subpixel smoothing.
- Cap the renderer pixel ratio: `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))`.
- Keep your GLB under 3 MB for fast first load.
Run it through gltf-transform or gltf.report if Meshy's export is too large.
- On mobile, consider reducing canvas height or showing a static screenshot as a `<noscript>` fallback.

### Loading State

The GLB fetch takes time on slow connections.
Show a CSS spinner over the canvas while the model loads, then hide it in the GLTFLoader `onLoad` callback.
This prevents a jarring empty canvas.

## Key Terminology

| Term | Definition |
|------|-----------|
| GLB | Binary GLTF file format - a single file containing mesh, materials, and textures |
| GLTFLoader | Three.js utility class that parses GLB/GLTF files into Three.js objects |
| WebGLRenderer | Three.js class that renders a scene to a WebGL canvas element |
| requestAnimationFrame | Browser API that calls a function before each repaint - used for animation loops |
| PerspectiveCamera | Three.js camera that simulates realistic perspective depth |
| AmbientLight | Omnidirectional light with no position - provides a base brightness with no shadows |
| Bounding Box | Axis-aligned box that fits tightly around a 3D object - used to center and scale models |
| OrbitControls | Three.js addon that lets users rotate and zoom the camera by dragging the canvas |

## Code Examples

### Complete Hero Section HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI 3D Hero</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: system-ui, sans-serif;
      background: #0a0e1a;
      color: #e0e8ff;
      min-height: 100vh;
    }

    .hero {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      padding: 40px 20px;
      text-align: center;
      position: relative;
    }

    .hero-canvas-wrap {
      position: relative;
      width: min(480px, 90vw);
      height: min(480px, 90vw);
      margin-bottom: 32px;
    }

    #hero-canvas {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      display: block;
    }

    .loader {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .spinner {
      width: 40px;
      height: 40px;
      border: 3px solid rgba(255,255,255,0.15);
      border-top-color: #6ee7f7;
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
    }

    @keyframes spin {
      to { transform: rotate(360deg); }
    }

    h1 {
      font-size: clamp(2rem, 6vw, 3.5rem);
      font-weight: 800;
      letter-spacing: -0.02em;
      line-height: 1.1;
      margin-bottom: 16px;
    }

    .subtitle {
      font-size: clamp(1rem, 2.5vw, 1.25rem);
      color: rgba(224,232,255,0.7);
      max-width: 520px;
      line-height: 1.6;
      margin-bottom: 32px;
    }

    .cta {
      background: linear-gradient(135deg, #6ee7f7, #7c6fff);
      color: #0a0e1a;
      border: none;
      border-radius: 50px;
      padding: 14px 36px;
      font-size: 1rem;
      font-weight: 700;
      cursor: pointer;
      transition: opacity 0.2s;
    }

    .cta:hover { opacity: 0.85; }
  </style>
</head>
<body>
  <section class="hero">
    <div class="hero-canvas-wrap">
      <canvas id="hero-canvas"></canvas>
      <div class="loader" id="loader">
        <div class="spinner"></div>
      </div>
    </div>

    <h1>Your Product.<br>In Three Dimensions.</h1>
    <p class="subtitle">
      An AI-generated 3D experience that makes visitors stop scrolling.
    </p>
    <button class="cta">Get Started</button>
  </section>

  <!-- Three.js via CDN (r160 is stable) -->
  <script type="importmap">
    {
      "imports": {
        "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
        "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
      }
    }
  </script>

  <script type="module">
    import * as THREE from 'three';
    import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    // --- Scene setup ---
    const canvas  = document.getElementById('hero-canvas');
    const loader  = document.getElementById('loader');
    const wrap    = canvas.parentElement;

    const renderer = new THREE.WebGLRenderer({
      canvas,
      antialias: true,
      alpha: true
    });
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.setSize(wrap.clientWidth, wrap.clientHeight);
    renderer.outputColorSpace = THREE.SRGBColorSpace;

    const scene  = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(45, 1, 0.1, 100);
    camera.position.set(0, 0, 5);

    // Lighting
    scene.add(new THREE.AmbientLight(0xffffff, 0.6));
    const dirLight = new THREE.DirectionalLight(0x6ee7f7, 2);
    dirLight.position.set(3, 5, 3);
    scene.add(dirLight);
    const fillLight = new THREE.DirectionalLight(0x7c6fff, 0.8);
    fillLight.position.set(-3, -2, -3);
    scene.add(fillLight);

    // Optional: let the user spin the model
    const controls = new OrbitControls(camera, canvas);
    controls.enableZoom  = false;
    controls.enablePan   = false;
    controls.autoRotate  = false;
    controls.dampingFactor = 0.08;
    controls.enableDamping = true;

    // --- Load GLB ---
    // Replace this URL with your own GLB path or CDN URL
    const MODEL_URL = 'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/main/2.0/ToyCar/glTF-Binary/ToyCar.glb';

    let model = null;
    let clock = new THREE.Clock();

    const gltfLoader = new GLTFLoader();
    gltfLoader.load(
      MODEL_URL,
      function onLoad(gltf) {
        model = gltf.scene;

        // Center and scale model to fit in a 2-unit bounding box
        const box    = new THREE.Box3().setFromObject(model);
        const size   = box.getSize(new THREE.Vector3());
        const center = box.getCenter(new THREE.Vector3());
        const maxDim = Math.max(size.x, size.y, size.z);
        model.scale.setScalar(2 / maxDim);
        model.position.sub(center.multiplyScalar(2 / maxDim));

        scene.add(model);
        loader.style.display = 'none';
      },
      undefined,
      function onError(err) {
        console.error('GLB load error:', err);
        loader.innerHTML = '<p style="color:#f87171;font-size:0.8rem">Failed to load model</p>';
      }
    );

    // --- Responsive resize ---
    window.addEventListener('resize', () => {
      renderer.setSize(wrap.clientWidth, wrap.clientHeight);
    });

    // --- Animation loop ---
    function animate() {
      requestAnimationFrame(animate);
      const t = clock.getElapsedTime();

      if (model) {
        model.rotation.y += 0.003;
        model.position.y  = Math.sin(t * 0.8) * 0.08;
      }

      controls.update();
      renderer.render(scene, camera);
    }
    animate();
  </script>
</body>
</html>
```

## Hands-On Exercises

1. Generate a product or object on Meshy.ai using a text prompt such as "futuristic sneaker" or "crystal gem".
Download the GLB export and swap out the `MODEL_URL` constant in the code above.

2. Change the two directional light colors from teal/purple to match a brand palette of your choice.
Observe how dramatically lighting affects the mood of a 3D scene.

3. Add a second `DirectionalLight` pointing from below (negative Y position) at low intensity.
Notice how it removes harsh under-shadows and gives the model a more polished look.

4. Experiment with the canvas border-radius CSS property: try `border-radius: 16px` instead of `50%` for a square card aesthetic.

5. Replace the placeholder button with a form element and observe how the layout adapts on mobile by resizing your browser window.

## Resources

- [Three.js Documentation](https://threejs.org/docs/)
- [Three.js GLTFLoader examples](https://threejs.org/examples/#webgl_loader_gltf)
- [Meshy.ai - AI 3D model generation](https://www.meshy.ai/)
- [KhronosGroup GLTF Sample Models (GitHub)](https://github.com/KhronosGroup/glTF-Sample-Models)
- [gltf.report - Optimize and inspect GLB files](https://gltf.report/)

## Reflection Questions

1. Why is it important to center and scale a GLB model after loading it, rather than relying on the model's built-in origin?

2. What would happen to the animation loop if you called `requestAnimationFrame` only once rather than recursively inside `animate()`?

3. When would you choose Spline's `<spline-viewer>` embed over writing a custom Three.js scene?

4. How would you show a low-resolution static image of the model on mobile devices to save bandwidth and battery?

5. The hero uses `alpha: true` on the renderer.
What does this enable, and when would you set it to `false` instead?

## Next Steps

**Week 2 starts tomorrow.**
You move from foundations to AI-powered workflows that turn photographs and sketches directly into 3D assets.

- Day 8: Image-to-3D AI tools - NeRF, photogrammetry, and diffusion-based approaches
- Day 9: AI texture generation - PBR maps, Polyhaven, and applying textures in Three.js
