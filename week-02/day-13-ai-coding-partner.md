# Day 13: AI as Your 3D Coding Partner

## Learning Objectives

- Use Claude and ChatGPT to generate working Three.js and React Three Fiber code
- Write effective prompts that produce accurate 3D code on the first try
- Recognize and fix common AI hallucinations in Three.js API calls
- Use AI to decode Three.js documentation and GLSL shader syntax
- Set up GitHub Copilot for 3D development workflows

## Core Concepts

### Why AI Accelerates 3D Development

3D programming has a steep learning curve.
APIs are large, shader syntax is cryptic, and the feedback loop (code, save, see broken scene) is slow.
AI coding assistants collapse that loop dramatically.
You can describe what you want in plain English and get a working starting point in seconds.
The key is knowing how to prompt well - and how to spot when the AI is wrong.

### Prompting Patterns That Work

**Give full context up front.**
AI models do not know which version of Three.js you are using or whether you are in a React project.
Always include this context at the start of your prompt.
For example: "I am using Three.js r168 with a Vite vanilla-JS setup. I am not using React."

**Describe the visual outcome, not the steps.**
Instead of "add a PointLight at position 2,3,2," say "I want warm ambient light that makes the scene feel like late afternoon sun."
AI is better at translating intent into implementation than at guessing what API calls you want.

**Iterate in the same conversation.**
Do not start a new chat for each follow-up.
Corrections like "the shadow is too sharp" or "make the rotation loop instead of stopping" work because the model still has the previous code in context.

**Ask for explanation alongside code.**
Add "explain each step with inline comments" to any prompt.
This doubles as a learning tool and helps you catch hallucinations because you can verify the reasoning.

### 5 Ready-to-Use Prompt Templates

**Template 1 - Basic Three.js scene:**
```
I'm using Three.js r168 with a Vite + vanilla JS setup.
Write a complete scene with:
- A WebGLRenderer that fills the window and resizes correctly
- A PerspectiveCamera at position (0, 2, 5)
- An ambient light (intensity 0.5) and a directional light with soft shadows
- A rotating metallic sphere using MeshStandardMaterial
- An animation loop using requestAnimationFrame
Include a resize event listener and dispose cleanup.
```

**Template 2 - React Three Fiber component:**
```
I'm using React Three Fiber with @react-three/drei.
Write a self-contained React component called <ProductViewer> that:
- Loads a GLB file from /models/shoe.glb using useGLTF
- Shows a Suspense fallback with a spinning placeholder cube
- Uses OrbitControls (from drei) with damping enabled
- Adds an Environment preset="city" for lighting
- Accepts a `color` prop and applies it to the first MeshStandardMaterial in the loaded model
The component should be exported as default.
```

**Template 3 - GLSL shader:**
```
I'm writing a Three.js ShaderMaterial.
Write a vertex shader and fragment shader pair that creates an animated wave effect.
The vertex shader should displace Y position using sin(position.x * frequency + time) * amplitude.
The fragment shader should color the mesh with a gradient from deep blue at the base to white at the peaks.
Include the required uniforms: time (float), frequency (float), amplitude (float).
Show how to pass these uniforms when creating the ShaderMaterial.
```

**Template 4 - Debugging a broken scene:**
```
My Three.js scene renders a black screen. Here is my full setup code:
[PASTE CODE HERE]
I am using Three.js r168 and a Vite dev server.
What are the three most likely causes for a black screen, and which one matches my code?
Fix the issue and explain what was wrong.
```

**Template 5 - Performance optimization:**
```
I have a React Three Fiber scene with [describe your scene].
It runs at 25 FPS on a mid-range laptop.
Suggest five specific optimizations I can apply to this R3F scene to hit 60 FPS.
For each suggestion, show the before and after code.
Focus on geometry instancing, draw call reduction, and texture compression.
```

### Common Pitfalls and Hallucinations

