# Day 11: 3D for Websites: Scroll & Parallax

## Learning Objectives

- Describe at least three canvas positioning strategies (fixed, sticky, inline) and when to use each.
- Use a scroll event listener or IntersectionObserver to drive a Three.js camera or object transform.
- Integrate GSAP ScrollTrigger with a React Three Fiber scene to animate 3D objects on scroll.
- Build a parallax depth effect where foreground and background objects move at different scroll rates.
- Name real-world examples of scroll-driven 3D and explain what makes them effective.

## Core Concepts

### Why Scroll-Driven 3D?

Static 3D scenes are impressive, but scroll-driven 3D scenes are immersive.
When movement is tied to the user's own gesture, the brain interprets the experience as exploration rather than passive viewing.
That is the secret behind Apple's product reveal pages, where the iPhone rotates and disassembles as you scroll down.
The 3D isn't decorating the page - it *is* the page.

### Canvas Positioning Strategies

The first architectural decision for any scroll-driven scene is how the canvas sits in the document flow.

**Fixed canvas.**
The `<canvas>` has `position: fixed` and covers the full viewport at `z-index: -1` (or whatever layer suits).
The rest of the page scrolls normally over it.
JavaScript reads `window.scrollY` and maps it to camera or object transforms.
This approach is simple and works well when one 3D scene spans the whole page.

**Sticky canvas.**
A wrapper div has `height: 300vh` (or however many scroll lengths you need).
The canvas inside it has `position: sticky; top: 0`.
As the user scrolls through the tall wrapper, the canvas stays in view.
Once the wrapper scrolls out, the canvas follows it.
This is ideal for "chapters" of a page where 3D is active only in one section.

**Inline canvas.**
The canvas is a normal block element in document flow.
IntersectionObserver fires when it enters the viewport and pauses the render loop when it leaves.
This pattern suits product pages with multiple 3D moments separated by regular text content.
It is the most performance-friendly option because rendering only happens when visible.

### Mapping Scroll to 3D State

The core technique is normalizing scroll progress into a 0-to-1 value and using it to lerp (linearly interpolate) a 3D property.

```js
const scrollProgress = window.scrollY / (document.body.scrollHeight - window.innerHeight);
```

Once you have a 0-1 number, map it to anything: camera Z position, object Y rotation, material opacity, or morph target influence.

**Lerp for smoothness.**
Raw scroll values produce jittery motion.
Apply a lerp on every animation frame so the 3D state "chases" the scroll target:

```js
currentValue += (targetValue - currentValue) * 0.1; // 10% closer each frame
```

The 0.1 factor controls lag - smaller = more lag, larger = more snappy.

### IntersectionObserver Pattern

IntersectionObserver lets you start and stop the render loop based on canvas visibility.
This is important for performance on pages with many sections.

```js
const observer = new IntersectionObserver(
  ([entry]) => {
    if (entry.isIntersecting) {
      renderer.setAnimationLoop(animate);
    } else {
      renderer.setAnimationLoop(null); // pause when off-screen
    }
  },
  { threshold: 0 }
);
observer.observe(renderer.domElement);
```

### GSAP ScrollTrigger with React Three Fiber

GSAP (GreenSock Animation Platform) is a professional animation library with a ScrollTrigger plugin.
In React Three Fiber, you can use ScrollTrigger to drive `useRef` values linked to Three.js objects.

Install the dependencies:

```bash
npm install gsap @gsap/react
```

The key pattern is registering ScrollTrigger, creating a GSAP timeline pinned to a scroll container, and driving a ref that Three.js reads in `useFrame`.

### Parallax Depth Effect

Parallax is the visual phenomenon where objects at different distances appear to move at different speeds.
In 3D, you can fake convincing depth by layering objects and advancing their positions at different rates as the user scrolls.

A particle field in the far background might advance 0.1 units per 100px of scroll.
A foreground logo mesh might advance 0.5 units for the same scroll distance.
The resulting difference in motion speed reads as depth, even in a flat viewport.

### Real-World Examples

**Apple product pages** use WebGL to rotate and disassemble products synchronized to scroll - scroll reveals internal components one by one.
**Awwwards-winning sites** (awwwards.com) regularly feature scroll-driven WebGL experiences where geometry morphs or particles form words.
**Stripe's homepage** has used a subtle 3D gradient mesh that parallaxes on scroll and hover.
These experiences succeed because the scroll motion feels *native* - it never fights the user's gestures.

## Key Terminology

| Term | Definition |
|------|-----------|
| Scroll progress | A 0-to-1 normalized value representing how far the user has scrolled through a section. |
| Fixed canvas | Canvas with `position:fixed` that stays in place while page content scrolls over it. |
| Sticky canvas | Canvas that sticks to the viewport only while inside a tall scroll container. |
| Lerp | Linear interpolation: smoothly blending a current value toward a target each frame. |
| IntersectionObserver | Web API that fires a callback when an element enters or exits the viewport. |
| ScrollTrigger | GSAP plugin that ties animation timelines to the scroll position. |
| Parallax | The illusion of depth created by moving objects at different rates relative to the camera. |

## Code Examples

### Scroll-Driven Camera with Plain Three.js

