# Day 8: Image-to-3D AI Tools

## Learning Objectives
- Distinguish between NeRF, photogrammetry, and diffusion-based image-to-3D techniques.
- Identify the strengths and limitations of each approach for web use.
- Use Meshy.ai's image-to-3D workflow to turn a product photo into a GLB file.
- Evaluate a generated model's polygon count and texture quality for web readiness.
- Describe a basic cleanup workflow for models that need manual polish.

## Core Concepts

### How Image-to-3D Works

Three fundamentally different technologies compete in this space, and each has a distinct set of trade-offs.

**Photogrammetry**
Photogrammetry reconstructs 3D geometry from dozens or hundreds of overlapping photographs.
Software like Polycam (mobile), Reality Capture, or Metashape finds matching feature points across images and triangulates their 3D positions.
The result is high-fidelity geometry with real-world textures - great for objects that actually exist.
The limitation is that you need many photos from different angles and the output is often dense (millions of polygons) and needs decimation before web use.

**NeRF (Neural Radiance Field)**
NeRF is a machine-learning technique that encodes a 3D scene as a neural network.
Given a set of posed photographs, the network learns to predict the color at any point in 3D space from any viewing direction.
Luma AI popularized NeRF-based capture: you walk around an object with your phone and Luma generates a photorealistic volumetric representation.
The trade-off is that NeRFs do not produce a traditional polygon mesh - they produce a "radiance field" that must be baked or converted to a mesh (a step Luma and similar tools automate, but imperfectly).

**Diffusion-based (AI generative)**
Tools like Meshy.ai's image-to-3D, CSM (Common Sense Machines), and Stable Zero123 use image diffusion models to hallucinate the missing views of an object and then reconstruct geometry from them.
You supply one or a few images and the AI infers what the back and sides look like.
This is the fastest path to a web-ready GLB - often under two minutes - but the back of the model is an educated guess, not a scan.
For products like shoes, bottles, or electronic devices that have well-defined symmetry, the results are good.
For highly irregular or transparent objects, quality drops.

### Practical Tool Comparison

| Tool | Technique | Input | Typical Output | Free Tier |
|------|-----------|-------|----------------|-----------|
| Meshy.ai image-to-3D | Diffusion | 1-4 photos | GLB, 10-50k triangles | Yes (limited) |
| Luma AI | NeRF | ~30+ photos or video | Mesh + textures | Yes (limited) |
| Polycam | Photogrammetry | 30+ photos (mobile) | OBJ/GLB | Yes (low res) |
| CSM (Common Sense Machines) | Diffusion | 1-4 photos | GLB | Yes (limited) |

### Limitations to Know

**Polygon count** - AI-generated models from diffusion tools tend to produce 10,000 to 80,000 triangles.
For a single hero object this is acceptable, but loading ten such models on one page will hurt performance.
Use gltf-transform or Blender's Decimate modifier to reduce if needed.

**Texture quality** - The AI-generated texture is often 1024x1024 or 2048x2048.
Diffusion models can produce slightly blurry or hallucinated textures on the back of objects.
For close-up product shots you may need to manually touch up textures in Photoshop or Substance Painter.

**No internal geometry** - Diffusion models reconstruct the surface only.
If your product has internal parts visible through glass or holes, the model will not capture them correctly.

**Cleanup workflow**
1. Download the GLB from the tool.
2. Open in gltf.report (browser-based) to check polygon count and file size.
3. If over 5 MB or 100k triangles, drag the file into Blender, apply the Decimate modifier, and re-export.
4. Check for inverted normals (dark splotches) and fix them in Blender's Edit Mode.
5. Re-export as GLB with Draco compression enabled in gltf-transform for maximum compression.

### Practical: Turn a Product Photo into a GLB

1. Take or find a clean product photo against a white or neutral background.
2. Go to meshy.ai and choose "Image to 3D".
3. Upload the image and choose your target style (realistic or stylized).
4. Wait for generation (typically 1-3 minutes).
5. Preview the result in Meshy's viewer - check all angles.
6. Download the GLB file.
7. Drop it into the Day 7 hero section by replacing `MODEL_URL`.

