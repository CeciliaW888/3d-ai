# Day 5: React Three Fiber: 3D in React

## Learning Objectives
- Explain what React Three Fiber is and how it maps to Three.js concepts
- Build a 3D scene using the `<Canvas>` component and declarative JSX
- Animate objects using the `useFrame` hook
- Load a GLB model with `useGLTF` from `@react-three/drei`
- Enhance a scene with `Environment`, `OrbitControls`, and `Html` from drei

## Core Concepts

### What is React Three Fiber?

React Three Fiber (R3F) is a React renderer for Three.js.
Instead of writing imperative code like `scene.add(mesh)` and `renderer.render(scene, camera)`, you describe your 3D scene as a React component tree using JSX.

R3F maps every Three.js class to a JSX element automatically.
`new THREE.Mesh()` becomes `<mesh>`, `new THREE.BoxGeometry()` becomes `<boxGeometry>`, and `new THREE.MeshStandardMaterial()` becomes `<meshStandardMaterial>`.
Constructor arguments become the `args` prop, and object properties become JSX attributes.

The key advantage for React developers: your 3D scene is now just components.
You can use state, effects, context, and any React library alongside your 3D content.

### The Canvas Component

Everything starts with `<Canvas>`, which creates the WebGL context, default camera, and renderer.

```jsx
import { Canvas } from '@react-three/fiber';

function App() {
  return (
    <Canvas camera={{ position: [0, 1, 4], fov: 75 }}>
      {/* 3D scene goes here */}
    </Canvas>
  );
}
```

`<Canvas>` fills its parent element by default.
Wrap it in a div with explicit width and height to control its size.

### JSX Scene Graph

A basic spinning cube in R3F is a self-contained component:

```jsx
function SpinningCube() {
  const meshRef = useRef();
  useFrame((state, delta) => {
    meshRef.current.rotation.y += delta;
  });
  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="teal" roughness={0.4} />
    </mesh>
  );
}
```

Notice that JSX children of a `<mesh>` automatically attach to the mesh - `<boxGeometry>` sets `mesh.geometry` and `<meshStandardMaterial>` sets `mesh.material`.
R3F figures out the attachment type from the Three.js class name.

### Core Hooks

**`useFrame(callback)`** - runs every animation frame, inside the R3F render loop.
The callback receives `state` (containing the scene, camera, clock, etc.) and `delta` (seconds since last frame).
Using `delta` for rotations makes animation framerate-independent.

**`useThree()`** - accesses the R3F context from inside any component inside `<Canvas>`.
Returns `{ scene, camera, renderer, size, gl }`.
Useful when you need to interact directly with underlying Three.js objects.

**`useLoader(LoaderClass, url)`** - loads any Three.js loader asynchronously.
Returns the loaded asset.
Must be used inside a `<Suspense>` boundary.

### @react-three/drei

Drei (German for "three") is the official utility library for R3F.
It provides pre-built components for the most common Three.js patterns.

**`<OrbitControls>`** - adds orbit/pan/zoom mouse controls with one line.

**`<Environment preset="city">`** - adds HDR environment lighting from a preset.
This is the fastest way to make materials look realistic.

**`<Html>`** - renders DOM HTML anchored to a 3D position.
Use it for labels, tooltips, or UI overlays that need to exist in 3D space.

**`useGLTF(url)`** - an enhanced GLTFLoader hook with automatic caching and preloading.
Returns `{ scene, nodes, materials, animations }`.

## Key Terminology

| Term | Definition |
|------|-----------|
| React Three Fiber | A React renderer that lets you describe Three.js scenes as JSX component trees |
| `<Canvas>` | The R3F root component - creates the WebGL renderer, camera, and render loop |
| `useFrame` | Hook that runs a callback every animation frame - your main animation tool |
| `useThree` | Hook to access the underlying Three.js renderer, scene, and camera |
| `useGLTF` | Drei hook that loads a GLB/GLTF model with caching |
| `<Environment>` | Drei component that sets HDR environment lighting from a preset image |
| `drei` | The `@react-three/drei` helper library - pre-built components for R3F |
| `args` | R3F prop for passing constructor arguments to a Three.js class |
| `<Suspense>` | React component used to show a fallback while async assets (models) load |
| `delta` | Seconds since the last frame - use it to make animations framerate-independent |

## Code Examples

