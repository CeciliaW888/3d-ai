# Day 16: AI Game Asset Creation

## Learning Objectives

- Understand the full pipeline from AI prompt to animated in-game character.
- Know the polygon budgets for web games on mobile and desktop targets.
- Use Meshy.ai to generate environment props and characters from text.
- Rig a generated character with Mixamo and export it ready for Three.js.
- Apply the LOD (Level of Detail) concept to keep frame rates high.

## Core Concepts

### The AI Game Asset Pipeline

Creating game-ready 3D assets used to require weeks of skilled 3D artist time.
AI tools now compress that to hours.
The pipeline has five stages, and understanding each one lets you make smart decisions at every step.

**Stage 1 - Generate.**
Start with a text prompt in Meshy.ai or Tripo3D.
Be specific: "low-poly medieval wooden barrel, game asset, single color, smooth topology" works far better than just "barrel".
Generate several variations (Meshy gives you four previews) and pick the cleanest topology.
Clean topology means quads, minimal overlapping geometry, and no floating islands.

**Stage 2 - Cleanup.**
AI models often have extra polygons, disconnected geometry, or messy UVs.
Import the downloaded GLB into Blender (free) and run the Decimate modifier to reduce polygon count to your budget.
Check for inverted normals (faces pointing inward) using Blender's Viewport Overlay > Face Orientation.
Fix any red faces by selecting them and pressing Alt+N > Flip.

**Stage 3 - Optimize.**
Export as a compressed GLB using Draco compression.
Use the `gltf-transform` CLI: `npx @gltf-transform/cli optimize input.glb output.glb --compress draco`.
For textures, aim for 512x512 or 1024x1024 PNG for web; use KTX2 format for GPU-native compression on production builds.

**Stage 4 - Rig.**
For characters that need movement, upload your cleaned mesh to Mixamo (free, no account needed on the web).
Mixamo's Auto-Rigger places a skeleton automatically - just mark the chin, wrists, and elbows in the setup wizard.
Download with the skin (the mesh + skeleton together) as an FBX.

**Stage 5 - Animate.**
Browse Mixamo's motion library: over 2,500 animations including walk, run, jump, and attack cycles.
Download the animation as FBX "without skin" (just the motion data) if you already have the rigged character.
Import both into Blender and use the NLA editor to combine animations.
Export the final file as a single GLB with embedded animations.

### Polygon Budgets for Web Games

Every polygon you render costs GPU time.
Web games run on a huge range of hardware - from high-end gaming laptops to five-year-old phones.
These budgets give you a practical target per object:

| Platform | Triangles per Object | Texture Resolution |
|----------|---------------------|-------------------|
| Mobile web | 2,000 - 5,000 | 256x256 - 512x512 |
| Desktop web | 10,000 - 50,000 | 512x512 - 2048x2048 |
| Hero object (main character) | Up to 15k mobile | As above |

A good rule: count your simultaneous objects.
If 50 enemy characters appear at once on mobile, each should be around 500 triangles, not 5,000.

### Level of Detail (LOD)

LOD is the technique of swapping a high-polygon model for a simpler version when the object is far from the camera.
A character 100 meters away does not need 5,000 triangles - 200 is enough.
Three.js has a built-in `THREE.LOD` object that handles this automatically.

```js
const lod = new THREE.LOD();
lod.addLevel(highPolyMesh, 0);    // show when < 10 units away
lod.addLevel(midPolyMesh,  10);   // show when 10–50 units away
lod.addLevel(lowPolyMesh,  50);   // show beyond 50 units
scene.add(lod);
```

### Free Assets and Kenney.nl

Not every asset needs AI generation.
Kenney.nl (kenney.nl) publishes thousands of CC0 (completely free, no attribution required) game assets: UI kits, character packs, nature scenes, and more.
These assets are already optimized for games and come in GLB or OBJ format.
Use Kenney assets for prototyping, then swap them for AI-generated custom art when the game feel is locked in.

## Key Terminology

| Term | Definition |
|------|-----------|
| GLB | Binary GLTF format - a single file containing mesh, textures, and animations |
| Rigging | Adding a skeleton (bones) to a mesh so it can be posed and animated |
| LOD | Level of Detail - swapping mesh complexity based on camera distance |
| Polygon budget | The maximum triangle count per object for a given platform target |
| Draco compression | Google's algorithm for compressing GLTF geometry, reducing file size 5-10x |
| UV map | A 2D map that tells the renderer how to wrap a flat texture around a 3D surface |
| KTX2 | GPU-native compressed texture format that reduces VRAM usage at runtime |