## Key Terminology

| Term | Definition |
|------|-----------|
| NeRF | Neural Radiance Field - a neural network that encodes a 3D scene learned from photos |
| Photogrammetry | Computing 3D geometry from overlapping 2D photographs using feature matching |
| Diffusion model | Generative AI that reconstructs unseen views by denoising latent representations |
| Decimate | Reducing polygon count while preserving shape - a mesh simplification operation |
| Draco compression | Google's geometry compression algorithm for GLTF/GLB files, reduces file size ~60-80% |
| Inverted normals | Polygon faces pointing inward rather than outward - appear as dark holes or invisible surfaces |
| Baking | Rendering lighting, detail, or texture information from a high-res mesh onto a low-res version |

## Code Examples

### Loading and Centering Any GLB Output

```js
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

// Set up DRACO decoder for compressed GLBs from AI tools
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/libs/draco/');

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

// Helper: load a GLB, center it, and return the scene group
function loadGLBCentered(url, scene, onReady) {
  gltfLoader.load(url, (gltf) => {
    const root = gltf.scene;

    // Compute bounding box and center
    const box    = new THREE.Box3().setFromObject(root);
    const center = box.getCenter(new THREE.Vector3());
    const size   = box.getSize(new THREE.Vector3());
    const maxDim = Math.max(size.x, size.y, size.z);

    // Scale to fit in a 2-unit cube, centered at origin
    root.scale.setScalar(2 / maxDim);
    root.position.sub(center.multiplyScalar(2 / maxDim));

    scene.add(root);
    if (onReady) onReady(root);
  },
  undefined,
  (err) => console.error('Load failed:', err));
}

// Usage
loadGLBCentered('./my-product.glb', scene, (model) => {
  console.log('Model loaded, polygon count:',
    model.children.reduce((n, c) =>
      n + (c.isMesh && c.geometry ? c.geometry.index.count / 3 : 0), 0)
  );
});
```

## Hands-On Exercises

1. Take a photo of any everyday object (a coffee mug, a shoe, a small toy) against a light background.
Upload it to meshy.ai and generate a 3D model.
Download the GLB and embed it in the Day 7 hero section.

2. Open the downloaded GLB at gltf.report in your browser.
Note the polygon count, texture size, and file size.
Compare these to the guidelines: under 100k triangles, under 3 MB for a web hero.

3. Try generating the same product image in both Meshy.ai and Luma AI.
Compare the quality of the resulting geometry on the back and sides of the object.
Write down two differences you observe.

4. Find a free 3D scan on Sketchfab (search "photogrammetry").
Download the GLB or OBJ and note the polygon count.
How does it compare to the AI-generated versions you created?

5. Research Draco compression: run `npx gltf-pipeline -i input.glb -o output.glb -d` in a terminal (requires Node.js).
Compare the before and after file sizes.

## Resources

- [Meshy.ai - AI image-to-3D workflow](https://www.meshy.ai/)
- [Luma AI NeRF capture](https://lumalabs.ai/)
- [gltf.report - browser GLB analyzer](https://gltf.report/)
- [Sketchfab 3D model library](https://sketchfab.com/)
- [glTF-Transform CLI documentation](https://gltf-transform.dev/)

## Reflection Questions

1. You need a 3D model of a crystal perfume bottle for a web hero section.
The bottle is transparent and asymmetric.
Which image-to-3D technique would you choose and why?

2. A model generated by Meshy.ai has 250,000 triangles.
What are two strategies for reducing this count before web deployment?

3. NeRF-based tools like Luma AI produce photorealistic results.
Why might you still prefer a diffusion-based tool like Meshy for a product landing page?

4. What is the trade-off between a higher-resolution texture (4096x4096) and a lower-resolution one (1024x1024) on a web 3D model?

## Next Steps

- Day 9: AI texture generation - learn what PBR texture sets are and how to generate and apply them in Three.js
- Day 10: AI animation with Mixamo - bring your static 3D characters to life
