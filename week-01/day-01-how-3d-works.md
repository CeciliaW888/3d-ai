# Day 1: How 3D Works - Meshes, Materials & Lighting

## Learning Objectives

- Identify the fundamental components of a 3D mesh: vertices, edges, and faces
- Explain how UV mapping connects 2D textures to 3D surfaces
- Describe the four key properties of a PBR material model
- Distinguish the four primary light types used in real-time 3D scenes
- Compare perspective and orthographic cameras and their appropriate uses

## Core Concepts

### The Building Blocks: Vertices, Edges, and Faces

Every 3D object you see - a car model on a product page, a game character, a spinning logo - is an illusion built from triangles.
At the lowest level, a 3D model is just a collection of points in space called **vertices**.
Connect two vertices and you get an **edge**.
Enclose an area with three or more vertices and you get a **face** (also called a polygon).
Combine enough faces and you have a **mesh** - the skeleton structure that defines any 3D shape.

When your GPU renders a 3D scene, it ultimately reduces everything to triangles.
Quads (4-sided faces) are convenient for artists to model with, but they get triangulated at render time.
A game character might use 5,000-15,000 triangles; a film character might use millions.
For the web, polygon count matters directly for performance - you will learn to balance quality against budget.

### UV Mapping: Wrapping 2D Images onto 3D Surfaces

Imagine peeling an orange - you are unfolding its 3D surface into a flat 2D shape.
UV mapping is the reverse: defining how a flat 2D image (a texture) wraps around your 3D mesh.
The letters "U" and "V" are the 2D axis names for texture space (we reserve X, Y, Z for the 3D world).
Every vertex on your mesh has a corresponding UV coordinate that tells the renderer: "this point on the mesh should display this pixel on the texture."

Poor UV mapping leads to textures that look stretched, squished, or misaligned.
Good UV mapping is one of the most important and underappreciated skills in 3D art.
AI-generated models usually include UV maps automatically - but they are often imperfect, which is why a cleanup pass matters.

### PBR Materials: Making Surfaces Look Real

**Physically Based Rendering (PBR)** is the material model used in virtually all modern 3D - games, film, AR, and web.
Instead of faking how light interacts with surfaces, PBR simulates it using physically accurate equations.
This means a PBR material looks correct under any lighting condition.

The four core PBR properties are:

**Albedo (Base Color)** - The raw color of the surface, with no lighting baked in.
Think of it as the color the object would be in pure white light, like a paint chip.
This is the texture you paint; it contains no shadows or highlights.

**Roughness** - How microscopically bumpy the surface is, from 0.0 (mirror-perfect) to 1.0 (completely matte).
A smooth surface creates tight, sharp reflections.
A rough surface scatters light in all directions, creating soft, blurry reflections.

**Metalness** - Whether the surface behaves like a metal (1.0) or a non-metal (0.0).
Metals reflect colored light - gold reflects yellow, copper reflects orange.
Non-metals always reflect white highlights regardless of their albedo color.

**Normal Map** - A texture that fakes small-scale bumps and surface detail without adding any geometry.
The RGB channels encode the X, Y, Z direction of the surface normal at each point.
This tricks the shading math into thinking there are bumps that are not geometrically there.
Normal maps are why a flat polygon can look like rough stone or embossed leather.

### Lighting: The Invisible Sculptor

A 3D scene without light is completely invisible.
Lighting is also the single biggest factor in how realistic or stylized a scene feels - more so than the geometry or textures alone.

The four main light types in real-time 3D:

**Ambient Light** - Adds a flat, uniform base brightness to everything in the scene.
It has no direction and casts no shadows.
Its job is to prevent the unlit side of objects from going completely black.

**Directional Light** - Simulates a distant, infinitely large light source like the sun.
All rays are perfectly parallel; there is no falloff with distance.
Everything in the scene receives equal illumination from it.

**Point Light** - Emits light in all directions from a single point, like a bare light bulb.
Brightness falls off with distance, creating natural-feeling pools of light.

**Spot Light** - Like a point light constrained to a cone, like a flashlight or a stage spotlight.
Useful for focused dramatic lighting and performance-conscious scenes.

### Cameras: Your Window Into the Scene

A **perspective camera** simulates how human eyes and camera lenses work: objects farther away appear smaller.
This is the camera you will use for almost everything - it creates the natural sense of depth that feels right.

An **orthographic camera** renders objects at the same size regardless of distance, with no perspective distortion.
It is used for 2D games, isometric views, technical diagrams, and certain stylized flat aesthetics.

### The Render Pipeline (Simplified)

When a frame is rendered, the GPU runs these steps in sequence:

