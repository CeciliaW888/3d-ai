# Day 10: AI Animation: Mixamo & Beyond

## Learning Objectives

- Explain what skeletal animation is and identify its components: armature, bones, skinning, and keyframes.
- Complete the full Mixamo workflow: upload a mesh, auto-rig it, choose an animation, and download a GLB.
- Use Three.js `AnimationMixer` to load and play animations from a Mixamo GLB.
- Blend between two animation clips using `crossFadeTo`.
- Name at least two AI-powered motion generation tools beyond Mixamo.

## Core Concepts

### What Is Skeletal Animation?

When you watch a game character walk, the mesh - the visible skin made of triangles - doesn't move vertex by vertex.
Instead, it is driven by a hidden skeleton called an **armature**.

An armature is a hierarchy of **bones**.
Each bone has a position, a rotation, and a parent-child relationship to the bones around it (the spine bone is the parent of the rib bones, for instance).
When a bone rotates, all its children rotate with it - exactly like a real joint.

**Skinning** is the process of binding the mesh to the armature.
Each vertex in the mesh is assigned a weight that says how much each nearby bone influences it.
A vertex at the elbow might be 60% influenced by the forearm bone and 40% by the upper arm bone, so it deforms smoothly as the elbow bends.

A **keyframe** records the state of every bone at a specific point in time.
The animation system interpolates - smoothly calculates - bone positions between keyframes.
A walk cycle might have 30 keyframes over half a second; the renderer fills in every frame between them.

### The Mixamo Workflow

Mixamo (mixamo.com) is a free Adobe service that automates the hardest part of character animation: rigging.
The workflow has four steps.

**Step 1 - Upload your mesh.**
Mixamo accepts FBX and OBJ files.
The mesh should be a humanoid character in a neutral T-pose (arms out, feet together).
Meshy.ai and Tripo3D can both export FBX, making them natural upstream tools.

**Step 2 - Auto-rig.**
Mixamo uses AI to detect the character's body landmarks - head top, chin, wrists, groin, and knees.
You place five markers in the UI, click "Next," and Mixamo generates a full humanoid rig with roughly 65 bones in seconds.

**Step 3 - Pick an animation.**
Mixamo has thousands of motion-captured animations: walks, runs, jumps, dances, combat moves, and idle cycles.
Use the search bar.
You can preview any animation on your character in real time.
Adjust "Overdrive," "Strength," and "Arm Space" sliders to fine-tune how the animation maps onto your specific mesh proportions.

**Step 4 - Download.**
Choose "FBX for Unity" (which works well with Three.js) or the plain GLB format.
Check "Without Skin" if you want only the animation data to apply to a mesh you already have in Three.js.

### Three.js AnimationMixer

Three.js handles skeletal animation through three classes.

`AnimationClip` is a named set of keyframe tracks - one track per bone property (position X, rotation Y, etc.).
When you load a Mixamo GLB with `GLTFLoader`, the `animations` array on the result contains these clips.

`AnimationMixer` is the playback engine attached to a specific object.
You create one per animated character: `const mixer = new THREE.AnimationMixer(model)`.
On every frame you call `mixer.update(delta)` to advance time.

`AnimationAction` is the handle for playing a single clip.
You get one via `mixer.clipAction(clip)`, then call `.play()` on it.

To transition between two animations, use `crossFadeTo`:

```js
currentAction.crossFadeTo(nextAction, 0.5, false);
```

The second argument is the blend duration in seconds.
During the blend, both animations play simultaneously with their weights interpolating.

### AI Motion Generation Beyond Mixamo

Mixamo is ideal for humanoid characters, but the field is expanding fast.

**Cascadeur** (cascadeur.com) uses physics-based AI to generate and edit full-body motion for any character shape, not just humanoids.
It predicts secondary motion like hair, clothing, and tail sway automatically.

**Move.ai** captures motion from regular video footage using AI pose estimation - no motion-capture suit needed.
You film someone walking with a phone camera and get a BVH/FBX skeleton animation in return.

Both tools export formats Three.js can consume, so the AnimationMixer workflow applies to their output too.

## Key Terminology