```js
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 100);
camera.position.set(0, 0, 5);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(innerWidth, innerHeight);
renderer.domElement.style.position = 'fixed';
renderer.domElement.style.top = '0';
renderer.domElement.style.left = '0';
renderer.domElement.style.zIndex = '-1';
document.body.appendChild(renderer.domElement);

// Make the page tall enough to scroll
document.body.style.height = '400vh';

// Add something to look at
const geo = new THREE.TorusKnotGeometry(1, 0.3, 128, 32);
const mat = new THREE.MeshStandardMaterial({ color: 0x6d5eff });
const mesh = new THREE.Mesh(geo, mat);
scene.add(mesh);

scene.add(new THREE.AmbientLight(0xffffff, 0.5));
const point = new THREE.PointLight(0xffffff, 2, 20);
point.position.set(5, 5, 5);
scene.add(point);

// Target values (driven by scroll)
let targetRotationY = 0;
let targetCameraZ = 5;

// Current smoothed values
let currentRotationY = 0;
let currentCameraZ = 5;

window.addEventListener('scroll', () => {
  const scrollFraction = window.scrollY / (document.body.scrollHeight - window.innerHeight);
  // Rotate the mesh a full turn as user scrolls from top to bottom
  targetRotationY = scrollFraction * Math.PI * 2;
  // Move camera closer as user scrolls
  targetCameraZ = 5 - scrollFraction * 3;
});

function animate() {
  requestAnimationFrame(animate);

  // Lerp toward targets for smooth motion
  currentRotationY += (targetRotationY - currentRotationY) * 0.08;
  currentCameraZ += (targetCameraZ - currentCameraZ) * 0.08;

  mesh.rotation.y = currentRotationY;
  camera.position.z = currentCameraZ;

  renderer.render(scene, camera);
}

animate();
```

### React Three Fiber with GSAP ScrollTrigger

```jsx
import { useRef, useEffect } from 'react';
import { Canvas, useFrame } from '@react-three/fiber';
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

function SpinningBox() {
  const meshRef = useRef();
  const rotationTarget = useRef(0);

  useEffect(() => {
    // Scroll container must exist in the DOM
    ScrollTrigger.create({
      trigger: '#scroll-section',
      start: 'top top',
      end: 'bottom bottom',
      scrub: 1, // smooth scrubbing
      onUpdate: (self) => {
        // self.progress goes 0 -> 1 as user scrolls through the section
        rotationTarget.current = self.progress * Math.PI * 4; // two full rotations
      },
    });
    return () => ScrollTrigger.killAll();
  }, []);

  useFrame(() => {
    if (!meshRef.current) return;
    // Lerp inside useFrame for smooth following
    meshRef.current.rotation.y +=
      (rotationTarget.current - meshRef.current.rotation.y) * 0.1;
  });

  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1.5, 1.5, 1.5]} />
      <meshStandardMaterial color="#6d5eff" />
    </mesh>
  );
}

export default function ScrollScene() {
  return (
    <>
      {/* Tall scroll container */}
      <div id="scroll-section" style={{ height: '300vh' }}>
        {/* Canvas is sticky inside the scroll container */}
        <div style={{ position: 'sticky', top: 0, height: '100vh' }}>
          <Canvas camera={{ position: [0, 0, 4] }}>
            <ambientLight intensity={0.5} />
            <pointLight position={[5, 5, 5]} intensity={2} />
            <SpinningBox />
          </Canvas>
        </div>
      </div>
    </>
  );
}
```

## Hands-On Exercises

1. **Implement the fixed canvas example** - Run the plain Three.js scroll demo and confirm the object rotates smoothly as you scroll.
   Tweak the lerp factor (0.08) and observe how it changes the feel.
2. **Add a second object** - Add a second mesh at a different Z depth.
   Make it scroll at half the rate of the first to create a parallax effect.
3. **Try IntersectionObserver** - Add the observer pattern to pause rendering when the canvas is off-screen.
   Open DevTools > Performance and compare CPU usage with and without it.
4. **Upgrade to GSAP ScrollTrigger** - Convert the R3F example above so that the object also scales from 0.5 to 1.5 as you scroll.
5. **Study a real example** - Visit an Awwwards site and use the browser DevTools to identify whether it uses a fixed, sticky, or inline canvas.

## Resources

- [Three.js WebGLRenderer.setAnimationLoop](https://threejs.org/docs/#api/en/renderers/WebGLRenderer.setAnimationLoop)
- [React Three Fiber useFrame](https://docs.pmnd.rs/react-three-fiber/api/hooks#useframe)
- [GSAP ScrollTrigger docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- [MDN IntersectionObserver API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [Awwwards - scroll-driven site examples](https://www.awwwards.com/awwwards/collections/webgl/)

## Reflection Questions

1. What is the difference between a fixed canvas and a sticky canvas, and which is better for a multi-section page?
2. Why do you lerp inside `requestAnimationFrame` rather than directly setting the value in the scroll event?
3. What does the `scrub: 1` option do in a GSAP ScrollTrigger, and what would `scrub: true` do differently?
4. If a page has three separate 3D sections that are rarely all visible at once, what pattern would reduce GPU usage the most?
5. Describe in your own words how parallax creates the illusion of depth without changing the camera's projection.

## Next Steps

Day 12 moves from presentations to interactions - you'll build a 3D product configurator where users can switch colors and materials on a 3D model in real time.

- Previous: [Day 10 - AI Animation: Mixamo & Beyond](./day-10-ai-animation.md)
- Next: [Day 12 - 3D for Web Apps: Configurators & Viewers](./day-12-3d-for-web-apps.md)