1. Transform all vertices from 3D world space into 2D screen positions.
2. Rasterize triangles into individual pixels.
3. Run the **fragment shader** - the program that computes the final color of each pixel using material properties, texture lookups, and light calculations.
4. Apply post-processing effects like bloom, depth of field, or tone mapping.

Understanding this pipeline explains why certain visual problems happen - a texture stretching, a shadow artifact, a material looking wrong - and points you toward the right fix.

## Key Terminology

| Term | Definition |
|------|-----------|
| Vertex | A single point in 3D space, defined by X, Y, Z coordinates |
| Edge | A line segment connecting two vertices |
| Face | A flat surface enclosed by edges - the basic unit of mesh geometry |
| Mesh | A complete 3D structure made of connected vertices, edges, and faces |
| UV Map | A 2D unwrapping of a mesh surface used to align 2D textures onto 3D geometry |
| PBR | Physically Based Rendering - a material model that simulates real-world light physics |
| Albedo | The base color of a surface, independent of any lighting or shadows |
| Normal Map | A texture encoding fake surface bumps as surface direction vectors in RGB channels |
| Roughness | How microscopically bumpy a surface is - controls sharpness of reflections |
| Fragment Shader | A GPU program that computes the final pixel color using materials, textures, and lights |

## Code Examples

```javascript
// Three.js: Create a sphere with a PBR material, directional light, and ambient light
import * as THREE from 'three';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100);
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Geometry defines the mesh shape
const geometry = new THREE.SphereGeometry(1, 32, 32);

// MeshStandardMaterial is Three.js's PBR material
const material = new THREE.MeshStandardMaterial({
  color: 0x4488ff,   // Albedo - base blue color
  roughness: 0.4,    // Moderate roughness - soft but visible reflections
  metalness: 0.1,    // Slightly metallic
});

const sphere = new THREE.Mesh(geometry, material);
scene.add(sphere);

// Directional light - simulates sunlight
const sunLight = new THREE.DirectionalLight(0xffffff, 1.0);
sunLight.position.set(5, 10, 5);
scene.add(sunLight);

// Ambient light - prevents pure black shadows
const ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
scene.add(ambientLight);

camera.position.z = 3;

function animate() {
  requestAnimationFrame(animate);
  sphere.rotation.y += 0.005;
  renderer.render(scene, camera);
}
animate();
```

Try changing `roughness` between 0 and 1 and observe the reflection quality shift from mirror-sharp to fully diffuse.
Try `metalness: 1.0` and watch how the blue albedo starts tinting the reflections.

## Hands-On Exercises

1. **Observe PBR in action**: Visit [threejs.org](https://threejs.org) and open one of the official examples featuring `MeshStandardMaterial`.
   Open the browser console and run `scene.traverse(obj => { if (obj.material) { obj.material.roughness = 0; } })` to set all roughness to 0 and watch the materials shift to mirror-like.
2. **Explore a mesh wireframe**: Go to [sketchfab.com](https://sketchfab.com), find any free model, and switch to Wireframe view mode.
   Count roughly how many visible triangles make up one face of the model.
3. **Mental PBR exercise**: Pick any object near you right now.
   Try to assign it an albedo color, a roughness value (0-1), and a metalness value (0 or 1) as if you were recreating it as a 3D material.
4. **Run the code example**: Create a plain HTML file, add Three.js via CDN import map, paste the code example, and open it in a browser.
   Modify `roughness` and `metalness` and reload to see the effect.
5. **Light comparison**: In your test scene, remove the `DirectionalLight` and observe the sphere with only ambient light.
   Then reverse - remove ambient and keep only directional - and observe the result.

## Resources

- [Three.js Documentation - MeshStandardMaterial](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial)
- [Three.js Examples Gallery](https://threejs.org/examples/)
- [Blender Manual - Materials Introduction](https://docs.blender.org/manual/en/latest/render/materials/introduction.html)
- [Khronos Art Pipeline for GLTF](https://www.khronos.org/blog/art-pipeline-for-gltf)
- [Sketchfab - Free 3D Models](https://sketchfab.com/features/free-3d-models)

## Reflection Questions

1. Why does a 3D object still look flat when it has good geometry and textures but no directional light?
2. What would happen visually if you set `metalness: 1.0` and `roughness: 0.0` on a bright red material?
3. How does a normal map "fake" geometric detail without adding any actual vertices to the mesh?
4. When would you choose an orthographic camera over a perspective camera for a web project?
5. Why does polygon count matter more for web 3D than for film visual effects?

## Next Steps

Tomorrow you will map the full 3D software landscape - from Blender to Three.js to AI generation tools - and understand which tool belongs at each stage of the web 3D pipeline.

[Next: The 3D Software Landscape →](./day-02-3d-software-landscape.md)