| Term | Definition |
|------|-----------|
| Armature | The invisible skeleton hierarchy of bones that drives mesh deformation. |
| Skinning | The process of assigning bone influence weights to each mesh vertex. |
| Keyframe | A recorded snapshot of bone transforms at a specific point in time. |
| AnimationMixer | Three.js class that advances and blends animation clips for a scene object. |
| AnimationClip | A named set of keyframe tracks representing one animation (e.g., "Walk"). |
| AnimationAction | A handle for playing or blending a specific AnimationClip on a mixer. |
| crossFadeTo | An AnimationMixer method that smoothly transitions weight from one action to another. |
| T-pose | The standard neutral position for rigging: arms out horizontally, feet together. |

## Code Examples

### Loading and Playing a Mixamo GLB

```js
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 100);
camera.position.set(0, 1.5, 3);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 0.6));
const dirLight = new THREE.DirectionalLight(0xffffff, 1);
dirLight.position.set(5, 10, 5);
scene.add(dirLight);

let mixer;
let idleAction;
let walkAction;

const loader = new GLTFLoader();
loader.load('/models/character-with-animations.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  // Create the mixer bound to this model
  mixer = new THREE.AnimationMixer(model);

  // gltf.animations is an array of AnimationClip objects
  // Mixamo names clips after the animation: "Idle", "Walking", etc.
  const idleClip = THREE.AnimationClip.findByName(gltf.animations, 'Idle');
  const walkClip = THREE.AnimationClip.findByName(gltf.animations, 'Walking');

  idleAction = mixer.clipAction(idleClip);
  walkAction = mixer.clipAction(walkClip);

  // Start in idle
  idleAction.play();
});

// Transition to walk on keypress
document.addEventListener('keydown', (e) => {
  if (e.key === 'w' && mixer) {
    idleAction.crossFadeTo(walkAction, 0.5, false);
    walkAction.play();
  }
});

document.addEventListener('keyup', (e) => {
  if (e.key === 'w' && mixer) {
    walkAction.crossFadeTo(idleAction, 0.5, false);
    idleAction.play();
  }
});

const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta(); // seconds since last frame

  // CRITICAL: always call mixer.update with delta time, not total time
  if (mixer) mixer.update(delta);

  renderer.render(scene, camera);
}

animate();
```

### Getting the Delta Time Correctly

```js
// BAD: passing total elapsed time breaks animation speed
const elapsed = clock.getElapsedTime();
mixer.update(elapsed); // animations will run faster and faster

// GOOD: pass the time since the last frame
const delta = clock.getDelta();
mixer.update(delta); // consistent playback speed
```

## Hands-On Exercises

1. **Rig a character** - Take any humanoid mesh from Meshy.ai (from Day 3 or Day 8) and run it through the Mixamo auto-rigger.
   Place the five body markers carefully and observe the rig quality.
2. **Download two animations** - Download an "Idle" and a "Running" animation as separate GLB files, then as a single GLB with both included.
3. **Play an animation** - Use the code example above with your downloaded Mixamo GLB.
   Confirm the character animates in the browser.
4. **Trigger a transition** - Wire up a keyboard key to `crossFadeTo` between idle and running.
   Experiment with blend durations from 0.1 to 1.0 seconds and note how they feel.
5. **Explore AI motion tools** - Visit the Cascadeur website and watch one demo video.
   Note one thing it can do that Mixamo cannot.

## Resources

- [Three.js AnimationMixer docs](https://threejs.org/docs/#api/en/animation/AnimationMixer)
- [Three.js animation system overview](https://threejs.org/docs/#manual/en/introduction/Animation-system)
- [Mixamo](https://www.mixamo.com/)
- [GLTFLoader docs](https://threejs.org/docs/#examples/en/loaders/GLTFLoader)
- [Cascadeur AI animation tool](https://cascadeur.com/)

## Reflection Questions

1. Why is `clock.getDelta()` used instead of `clock.getElapsedTime()` when calling `mixer.update()`?
2. What does "skinning" mean, and what would a character look like if the skin weights were all set to the same bone?
3. If you download a Mixamo animation "Without Skin," what data is in the file and what is missing?
4. What advantage does Move.ai have over Mixamo for capturing a real person's movement?
5. A character has an "Idle" action playing and you want to smoothly switch to "Jump."
   What method do you call, and on which object?

## Next Steps

Tomorrow on Day 11 you will learn how to tie 3D scenes to the browser scroll position, creating the kind of immersive scroll-driven experiences you see on Apple product pages.

- Previous: [Day 9 - AI Texture Generation](./day-09-ai-textures.md)
- Next: [Day 11 - 3D for Websites: Scroll & Parallax](./day-11-3d-for-websites.md)
