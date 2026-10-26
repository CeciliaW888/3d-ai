# Day 14: Week 2 Capstone - 3D Product Configurator

## Learning Objectives

- Build a complete, interactive 3D product configurator with React Three Fiber
- Load and display a GLB model using `useGLTF` with a Suspense loading state
- Implement a color/material switcher that updates the model in real time
- Add professional lighting with `Environment` and `OrbitControls` from Drei
- Deploy the finished project to Vercel or Netlify

## Core Concepts

### What Is a 3D Product Configurator?

A product configurator lets a user customize a 3D product in real time - changing color, material, or parts - before buying or ordering.
Brands like Nike, Tesla, and IKEA use configurators to reduce returns and increase purchase confidence.
React Three Fiber makes it practical to embed this experience directly on a website with no plugin required.

### Architecture Overview

Our configurator has four layers:

1. **Model layer** - a GLB file loaded with `useGLTF`
2. **Material layer** - a `MeshStandardMaterial` whose color we update via React state
3. **Camera layer** - `OrbitControls` so the user can orbit, zoom, and pan
4. **Environment layer** - an HDR preset that provides realistic reflections

The React component tree looks like this:

```
<Canvas>
  <Suspense fallback={<LoadingCube />}>
    <Environment preset="city" />
    <ProductModel color={selectedColor} />
    <OrbitControls enableDamping />
  </Suspense>
</Canvas>
```

### GLB Materials and useGLTF

`useGLTF` returns a `scene` object that is a Three.js `Group`.
To change the color of the model, traverse the scene and update every `MeshStandardMaterial`.
The `useEffect` hook re-runs whenever the `color` prop changes, making the update reactive.

### Environment and Lighting

`<Environment preset="city" />` from Drei loads a pre-built HDR environment map that provides both ambient illumination and realistic reflections for metallic/glossy materials.
It replaces the need for manually adding `AmbientLight` and `DirectionalLight` in most product scenes.
The available presets include `"apartment"`, `"city"`, `"dawn"`, `"forest"`, `"lobby"`, `"night"`, `"park"`, `"studio"`, `"sunset"`, and `"warehouse"`.

### Deployment

React Three Fiber projects built with Vite produce a standard static `dist/` folder.
That folder can be deployed to any static host - Vercel, Netlify, or GitHub Pages all work.
The only special consideration is that your GLB file must be in the `public/` folder so Vite copies it as-is without bundling it.

## Key Terminology

| Term | Definition |
|------|-----------|
| `useGLTF` | Drei hook that loads and caches a GLB/GLTF file, returning its scene, animations, and materials |
| `Suspense` | React boundary that shows a fallback component while async children (like a loading model) are not yet ready |
| `MeshStandardMaterial` | Physically based material in Three.js/R3F that supports metalness, roughness, and environment map reflections |
| `OrbitControls` | Drei component that adds mouse/touch camera orbiting, zooming, and panning to a canvas |
| `Environment` | Drei component that applies an HDR image as the scene's lighting and reflection source |
| `useEffect` | React hook used in R3F to respond to prop changes and update Three.js objects imperatively |
| `traverse` | Three.js `Object3D` method that walks every descendant in a scene graph, used to find all meshes in a loaded model |

## Code Examples

### Complete Product Configurator

Create a new Vite React project and install the dependencies:

```bash
npm create vite@latest product-configurator -- --template react
cd product-configurator
npm install three @react-three/fiber @react-three/drei
```

Place your GLB file at `public/models/product.glb`.
Replace the contents of `src/App.jsx` with the following:

