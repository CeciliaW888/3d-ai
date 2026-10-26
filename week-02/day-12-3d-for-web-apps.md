# Day 12: 3D for Web Apps: Configurators & Viewers

## Learning Objectives

- Explain the purpose of a 3D product configurator and list its core UI affordances.
- Use React Three Fiber with `useState` to switch between materials or colors on a 3D model.
- Set up an environment map to enable physically based reflections.
- Use `<Suspense>` and a loading fallback to handle async 3D asset loading gracefully.
- Embed a model viewer using the `@google/model-viewer` web component with minimal setup.

## Core Concepts

### What Is a 3D Configurator?

A 3D product configurator is an interactive web component that lets users customize a product - changing color, material, finish, or accessories - and see the result in real time on a photorealistic 3D model.
Nike's "Nike By You" shoe builder and car manufacturers' online spec tools are canonical examples.
Configurators increase purchase conversion because customers build emotional ownership before buying.

For developers, a configurator is fundamentally a state machine: the selected options live in React state, and the 3D model reads that state on every render to update its materials.

### The R3F Configurator Pattern

The pattern uses three React Three Fiber concepts working together.

**`useState` for selection.**
A simple `useState` hook holds the currently selected option - a color hex string, a material name, or an index into a materials array.
When the user clicks a swatch, `setSelectedColor` updates state and R3F's reconciler re-renders only the affected mesh.

**`meshStandardMaterial` for physically based materials.**
`meshStandardMaterial` supports the full PBR (physically based rendering) pipeline: `color`, `roughness`, `metalness`, `map` (albedo texture), `normalMap`, and `envMapIntensity`.
Swapping `color` is instant.
Swapping textures requires loading, so you should preload them.

**Environment maps for reflections.**
A plain material without environment lighting looks flat.
`@react-three/drei` provides `<Environment>` which wraps the scene in an HDR environment map - a panoramic image of a real or synthetic environment.
Metallic and glossy materials pick up reflections from it, making configurators look photorealistic with almost no extra code.

```jsx
import { Environment } from '@react-three/drei';
// Inside <Canvas>:
<Environment preset="studio" />
```

Preset names include `"studio"`, `"city"`, `"sunset"`, `"dawn"`, `"night"`, `"warehouse"`, and `"forest"`.

### Loading States with Suspense

3D models are large files.
Without a loading state, the user sees a blank canvas for seconds after arriving.
React `<Suspense>` integrates with R3F's asset loaders to show a fallback while the model downloads.

`@react-three/drei` provides `<Html>` and `useProgress` for a progress bar, and `<Loader>` for a ready-made loading overlay.

```jsx
import { Suspense } from 'react';
import { Loader } from '@react-three/drei';

<Canvas>
  <Suspense fallback={null}>
    <ProductModel />
  </Suspense>
</Canvas>
<Loader /> {/* renders outside Canvas, shows progress */}
```

### Orbit Controls for Model Viewing

A viewer that the user cannot rotate is frustrating.
`<OrbitControls>` from `@react-three/drei` adds mouse/touch rotate, zoom, and pan with one line.

```jsx
import { OrbitControls } from '@react-three/drei';
// Inside <Canvas>:
<OrbitControls enablePan={false} minDistance={2} maxDistance={8} />
```

Disabling pan (`enablePan={false}`) keeps the product centered - for configurators, you usually want users to orbit, not drag the model off-screen.

### @google/model-viewer

If you don't need React and just want a model viewer on an HTML page with zero JavaScript, use `@google/model-viewer`.
It is a custom HTML element (web component) that handles loading, orbit controls, AR mode, and environment lighting automatically.

```html
<script type="module" src="https://ajax.googleapis.com/ajax/libs/model-viewer/3.3.0/model-viewer.min.js"></script>

<model-viewer
  src="/models/product.glb"
  alt="A product model"
  auto-rotate
  camera-controls
  environment-image="neutral"
  shadow-intensity="1"
  style="width: 100%; height: 500px;">
</model-viewer>
```

Model-viewer is ideal for e-commerce product pages, documentation, and anywhere a simple drop-in viewer is sufficient.
For full configurator logic - color pickers, material switching, custom UI - React Three Fiber gives you more control.

### Annotations and Hotspots

Both model-viewer and custom R3F scenes support annotations - floating labels anchored to 3D points on the model.
In model-viewer these are `<button slot="hotspot-*">` elements positioned with `data-position` attributes in 3D space.
In R3F, you use `<Html>` from drei to render a DOM element at a world-space position that follows the camera as it orbits.

## Key Terminology

| Term | Definition |
|------|-----------|
| Configurator | An interactive 3D component that lets users customize product options and see updates in real time. |
| PBR | Physically Based Rendering: a shading model that simulates how light interacts with real materials. |
| Environment map | An HDR panoramic image wrapped around the scene to supply realistic reflections. |
| meshStandardMaterial | Three.js/R3F material that implements PBR with color, roughness, metalness, and map properties. |
| Suspense | React feature that pauses rendering a subtree while async data loads, showing a fallback. |
| OrbitControls | Three.js/drei component enabling mouse and touch rotate, zoom, and pan around a target. |
| model-viewer | Google's web component that embeds a GLB/GLTF viewer with AR support in plain HTML. |
| Hotspot | An annotation anchored to a 3D surface position, rendered as a DOM overlay that tracks the model. |

## Code Examples

### R3F Product Configurator with Color Switcher

