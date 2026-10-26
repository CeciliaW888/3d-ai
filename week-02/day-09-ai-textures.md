# Day 9: AI Texture Generation

## Learning Objectives
- Explain what a PBR texture set is and what each map contributes to the final look.
- Find and download free PBR texture sets from Polyhaven.
- Load a multi-map texture set into a Three.js `MeshStandardMaterial`.
- Use AI tools to generate custom textures for surfaces that don't exist in texture libraries.
- Apply a generated texture set to a basic Three.js mesh and tune the material properties.

## Core Concepts

### What Are Textures?

A texture is a 2D image that gets "wrapped" around a 3D mesh to give it surface detail.
Without textures, a 3D mesh is a flat-shaded blob.
With textures, a plastic bottle looks plastic, stone looks rocky, and metal looks metallic.

In modern 3D rendering, you almost never use just one texture image.
You use a **PBR (Physically Based Rendering) texture set** - a collection of maps where each map encodes a different physical property of the surface.

### The PBR Texture Maps

**Albedo / Diffuse (Color Map)**
This is the base color of the surface with no lighting information baked in.
Avoid baking highlights or shadows into this map - lighting is computed at runtime.

**Normal Map**
A normal map encodes surface micro-detail (bumps, scratches, grout lines) as colors.
The red channel stores X deviation, the green channel stores Y, and blue stores Z.
Normal maps create the illusion of geometry without adding polygons - they are the single biggest win for visual fidelity at zero performance cost.

**Roughness Map**
Controls how rough or smooth the surface is at each pixel.
White pixels = rough (diffuse, matte), black pixels = smooth (specular, mirror-like).

**Metalness Map**
Controls whether a surface behaves like a metal or a dielectric (non-metal).
White = fully metallic (gold, steel), black = non-metallic (plastic, wood).
Together with roughness, metalness forms the core of the PBR model.

**Ambient Occlusion Map (AO)**
Encodes how much ambient light reaches each part of the surface.
Dark areas in the AO map represent crevices and contact points where light gets blocked.
This map fakes global illumination and makes geometry look grounded.

**Height / Displacement Map**
Unlike a normal map, a displacement map actually moves vertices based on pixel brightness.
This is expensive (requires a dense mesh) and is often avoided for real-time web use.

### Free PBR Textures: Polyhaven

Polyhaven (polyhaven.com) is a library of CC0 (public domain) PBR textures, HDR environment maps, and 3D models.
Every asset is free for commercial use with no attribution required.
You can download textures at 1K, 2K, 4K, or 8K resolution.
For web use, 1K (1024x1024) or 2K (2048x2048) is sufficient.

### AI Texture Generation Tools

When a texture you need does not exist in any library, AI generation is the solution.

**Stable Diffusion (tiling mode)**
Using a ControlNet tile model or the "seamless tiling" setting in tools like InvokeAI, AUTOMATIC1111, or Comfy UI, you can generate seamless tileable textures.
Prompt: "seamless rust metal texture, 4k, tileable, pbr" produces a color base.
You then run a separate Normal Map generator pass to produce the other maps.

**Adobe Substance (AI-powered)**
Substance 3D Sampler lets you photograph a material and AI-converts it to a full PBR set.
This is the professional workflow - it produces correctly separated maps from a single reference photo.

**Materialize (free)**
Materialize is a free Windows/Mac application by Bounding Box Software.
You feed it a diffuse/color image and it automatically derives the normal, metalness, roughness, and AO maps using image processing algorithms.
Not full AI, but very useful for free.

**PolyPizza / Polyhaven Materials**
Both sites offer pre-made PBR sets ready to download.
Polyhaven is the higher-quality option.

### Applying PBR Textures in Three.js

Three.js's `MeshStandardMaterial` is the PBR-ready material class.
It accepts all the standard map slots.
The `TextureLoader` class loads image files asynchronously.
For multiple textures from the same source, use a single loader instance and call `.load()` multiple times.

Important: set `colorSpace = THREE.SRGBColorSpace` on the albedo texture only.
All other maps (normal, roughness, metalness, AO) encode data values, not color, so they must stay in linear space.

## Key Terminology

| Term | Definition |
|------|-----------|
| PBR | Physically Based Rendering - a shading model that simulates real-world light interaction |
| Albedo | The base color of a surface, independent of lighting |
| Normal map | An RGB image encoding surface micro-bump directions to simulate fine detail without polygons |
| Roughness map | A grayscale map where white = rough/matte and black = smooth/specular |
| Metalness map | A grayscale map where white = metallic behavior and black = dielectric behavior |
| AO map | Ambient Occlusion map - encodes contact shadows and crevice darkening |
| Tiling | Repeating a texture multiple times across a surface using UV coordinates |
| UV mapping | The process of projecting a 3D mesh's surface onto a 2D coordinate space for texture placement |

## Code Examples

### Loading a PBR Texture Set and Applying to a Mesh