**Outdated API syntax.**
Three.js releases frequently.
AI models trained before r152 may suggest `geometry.faces` (removed), `THREE.Geometry` (removed in r125), or `renderer.gammaOutput` (deprecated).
Always check the [Three.js migration guide](https://github.com/mrdoob/three.js/wiki/Migration-Guide) when something looks off.

**Invented method names.**
Models sometimes generate plausible-sounding but non-existent methods like `mesh.setMaterial()` or `loader.loadAsync()` with wrong signatures.
If you get a `TypeError: x is not a function`, search the Three.js docs directly before assuming your code is wrong.

**Wrong import paths.**
AI may suggest `import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'` when the correct path for newer setups is `import { OrbitControls } from 'three/addons/controls/OrbitControls.js'`.

**Stale React Three Fiber patterns.**
`useFrame` hooks, `extend`, and `<primitive>` syntax have evolved.
Always check the [R3F docs](https://docs.pmnd.rs/react-three-fiber) for the current idiom before trusting AI output.

### Using AI to Read the Docs

Three.js documentation is dense.
Try pasting a section of the docs into Claude with: "Explain what TextureLoader, CubeTextureLoader, and RGBELoader each do, when to use each, and show one real example of each."
This is faster than reading the docs linearly and produces context-specific explanations.

### GitHub Copilot for 3D Code

Copilot shines for boilerplate: it auto-completes renderer setup, geometry constructors, and uniform declarations.
Write a detailed comment above each function describing what it should do.
Copilot uses the comment as a prompt.
For shader code, start with the `uniform` and `varying` declarations - Copilot will often complete the rest of the function.

## Key Terminology

| Term | Definition |
|------|-----------|
| Prompt template | A reusable, parameterized text structure that reliably gets good AI output for a specific task type |
| Hallucination | When an AI generates plausible-looking but factually incorrect code, such as a method that does not exist |
| Context window | The amount of conversation history an AI model can use when generating a response |
| Migration guide | A document listing API changes between library versions, essential for catching outdated AI suggestions |
| Iterative refinement | The practice of building on AI output in the same conversation thread rather than starting fresh each time |
| Inline comments | Code comments that explain intent, used both to guide AI generation and to verify its reasoning |
| ShaderMaterial uniforms | Named variables passed from JavaScript into GLSL shaders, updated each frame to drive animations |

## Code Examples

### Validating AI-Generated Three.js Code

```js
// Step 1: Check for removed APIs before running
// Run this in the browser console if you suspect outdated code:
console.log(typeof THREE.Geometry);        // Should be "undefined" in r125+
console.log(typeof THREE.WebGLRenderer);   // Should be "function"
console.log(THREE.REVISION);              // Check the actual version

// Step 2: Verify a ShaderMaterial uniform object (AI sometimes gets the structure wrong)
const material = new THREE.ShaderMaterial({
  uniforms: {
    uTime:      { value: 0.0 },      // Correct: object with `value` key
    uColor:     { value: new THREE.Color(0x0077ff) },
    uAmplitude: { value: 0.5 },
  },
  vertexShader: vertSrc,
  fragmentShader: fragSrc,
});

// Step 3: Update uniforms in the animation loop (AI sometimes forgets this)
function animate() {
  requestAnimationFrame(animate);
  material.uniforms.uTime.value += 0.016; // ~60fps increment
  renderer.render(scene, camera);
}
animate();
```

### AI-Generated Shader (with verification checklist)

```glsl
// Vertex shader — check: does it declare `varying` for anything the frag shader needs?
uniform float uTime;
uniform float uFrequency;
uniform float uAmplitude;
varying float vElevation;        // <-- passed to fragment shader

void main() {
  vec3 pos = position;
  pos.y += sin(pos.x * uFrequency + uTime) * uAmplitude;
  vElevation = pos.y;
  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}
```

```glsl
// Fragment shader — check: does it declare the same `varying` name?
varying float vElevation;        // <-- must match vertex shader

void main() {
  vec3 deepBlue = vec3(0.0, 0.1, 0.6);
  vec3 white    = vec3(1.0, 1.0, 1.0);
  vec3 color    = mix(deepBlue, white, vElevation + 0.5);
  gl_FragColor  = vec4(color, 1.0);
}
```

## Hands-On Exercises

1. Use Template 1 from the prompt templates section.
   Paste it into Claude or ChatGPT, run the output in a Vite project, and note any errors.
   Ask the AI to fix each error in the same conversation thread.
2. Take a piece of Three.js code you wrote in a previous day and ask an AI to add three specific visual improvements.
   Review the diff to see what it changed and verify each change against the Three.js docs.
3. Open a Three.js example from [threejs.org/examples](https://threejs.org/examples/) and paste 20 lines of unfamiliar code into Claude.
   Ask it to explain each line.
   Identify one line the explanation gets wrong.
4. Use Template 3 to generate a GLSL wave shader.
   Modify the uniforms to change the wave frequency and amplitude using a dat.GUI or HTML range inputs.
5. Write a comment block at the top of an empty JS file describing a rotating 3D donut (torus) with a sparkle shader.
   Enable GitHub Copilot and observe how far it autocompletes.

## Resources

- [Three.js Migration Guide](https://github.com/mrdoob/three.js/wiki/Migration-Guide) - essential for catching outdated AI suggestions
- [React Three Fiber Docs](https://docs.pmnd.rs/react-three-fiber) - the source of truth for R3F patterns
- [The Book of Shaders](https://thebookofshaders.com/) - learn GLSL deeply enough to evaluate AI shader output
- [Three.js Examples](https://threejs.org/examples/) - working reference code to compare against AI output
- [Shadertoy](https://www.shadertoy.com/) - community GLSL shaders you can paste into prompts as style references

## Reflection Questions

1. What information do you always need to include in a Three.js prompt that AI cannot infer on its own?
2. Describe a scenario where accepting AI-generated 3D code without review could break your project.
3. How does asking an AI to explain its own code help you catch hallucinations?
4. Which of the five prompt templates do you think you will use most often in your own projects, and why?
5. How does iterating in a single conversation thread differ from starting a new chat for each change?

## Next Steps

Tomorrow is the Week 2 Capstone - you will build a complete 3D product configurator with React Three Fiber, bringing together GLB loading, environment lighting, and interactive material switching.

- Previous: [Day 12 - 3D for Web Apps](./day-12-web-app-3d.md)
- Next: [Day 14 - Week 2 Capstone: 3D Product Configurator](./day-14-week2-capstone.md)
