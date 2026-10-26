# Day 19: Blender to Web: Export Pipeline

## Learning Objectives

- Navigate Blender's two core modes (Object Mode and Edit Mode) and understand what each is for.
- Export a 3D model from Blender as a GLB file with correct settings for the web.
- Use the gltf-transform CLI to inspect, optimize, and compress exported files.
- Load a GLB into Three.js using GLTFLoader with Draco decompression support.
- Identify and fix the three most common Blender-to-web pitfalls: axis orientation, scale, and missing textures.

## Core Concepts

### Blender Basics: Two Modes That Matter

Blender has many modes, but two are essential for web export.

**Object Mode** is the default view.
You use it to select, move, rotate, and scale entire objects.
When you want to reposition your model or combine multiple parts, you work in Object Mode.

**Edit Mode** (press Tab to toggle) lets you work with the raw geometry: vertices, edges, and faces.
You use it to fix mesh problems, delete unnecessary geometry, and UV unwrap your model.

Before exporting, two steps in Edit Mode matter most.
First, check that your mesh has no overlapping faces or non-manifold edges.
Second, UV unwrap your model so textures bake correctly.
The quickest unwrap is Smart UV Project (UV menu > Smart UV Project), which works well for most web assets.

### The Modifier to Know Before Exporting: Decimate

AI-generated models often have far more polygons than a web scene needs.
The Decimate modifier (Properties > Modifier > Add Modifier > Decimate) reduces polygon count.
Set the Ratio to 0.3-0.5 and watch the face count drop dramatically.
Always apply modifiers before exporting (right-click the modifier > Apply).
Unapplied modifiers may not export as expected.

### Exporting to GLB from Blender

Go to **File > Export > glTF 2.0**.
Choose the **GLB** format (binary, single file), not GLTF+Bin.
GLB bundles geometry, textures, and materials into one file - ideal for web deployment.

Key settings to configure:

- **Apply Transformations**: Enable this.
  It "bakes in" any pending scale, rotation, or location transforms.
  Without it, a model scaled to 0.01 in Blender will arrive with that scale as a property in Three.js, causing unexpected behavior.
- **Draco Mesh Compression**: Enable this under Data > Mesh.
  Draco is Google's compression algorithm that reduces mesh size by 5-10x.
  You will need the DRACOLoader in Three.js to decode it.
- **Include > Selected Objects**: Enable this to export only what you want.
- **Materials**: Ensure PBR materials export correctly.
  Blender's Principled BSDF maps cleanly to GLTF's PBR metallic-roughness model.

### Common Pitfalls

**Y-up vs Z-up axes**: Blender uses Z as its up axis; GLTF uses Y.
The exporter converts this automatically, but verify your model stands upright after import.
If it is lying on its back, rotate -90 degrees on the X axis in your Three.js/R3F code.

**Scale issues**: Blender works in meters.
If your model looks tiny or giant in Three.js, it is a scale mismatch.
In Object Mode, select all and apply scale with Ctrl+A > Scale before exporting.

**Missing textures**: Textures must be packed into the GLB.
Before exporting, go to File > External Data > Pack Resources.
This embeds all image data into the file.

### Post-Processing with gltf-transform

gltf-transform is a CLI tool for optimizing GLB files after export.
It can further compress, resize textures, and analyze your file.

```bash
# Install globally
npm install -g @gltf-transform/cli

# Inspect file contents and sizes
npx gltf-transform inspect input.glb

# Optimize with Draco compression
npx gltf-transform optimize input.glb output.glb --draco

# Resize embedded textures to 1024px
npx gltf-transform resize --width 1024 --height 1024 input.glb output.glb

# Combine: resize then optimize
npx gltf-transform resize --width 1024 --height 1024 input.glb resized.glb
npx gltf-transform optimize resized.glb final.glb --draco
```

Running `inspect` first tells you which textures are oversized, how many polygons you have, and total file size.
A target size for a single web model is under 2 MB for non-hero assets and under 5 MB for hero assets.

### Loading in Three.js and R3F

Three.js requires the GLTFLoader and - if you used Draco - the DRACOLoader.
R3F's `useGLTF` hook from `@react-three/drei` handles both automatically.

## Key Terminology

