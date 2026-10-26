# Day 17: GLSL Shaders: Visual Magic

## Learning Objectives

- Understand how the GPU pipeline uses vertex and fragment shaders together.
- Read and write basic GLSL syntax: uniforms, varyings, and built-in variables.
- Create a `ShaderMaterial` in Three.js with a custom animated wave effect.
- Pass time and resolution as uniforms to animate a shader live.
- Use `shaderMaterial` from `@react-three/drei` in a React Three Fiber scene.

## Core Concepts

### What Are Shaders?

Everything rendered in WebGL goes through two small programs called shaders.
These programs run on the GPU - not the CPU - which is why shaders are extremely fast even when applied to millions of pixels.
The GPU runs both shader programs in parallel, once for every vertex and once for every pixel.

**Vertex shader** - runs once per vertex (corner point) of a mesh.
Its job is to calculate where each vertex appears on screen.
The required output is `gl_Position`, a `vec4` holding the clip-space coordinate.
You can also move vertices here to distort the mesh shape.

**Fragment shader** - runs once per pixel (fragment) that the mesh covers on screen.
Its job is to calculate the color of that pixel.
The required output is `gl_FragColor`, a `vec4` holding RGBA values in the 0.0 to 1.0 range.

Both shaders are written in **GLSL** (OpenGL Shading Language), a C-like language with built-in types for 2D, 3D, and 4D vectors and matrices.

### GLSL Syntax Essentials

```glsl
// Types
float f = 1.0;       // always use decimal point
vec2 uv = vec2(0.5, 0.5);
vec3 color = vec3(1.0, 0.0, 0.0);   // red
vec4 rgba  = vec4(color, 1.0);       // add alpha

// Swizzling - access components by name
vec3 rgb = rgba.rgb;     // same as vec3(rgba.r, rgba.g, rgba.b)
float x  = uv.x;        // or uv.r, or uv.s — all the same first component

// Built-in math functions
float t = sin(u_time);          // -1.0 to 1.0 oscillation
float f = fract(x * 5.0);      // fractional part — great for repeating patterns
float m = mix(a, b, t);        // linear interpolation: a*(1-t) + b*t
float c = clamp(v, 0.0, 1.0);  // constrain to range
```

### Uniforms and Varyings

**Uniforms** are values sent from your JavaScript code to both shaders.
They are the same ("uniform") across all vertices and fragments in one draw call.
Typical uniforms: time, resolution, mouse position, a custom color.

**Varyings** are values computed in the vertex shader and passed to the fragment shader.
They are interpolated across the surface - a varying that is 1.0 at one vertex and 0.0 at another will be 0.5 halfway between them.
The UV coordinate (`uv`) is built into Three.js and is automatically available as a varying when you use `ShaderMaterial`.

### ShaderMaterial in Three.js

`THREE.ShaderMaterial` lets you replace the built-in Phong/PBR shading with your own GLSL programs.

```js
import * as THREE from 'three';

const material = new THREE.ShaderMaterial({
  uniforms: {
    u_time:       { value: 0.0 },
    u_resolution: { value: new THREE.Vector2(window.innerWidth, window.innerHeight) }
  },
  vertexShader: `
    uniform float u_time;
    varying vec2 v_uv;

    void main() {
      v_uv = uv;
      vec3 pos = position;

      // Wave distortion: move Z based on X, Y, and time
      pos.z += sin(pos.x * 3.0 + u_time)        * 0.15;
      pos.z += sin(pos.y * 2.0 + u_time * 0.7)  * 0.15;

      gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
    }
  `,
  fragmentShader: `
    uniform float u_time;
    varying vec2 v_uv;

    void main() {
      vec3 teal   = vec3(0.05, 0.58, 0.53);
      vec3 orange = vec3(0.9,  0.40, 0.10);

      // Animated gradient from UV coordinates
      float wave = sin(v_uv.x * 6.28 + u_time) * 0.5 + 0.5;
      wave = mix(wave,
                 sin(v_uv.y * 6.28 - u_time * 0.6) * 0.5 + 0.5,
                 0.5);

      vec3 color = mix(teal, orange, wave);
      gl_FragColor = vec4(color, 1.0);
    }
  `
});

// Increment u_time in the animation loop
const clock = new THREE.Clock();
function animate() {
  requestAnimationFrame(animate);
  material.uniforms.u_time.value = clock.getElapsedTime();
  renderer.render(scene, camera);
}
```

Apply this material to any geometry:

```js
const geometry = new THREE.PlaneGeometry(4, 4, 32, 32);  // 32x32 segments for smooth wave
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```

### ShaderMaterial in React Three Fiber

The `shaderMaterial` helper from `@react-three/drei` creates a declarative shader material component.