```jsx
import { Suspense, useState, useEffect, useRef } from 'react'
import { Canvas } from '@react-three/fiber'
import {
  useGLTF,
  OrbitControls,
  Environment,
  ContactShadows,
} from '@react-three/drei'
import * as THREE from 'three'

/* ------------------------------------------------------------------
   Color options the user can choose from
   ------------------------------------------------------------------ */
const COLOR_OPTIONS = [
  { label: 'Midnight Black', hex: '#1a1a1a' },
  { label: 'Arctic White',   hex: '#f5f5f5' },
  { label: 'Ocean Blue',     hex: '#1a6eb5' },
  { label: 'Sunset Red',     hex: '#c0392b' },
  { label: 'Forest Green',   hex: '#27ae60' },
  { label: 'Sand Gold',      hex: '#d4ac0d' },
]

/* ------------------------------------------------------------------
   Fallback shown while the GLB is loading
   ------------------------------------------------------------------ */
function LoadingCube() {
  const meshRef = useRef()

  // Spin the fallback cube to indicate loading
  // (useFrame is not available outside Canvas, so we use a simple rotation in the render loop)
  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="#888888" wireframe />
    </mesh>
  )
}

/* ------------------------------------------------------------------
   The actual 3D model component
   ------------------------------------------------------------------ */
function ProductModel({ color }) {
  // useGLTF caches the file — hot-reloads work without re-downloading
  const { scene } = useGLTF('/models/product.glb')

  useEffect(() => {
    // Walk every object in the loaded scene
    scene.traverse((child) => {
      if (child.isMesh) {
        // Clone material so we don't mutate the cached original
        if (!child.userData.originalMaterial) {
          child.userData.originalMaterial = child.material
          child.material = child.material.clone()
        }
        child.material.color.set(color)
        child.material.needsUpdate = true
      }
    })
  }, [scene, color])

  return (
    <primitive
      object={scene}
      scale={1.5}
      position={[0, -0.5, 0]}
      castShadow
      receiveShadow
    />
  )
}

// Pre-load the model so it starts downloading immediately
useGLTF.preload('/models/product.glb')

/* ------------------------------------------------------------------
   Color Swatch UI (rendered outside the Canvas as plain HTML/CSS)
   ------------------------------------------------------------------ */
function ColorPicker({ options, selected, onSelect }) {
  return (
    <div className="color-picker">
      <p className="color-picker__label">
        {options.find(o => o.hex === selected)?.label ?? 'Custom'}
      </p>
      <div className="color-picker__swatches">
        {options.map((opt) => (
          <button
            key={opt.hex}
            className={`swatch${selected === opt.hex ? ' swatch--active' : ''}`}
            style={{ '--swatch-color': opt.hex }}
            onClick={() => onSelect(opt.hex)}
            aria-label={opt.label}
            title={opt.label}
          />
        ))}
      </div>
    </div>
  )
}

/* ------------------------------------------------------------------
   Root App
   ------------------------------------------------------------------ */
export default function App() {
  const [selectedColor, setSelectedColor] = useState(COLOR_OPTIONS[0].hex)

  return (
    <div className="app">
      <header className="app__header">
        <h1>3D Product Configurator</h1>
        <p>Click and drag to rotate &bull; Scroll to zoom</p>
      </header>

      {/* Three.js canvas — fills the main area */}
      <div className="canvas-wrapper">
        <Canvas
          shadows
          camera={{ position: [0, 1, 4], fov: 45 }}
          gl={{ antialias: true, toneMapping: THREE.ACESFilmicToneMapping }}
        >
          {/* Professional environment lighting */}
          <Environment preset="studio" background={false} />

          {/* The model, wrapped in Suspense for loading state */}
          <Suspense fallback={<LoadingCube />}>
            <ProductModel color={selectedColor} />
          </Suspense>

          {/* Subtle ground shadow */}
          <ContactShadows
            position={[0, -0.8, 0]}
            opacity={0.4}
            scale={5}
            blur={2}
          />

          {/* Camera controls */}
          <OrbitControls
            enableDamping
            dampingFactor={0.05}
            minDistance={2}
            maxDistance={8}
            maxPolarAngle={Math.PI / 1.8}
          />
        </Canvas>
      </div>

      {/* Color picker UI below the canvas */}
      <ColorPicker
        options={COLOR_OPTIONS}
        selected={selectedColor}
        onSelect={setSelectedColor}
      />
    </div>
  )
}
```

Add these styles to `src/index.css`:

```css
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: system-ui, sans-serif;
  background: #111;
  color: #eee;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.app__header {
  padding: 1rem 1.5rem;
  border-bottom: 1px solid #333;
}

.app__header h1 { font-size: 1.2rem; font-weight: 600; }
.app__header p  { font-size: 0.8rem; color: #888; margin-top: 0.25rem; }

.canvas-wrapper { flex: 1; }

.color-picker {
  padding: 1rem 1.5rem;
  border-top: 1px solid #333;
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.color-picker__label { font-size: 0.9rem; min-width: 120px; }

.color-picker__swatches { display: flex; gap: 0.5rem; flex-wrap: wrap; }

.swatch {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: var(--swatch-color);
  border: 2px solid #444;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s;
}

.swatch:hover       { transform: scale(1.15); border-color: #888; }
.swatch--active     { border-color: #fff; transform: scale(1.15); }
```

### Deployment

```bash
# Build the project
npm run build

# Deploy to Vercel (install Vercel CLI once: npm i -g vercel)
vercel --prod

# OR deploy to Netlify (install Netlify CLI once: npm i -g netlify-cli)
netlify deploy --prod --dir=dist
```

## Hands-On Exercises

1. Get the configurator running locally with any GLB model.
   Download a free model from [Sketchfab](https://sketchfab.com/features/free-3d-models) or use a model from a previous day.
2. Add a second property switch: create a `roughness` slider (range 0 to 1) and wire it to the material's `roughness` property using the same `useEffect` pattern.
3. Try changing the `Environment` preset from `"studio"` to `"sunset"` - observe how it affects the look of metallic vs. matte colors.
4. Add a screenshot button that calls `renderer.domElement.toDataURL()` and triggers a download.
   Look up how to access the renderer in R3F using `useThree`.
5. Deploy the finished configurator to Vercel and share the link.

## Resources

- [React Three Fiber Docs - useGLTF](https://docs.pmnd.rs/react-three-fiber/api/hooks) - official hook reference
- [Drei Docs - Environment](https://github.com/pmndrs/drei#environment) - all HDR preset names
- [Drei Docs - OrbitControls](https://github.com/pmndrs/drei#orbitcontrols) - full props reference
- [Sketchfab Free Models](https://sketchfab.com/features/free-3d-models) - GLB source files for testing
- [Vercel Deployment Docs](https://vercel.com/docs/deployments/overview) - static site deployment guide

## Reflection Questions

1. Why is the `useGLTF.preload()` call placed outside the component function?
2. Why do we clone the material in the `traverse` callback instead of modifying it directly?
3. What would break if you put the color picker UI inside the `<Canvas>` element?
4. How would you extend this configurator to let the user switch between different model files (e.g., shoe vs. bag)?
5. What are the trade-offs between using `Environment` for lighting vs. manually placing lights?

## Next Steps

Week 3 begins tomorrow with 3D browser games - you will learn the game loop, keyboard input, and physics fundamentals using Three.js and Rapier.js.

- Previous: [Day 13 - AI as Your 3D Coding Partner](./day-13-ai-coding-partner.md)
- Next: [Day 15 - Browser 3D Games with Three.js](../week-03/day-15-browser-3d-games.md)