```js
import * as THREE from 'three';

// --- Scene, Camera, Renderer (standard setup) ---
const scene    = new THREE.Scene();
const camera   = new THREE.PerspectiveCamera(45, innerWidth / innerHeight, 0.1, 100);
camera.position.set(0, 1, 4);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.outputColorSpace = THREE.SRGBColorSpace;
document.body.appendChild(renderer.domElement);

// --- Load a PBR texture set (Polyhaven example: "brick_wall") ---
// Download these from https://polyhaven.com/a/brick_wall at 1K resolution
const loader = new THREE.TextureLoader();

function loadTex(path) {
  return loader.load(path);
}

// Albedo needs sRGB color space; all other maps stay in linear (default)
const albedoMap    = loadTex('./textures/brick_albedo_1k.jpg');
albedoMap.colorSpace = THREE.SRGBColorSpace;

const normalMap    = loadTex('./textures/brick_normal_1k.jpg');
const roughnessMap = loadTex('./textures/brick_roughness_1k.jpg');
const aoMap        = loadTex('./textures/brick_ao_1k.jpg');

// Set UV tiling (repeat) for all maps
[albedoMap, normalMap, roughnessMap, aoMap].forEach(t => {
  t.wrapS = t.wrapT = THREE.RepeatWrapping;
  t.repeat.set(2, 2); // tile 2x in each direction
});

// --- Create material ---
const material = new THREE.MeshStandardMaterial({
  map:          albedoMap,
  normalMap:    normalMap,
  normalScale:  new THREE.Vector2(1, 1), // increase for stronger bump effect
  roughnessMap: roughnessMap,
  roughness:    1.0,  // multiplier: 1.0 uses the map fully
  aoMap:        aoMap,
  aoMapIntensity: 1.0
});

// --- Geometry ---
// A sphere shows texture wrapping and lighting well
const geo  = new THREE.SphereGeometry(1, 64, 64);
const mesh = new THREE.Mesh(geo, material);
scene.add(mesh);

// AO map requires a second UV set on the geometry
geo.setAttribute('uv2', geo.attributes.uv);

// --- Lighting ---
scene.add(new THREE.AmbientLight(0xffffff, 0.3));
const dirLight = new THREE.DirectionalLight(0xffffff, 2);
dirLight.position.set(5, 8, 5);
scene.add(dirLight);

// --- Animation loop ---
function animate() {
  requestAnimationFrame(animate);
  mesh.rotation.y += 0.005;
  renderer.render(scene, camera);
}
animate();

window.addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
```

## Hands-On Exercises

1. Visit polyhaven.com and download any PBR material at 1K resolution.
You will get a zip with multiple map files.
Apply the albedo, normal, and roughness maps to a `SphereGeometry` using the code above.

2. Change the `normalScale` value from `(1, 1)` to `(3, 3)`.
Observe the exaggerated bump effect.
Then try `(0.2, 0.2)` for a subtle effect.
This gives you intuition for how normal maps contribute to surface feel.

3. Set `roughness: 0.0` on the `MeshStandardMaterial` (ignoring the roughness map).
Compare this fully smooth, mirror-like result to `roughness: 1.0`.
Now set `metalness: 1.0` and watch it become a metallic mirror.

4. Download a second PBR texture set from Polyhaven (for example a wood or concrete texture).
Apply it to a `BoxGeometry` next to your sphere.
Use different `repeat` values to see how UV tiling affects the scale of the pattern.

5. Open Materialize (or use an online normal map generator like normalmap.online).
Take any photograph of a flat surface (a table, a book cover, a concrete wall).
Generate a normal map from it.
Apply the original photo as the albedo and the generated normal map to a Three.js mesh.

## Resources

- [Polyhaven - Free PBR Textures (CC0)](https://polyhaven.com/)
- [Three.js MeshStandardMaterial docs](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial)
- [Three.js TextureLoader docs](https://threejs.org/docs/#api/en/loaders/TextureLoader)
- [Normal map online generator](https://normalmap.online/)
- [Sketchfab 3D models with PBR materials](https://sketchfab.com/)

## Reflection Questions

1. Why should you set `colorSpace = THREE.SRGBColorSpace` on the albedo texture but not on the normal or roughness maps?

2. A texture artist gives you a displacement map for a stone wall.
Why would you prefer a normal map over a displacement map for a real-time web scene?

3. Your PBR sphere looks flat even though you loaded all five maps.
What is one common mistake that could cause this?

4. You generate a seamless texture with Stable Diffusion and it tiles well, but the surface looks uniformly bright with no depth variation.
Which PBR map would you add first to fix this?

5. Describe a scenario where buying a Substance subscription would be worth it over using free tools like Polyhaven and Materialize.

## Next Steps

- Day 10: AI animation - learn to use Mixamo to rig characters and export animated GLBs for Three.js
- Day 11: 3D for websites - scroll-driven animations and parallax with Three.js