```jsx
import { useState, Suspense } from 'react';
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Environment, useGLTF } from '@react-three/drei';

const COLOR_OPTIONS = [
  { label: 'Midnight', value: '#1a1a2e' },
  { label: 'Ocean',    value: '#0077b6' },
  { label: 'Coral',    value: '#e76f51' },
  { label: 'Sage',     value: '#74c69d' },
];

function ProductModel({ color }) {
  const { scene } = useGLTF('/models/product.glb');

  // Apply the selected color to every mesh in the model
  scene.traverse((node) => {
    if (node.isMesh && node.name.includes('Body')) {
      // Create a new material instance so we don't mutate the original
      if (!node.userData.originalMaterial) {
        node.userData.originalMaterial = node.material;
      }
      node.material = node.material.clone();
      node.material.color.set(color);
    }
  });

  return <primitive object={scene} />;
}

// Preload so it starts downloading immediately
useGLTF.preload('/models/product.glb');

export default function Configurator() {
  const [selectedColor, setSelectedColor] = useState(COLOR_OPTIONS[0].value);

  return (
    <div style={{ display: 'flex', gap: '2rem', alignItems: 'center' }}>
      {/* 3D Viewport */}
      <div style={{ width: '600px', height: '500px' }}>
        <Canvas camera={{ position: [0, 1, 4], fov: 45 }}>
          <Suspense fallback={null}>
            <ambientLight intensity={0.3} />
            <Environment preset="studio" />
            <ProductModel color={selectedColor} />
            <OrbitControls
              enablePan={false}
              minDistance={2}
              maxDistance={8}
              target={[0, 0.5, 0]}
            />
          </Suspense>
        </Canvas>
      </div>

      {/* Color Picker UI */}
      <div>
        <p style={{ fontWeight: 'bold', marginBottom: '0.5rem' }}>Choose a color:</p>
        <div style={{ display: 'flex', gap: '0.75rem' }}>
          {COLOR_OPTIONS.map((opt) => (
            <button
              key={opt.value}
              title={opt.label}
              onClick={() => setSelectedColor(opt.value)}
              style={{
                width: '40px',
                height: '40px',
                borderRadius: '50%',
                background: opt.value,
                border: selectedColor === opt.value ? '3px solid #fff' : '2px solid #555',
                outline: selectedColor === opt.value ? '2px solid #6d5eff' : 'none',
                cursor: 'pointer',
              }}
            />
          ))}
        </div>
        <p style={{ marginTop: '0.5rem', color: '#888' }}>
          {COLOR_OPTIONS.find((o) => o.value === selectedColor)?.label}
        </p>
      </div>
    </div>
  );
}
```

### Roughness and Metalness Switcher

```jsx
const MATERIAL_PRESETS = {
  matte:    { roughness: 0.9, metalness: 0.0 },
  satin:    { roughness: 0.4, metalness: 0.2 },
  chrome:   { roughness: 0.05, metalness: 1.0 },
};

function ProductMesh({ color, preset }) {
  const { roughness, metalness } = MATERIAL_PRESETS[preset];
  return (
    <mesh>
      <torusKnotGeometry args={[1, 0.3, 128, 32]} />
      <meshStandardMaterial
        color={color}
        roughness={roughness}
        metalness={metalness}
      />
    </mesh>
  );
}
```

## Hands-On Exercises

1. **Build the color configurator** - Run the full R3F example above with a model you downloaded from Meshy.ai or any free GLB.
   Confirm that clicking swatches changes the model color.
2. **Add a roughness slider** - Add an HTML `<input type="range">` that controls `roughness` from 0 to 1 and observe how highlights change.
3. **Try different environments** - Swap `<Environment preset="studio" />` for `"city"`, `"sunset"`, and `"warehouse"`.
   Note how each changes the perceived material.
4. **Drop in model-viewer** - Create a plain HTML file, add the `<model-viewer>` snippet, point it at your GLB, and open it in the browser.
   Enable `auto-rotate` and `camera-controls`.
5. **Add a loading indicator** - Import `<Loader>` from drei and add it to the configurator.
   Throttle your network to "Slow 3G" in DevTools to see the loading state.

## Resources

- [React Three Fiber docs](https://docs.pmnd.rs/react-three-fiber/getting-started/introduction)
- [Drei Environment component](https://github.com/pmndrs/drei#environment)
- [Drei OrbitControls](https://github.com/pmndrs/drei#orbitcontrols)
- [model-viewer documentation](https://modelviewer.dev/docs/)
- [Three.js meshStandardMaterial](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial)

## Reflection Questions

1. Why do you call `node.material.clone()` before changing the material color instead of editing it directly?
2. What does `useGLTF.preload('/models/product.glb')` do, and when should you call it?
3. A configurator loads fine on desktop but is slow on mobile.
   What are two things you could change in the 3D setup to improve mobile performance?
4. When would you choose `@google/model-viewer` over a custom React Three Fiber implementation?
5. Why does adding `<Environment preset="studio" />` make metallic materials look dramatically better even without adding any light components?

## Next Steps

Day 13 explores how to use AI coding assistants as your Three.js and GLSL writing partner - generating shader code, debugging material issues, and prototyping scenes faster with AI.

- Previous: [Day 11 - 3D for Websites: Scroll & Parallax](./day-11-3d-for-websites.md)
- Next: [Day 13 - AI as Your 3D Coding Partner](./day-13-ai-coding-partner.md)