| Term | Definition |
|------|-----------|
| GLB | Binary form of GLTF; single file containing geometry, textures, and materials |
| GLTF | GL Transmission Format; open standard for 3D scene and model interchange |
| Draco | Google's open-source mesh compression library; reduces file size 5-10x |
| UV Map | 2D coordinate layout that determines how a texture wraps onto a 3D surface |
| Object Mode | Blender mode for selecting and transforming whole objects |
| Edit Mode | Blender mode for editing mesh geometry at vertex, edge, and face level |
| Decimate | Blender modifier that reduces polygon count while preserving overall shape |
| gltf-transform | CLI and Node.js library for optimizing, compressing, and transforming GLTF files |

## Code Examples

### gltf-transform CLI Pipeline

```bash
# Step 1: inspect what you have
npx gltf-transform inspect scene.glb

# Step 2: downscale any large textures
npx gltf-transform resize --width 1024 --height 1024 scene.glb scene-resized.glb

# Step 3: Draco-compress the mesh
npx gltf-transform optimize scene-resized.glb scene-final.glb --draco

# One-liner for quick optimization (no texture resize)
npx gltf-transform optimize input.glb output.glb --draco
```

### Loading in Three.js with Draco Support

```javascript
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';

const scene = new THREE.Scene();

// Configure DRACOLoader - required for Draco-compressed models
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('https://www.gstatic.com/draco/versioned/decoders/1.5.6/');

const loader = new GLTFLoader();
loader.setDRACOLoader(dracoLoader);

loader.load('/models/character.glb', (gltf) => {
  const model = gltf.scene;

  // If the model appears tiny (Blender centimeter scale mismatch)
  // model.scale.setScalar(0.01);

  scene.add(model);
}, undefined, (error) => {
  console.error('Failed to load model:', error);
});
```

### Loading in React Three Fiber

```javascript
import { useGLTF } from '@react-three/drei';

// Preload at module level for better performance
useGLTF.preload('/models/character.glb');

function Character({ position }) {
  const { scene } = useGLTF('/models/character.glb');

  // Clone the scene so multiple instances work independently
  const cloned = scene.clone(true);

  return <primitive object={cloned} position={position} />;
}
```

### Fix Axis Orientation If Needed

```javascript
// In Three.js - model is lying on its back after Blender export
loader.load('/models/character.glb', (gltf) => {
  const model = gltf.scene;
  model.rotation.x = -Math.PI / 2; // rotate 90 degrees on X axis
  scene.add(model);
});

// In R3F
function Character() {
  const { scene } = useGLTF('/models/character.glb');
  return <primitive object={scene} rotation={[-Math.PI / 2, 0, 0]} />;
}
```

## Hands-On Exercises

1. Open Blender and import any GLB from a previous day (or download a free model from sketchfab.com).
   Switch between Object Mode and Edit Mode using Tab and note what changes.

2. Apply the Decimate modifier at ratio 0.3 and compare the polygon count before and after using the statistics overlay (Viewport Overlays > Statistics).

3. Export your model as a GLB with "Apply Transformations" and "Draco Compression" enabled.
   Check the file size.

4. Run `npx gltf-transform inspect` on your exported GLB.
   Then run `npx gltf-transform optimize input.glb output.glb --draco` and compare file sizes.

5. Load the optimized GLB into a Three.js scene using the code above.
   If the model appears upside-down or sideways, fix it with rotation.

## Resources

- [gltf-transform CLI documentation](https://gltf-transform.dev/cli) - full command reference with all options
- [Blender glTF exporter guide](https://docs.blender.org/manual/en/latest/addons/import_export/scene_gltf2.html) - official Blender docs on GLTF export settings
- [Three.js GLTFLoader examples](https://threejs.org/examples/#webgl_loader_gltf) - live demos with source code
- [Blender Artists community forum](https://blenderartists.org) - ask questions, find tutorials, share work
- [gltf.report](https://gltf.report/) - browser-based GLTF inspector and optimizer

## Reflection Questions

1. Why is it important to "Apply Transformations" in Blender before exporting?
   What goes wrong if you skip this step?

2. Draco compression requires a special decoder in Three.js.
   What is the trade-off between using Draco and not using it?

3. A colleague exports a model from Blender and it appears rotated 90 degrees in their Three.js scene.
   What is the most likely cause and how would you fix it both in Blender and in code?

4. At what point in the pipeline does gltf-transform fit best - before or after verifying the model loads correctly in Three.js?
   Why?

5. What is the difference between GLB and GLTF format, and why does GLB tend to work better for web deployment?

## Next Steps

Tomorrow in Day 20, you will take everything you have learned across all three weeks and put it into a complete end-to-end workflow.
You will see how each tool connects to the next, and learn a decision framework for choosing the right tool at each stage.

[Back to Day 18: Performance Optimization](../week-03/day-18-performance-optimization.md)