```jsx
import { shaderMaterial } from '@react-three/drei';
import { extend, useFrame } from '@react-three/fiber';
import { useRef } from 'react';
import * as THREE from 'three';

const WaveMaterial = shaderMaterial(
  { u_time: 0 },
  // Vertex shader
  `
    uniform float u_time;
    varying vec2 v_uv;
    void main() {
      v_uv = uv;
      vec3 pos = position;
      pos.z += sin(pos.x * 3.0 + u_time) * 0.15;
      pos.z += sin(pos.y * 2.0 + u_time * 0.7) * 0.15;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
    }
  `,
  // Fragment shader
  `
    uniform float u_time;
    varying vec2 v_uv;
    void main() {
      vec3 teal   = vec3(0.05, 0.58, 0.53);
      vec3 orange = vec3(0.9, 0.4, 0.1);
      float wave  = sin(v_uv.x * 6.28 + u_time) * 0.5 + 0.5;
      gl_FragColor = vec4(mix(teal, orange, wave), 1.0);
    }
  `
);

extend({ WaveMaterial });

function WavePlane() {
  const mat = useRef();
  useFrame(({ clock }) => {
    if (mat.current) mat.current.u_time = clock.getElapsedTime();
  });
  return (
    <mesh>
      <planeGeometry args={[4, 4, 32, 32]} />
      <waveMaterial ref={mat} />
    </mesh>
  );
}
```

### Using AI to Write Shaders

GLSL has a steep learning curve - but AI assistants (Claude, ChatGPT, Copilot) are very good at writing shader code.
A prompt like "Write a GLSL fragment shader that creates a Voronoi noise pattern with animated cells. Use a u_time uniform." will produce working code you can paste directly into `ShaderMaterial`.
Always test AI-generated shaders in a sandbox first: GLSL Sandbox (glslsandbox.com) lets you paste and run fragment shaders instantly in the browser with zero setup.

## Key Terminology

| Term | Definition |
|------|-----------|
| GLSL | OpenGL Shading Language - the C-like language used to write GPU shaders |
| Vertex shader | GPU program that runs per vertex, outputting `gl_Position` |
| Fragment shader | GPU program that runs per pixel, outputting `gl_FragColor` |
| Uniform | A value sent from JS to both shaders, constant across one draw call |
| Varying | A value interpolated from the vertex shader to the fragment shader |
| `gl_Position` | Built-in GLSL output: clip-space position of a vertex (vec4) |
| `gl_FragColor` | Built-in GLSL output: final RGBA color of a pixel (vec4) |
| Swizzling | Accessing vector components by name: `.rgb`, `.xy`, `.zwx`, etc. |

## Code Examples

See the complete `ShaderMaterial` example with animated wave in Core Concepts above.
Copy and run it in an existing Three.js scene with any PlaneGeometry.
The only requirement is a `PlaneGeometry` with enough segments (at least 16x16) for the wave displacement to appear smooth.

## Hands-On Exercises

1. **Run the wave shader.** Copy the `ShaderMaterial` example from Core Concepts into a Three.js scene.
   Set the geometry to `new THREE.PlaneGeometry(4, 4, 32, 32)`.
   Confirm the animated gradient and wave distortion both work.

2. **Modify the frequency.** Change `3.0` and `2.0` in the vertex shader to other values.
   Observe how higher numbers create more tightly packed waves.
   Try values like 8.0 and 0.5.

3. **Add a third color.** In the fragment shader, introduce a `vec3 purple = vec3(0.5, 0.0, 0.9)`.
   Use a second `mix()` to blend between your current color and purple based on `v_uv.y`.

4. **Experiment on GLSL Sandbox.** Visit glslsandbox.com.
   Paste this minimal fragment shader and run it:
   `void main() { float t = sin(time + gl_FragCoord.x * 0.01) * 0.5 + 0.5; gl_FragColor = vec4(t, 0.5, 1.0 - t, 1.0); }`
   Modify the multipliers and see the effect live.

5. **Ask AI for a shader.** Prompt an AI: "Write a GLSL ShaderMaterial for Three.js that creates a ripple effect from the center of the mesh. Use a u_time uniform."
   Paste the result into your scene and test it.

## Resources

- [Three.js ShaderMaterial documentation](https://threejs.org/docs/#api/en/materials/ShaderMaterial)
- [GLSL Sandbox - live shader editor](http://glslsandbox.com/)
- [The Book of Shaders - free GLSL learning guide](https://thebookofshaders.com/)
- [@react-three/drei shaderMaterial](https://github.com/pmndrs/drei#shadermaterial)
- [Shadertoy - community shader gallery](https://www.shadertoy.com/)

## Reflection Questions

1. Why does a fragment shader run many more times per frame than a vertex shader?
2. What is the difference between a uniform and a varying?
   Give an example of when you would use each one.
3. The vertex shader can move vertices, but can it change the color of the mesh?
   If not, how does color information get from the vertex stage to the fragment stage?
4. What does `mix(a, b, 0.0)` return, and what does `mix(a, b, 1.0)` return?
5. Why is `fract(sin(x) * 43758.5453)` commonly used in GLSL to generate pseudo-random values?

## Next Steps

Day 18 tackles the real-world challenge that will make or break your 3D web project: performance.
You will learn why draw calls are the biggest enemy of web 3D, how to profile your scene, and how `InstancedMesh` lets you render 1,000 objects for the cost of one.

- Previous: [Day 16 - AI Game Asset Creation](day-16-ai-game-assets.md)
- Next: [Day 18 - Performance Optimization for Web 3D](day-18-performance-optimization.md)