### Project Setup

```bash
npm create vite@latest my-r3f-app -- --template react
cd my-r3f-app
npm install three @react-three/fiber @react-three/drei
npm run dev
```

### Complete R3F Scene with Model Loading

Replace `src/App.jsx` with:

```jsx
import { Suspense, useRef } from 'react';
import { Canvas, useFrame } from '@react-three/fiber';
import {
  OrbitControls,
  Environment,
  useGLTF,
  Html,
} from '@react-three/drei';

// Loads a GLB model exported from Meshy or any other tool
function Model({ url }) {
  const { scene } = useGLTF(url);
  return <primitive object={scene} scale={1} />;
}

// A spinning teal cube as a fallback / placeholder
function SpinningCube() {
  const ref = useRef();
  useFrame((state, delta) => {
    ref.current.rotation.y += delta * 0.8;
  });
  return (
    <mesh ref={ref}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="#0d9488" roughness={0.3} metalness={0.2} />
    </mesh>
  );
}

// Loading fallback shown while the model fetches
function LoadingPlaceholder() {
  return (
    <Html center>
      <div style={{ color: 'white', fontFamily: 'sans-serif' }}>
        Loading model…
      </div>
    </Html>
  );
}

export default function App() {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <Canvas camera={{ position: [0, 1.5, 4], fov: 60 }}>
        {/* Environment lighting - no explicit light components needed */}
        <Environment preset="city" />

        {/* Suspense shows the fallback while the model is loading */}
        <Suspense fallback={<LoadingPlaceholder />}>
          <Model url="/model.glb" />
        </Suspense>

        {/* Uncomment to use the cube instead of a model */}
        {/* <SpinningCube /> */}

        {/* Interactive camera controls */}
        <OrbitControls enablePan={false} minDistance={2} maxDistance={10} />
      </Canvas>
    </div>
  );
}
```

Place your GLB file in the `public/` folder as `model.glb`.
R3F's `useGLTF` preloads the model at the module level for best performance:

```js
// Add this after the Model component definition
useGLTF.preload('/model.glb');
```

## Hands-On Exercises

1. Get the project running with the spinning cube.
Add a second `<SpinningCube />` component inside the Canvas and offset it using the `position` prop: `<SpinningCube position={[2, 0, 0]} />`.

2. Export a model from Meshy (from Day 3) as GLB, copy it to `public/model.glb`, and load it with the `<Model>` component.
Adjust the `scale` prop if it appears too large or small.

3. Try different `<Environment>` presets: `"sunset"`, `"dawn"`, `"warehouse"`, `"forest"`.
Notice how each changes the color and feel of the reflections.

4. Add a floating `<Html>` label inside the Canvas, anchored to `position={[0, 1.5, 0]}`.
Give it any text you like.
Observe how it stays attached to that 3D point as you orbit.

5. Read `state.clock.elapsedTime` inside `useFrame` to drive a sine-wave hover animation: `mesh.position.y = Math.sin(state.clock.elapsedTime) * 0.3`.

## Resources

- [React Three Fiber Docs](https://r3f.docs.pmnd.rs/) - official documentation for R3F
- [Drei Documentation](https://github.com/pmndrs/drei) - full list of helper components with demos
- [R3F Examples on Codesandbox](https://codesandbox.io/examples/package/@react-three/fiber) - interactive working examples
- [Three.js Docs](https://threejs.org/docs/) - still useful for understanding underlying Three.js classes
- [GLTF Transform](https://gltf-transform.dev/) - optimize and compress GLB files before loading them

## Reflection Questions

1. What is the key difference between the `useFrame` hook and a plain React `useEffect`?
2. Why do we wrap model-loading components in `<Suspense>` rather than handling loading state manually?
3. How does `<Environment preset="city">` replace the need for explicit light components in the scene?
4. What does the `args` prop do in a JSX element like `<boxGeometry args={[1, 1, 1]} />`?
5. If you wanted to access the Three.js camera directly from inside a Canvas component, which hook would you use?

## Next Steps

Day 6 introduces Spline - a visual, no-code 3D editor designed specifically for web embeds.
You will build an interactive 3D hero section without writing any Three.js code at all.

---

[← Day 4: Three.js Fundamentals](./day-04-threejs-fundamentals.md) | [Day 6: Spline for Web →](./day-06-spline-for-web.md)