## Code Examples

### Loading an Animated Character from Mixamo

```js
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');

const loader = new GLTFLoader();
loader.setDRACOLoader(dracoLoader);

let mixer;

loader.load('character.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  // Set up AnimationMixer to play clips
  mixer = new THREE.AnimationMixer(model);

  // Play the first animation (e.g., idle)
  if (gltf.animations.length > 0) {
    const idleAction = mixer.clipAction(gltf.animations[0]);
    idleAction.play();
  }
});

// In your animation loop, advance the mixer
function animate() {
  requestAnimationFrame(animate);
  const delta = clock.getDelta();
  if (mixer) mixer.update(delta);
  renderer.render(scene, camera);
}
```

### LOD Setup with Three.js

```js
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();
const lod = new THREE.LOD();

// Load three versions of the same asset
loader.load('tree-high.glb',  gltf => lod.addLevel(gltf.scene, 0));
loader.load('tree-mid.glb',   gltf => lod.addLevel(gltf.scene, 15));
loader.load('tree-low.glb',   gltf => lod.addLevel(gltf.scene, 40));

scene.add(lod);

// LOD updates automatically in the render loop when you call:
function animate() {
  requestAnimationFrame(animate);
  lod.update(camera);   // switches mesh based on camera distance
  renderer.render(scene, camera);
}
```

## Hands-On Exercises

1. **Generate a game prop.** Open Meshy.ai and generate three variations of "wooden crate, game asset, low poly, worn wood texture".
   Download the best result as a GLB file.
   Open it in an online viewer like `gltf.report` and check the triangle count.

2. **Rig a character.** Generate a humanoid character in Meshy.ai.
   Upload it to Mixamo and run Auto-Rig.
   Apply a "walking" animation and download the result as FBX.

3. **Check your polygon budget.** Open your downloaded FBX in Blender.
   Press `N` in the 3D viewport to open the sidebar and read the triangle count under "Statistics".
   If it exceeds 5,000 for mobile, apply a Decimate modifier (ratio 0.5) and measure again.

4. **Compare Kenney.nl assets.** Browse kenney.nl/assets/3d and download a free environment pack.
   Load one of the GLB files into a Three.js scene alongside your AI-generated prop.
   Note the difference in polygon count and texture size.

5. **Implement LOD.** Take your high-poly model.
   Use Blender's Decimate modifier at ratios 1.0, 0.3, and 0.08 to create three LOD versions.
   Export each as a separate GLB and wire them up with `THREE.LOD` in your scene.

## Resources

- [Three.js LOD documentation](https://threejs.org/docs/#api/en/objects/LOD)
- [Meshy.ai - Text to 3D](https://www.meshy.ai/)
- [Mixamo - Free Auto-Rigging & Animations](https://www.mixamo.com/)
- [Kenney.nl - Free CC0 Game Assets](https://kenney.nl/assets/3d)
- [gltf-transform CLI - Optimization Tool](https://gltf-transform.dev/)

## Reflection Questions

1. Why does mobile web have a stricter polygon budget than desktop web, and what hardware difference drives this?
2. At what point in the pipeline should you apply Draco compression - before or after rigging? Why?
3. How would you decide which objects in a game scene should use LOD versus which ones can stay single-resolution?
4. Kenney.nl assets are free, consistent, and optimized. When would you choose AI generation over free library assets?
5. If Mixamo's auto-rigger produces a skeleton that causes mesh distortion at the shoulders, what is the most likely cause?

## Next Steps

Day 17 takes you into one of the most powerful topics in 3D graphics: GLSL shaders.
You will write your first vertex and fragment shader from scratch, understand how the GPU processes them, and create a live animated wave effect.
Shaders unlock visual effects that no mesh manipulation alone can achieve - and AI tools can help you write them.

- Previous: [Day 15 - Browser 3D Games with Three.js](day-15-browser-3d-games.md)
- Next: [Day 17 - GLSL Shaders: Visual Magic](day-17-glsl-shaders.md)
