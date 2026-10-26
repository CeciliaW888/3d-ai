# Day 6: Spline: No-Code 3D for Web

## Learning Objectives
- Navigate the Spline editor and understand its core objects, materials, and camera system
- Build an interactive 3D scene using Spline's built-in physics and event system
- Embed a Spline scene in a plain HTML page using the Spline Viewer web component
- Integrate a Spline scene into a React app using `@splinetool/react-spline`
- Apply web-performance best practices: polygon budget, texture sizing, and export settings

## Core Concepts

### What is Spline?

Spline is a browser-based 3D design tool built for web developers and designers.
Unlike Blender or Cinema 4D which target film and games, Spline is designed from the ground up to output interactive 3D scenes that embed directly into websites.

The workflow is: design in the Spline editor, click Export, paste a snippet into your HTML or React app.
No Three.js, no model files, no build configuration needed.
For hero sections, product showcases, and interactive UI elements, this is often the fastest path to shipping 3D on the web.

### The Spline Editor

When you open a Spline project you see:

**Viewport** - the 3D canvas in the center.
You orbit with alt+drag (or option+drag on Mac) and zoom with the scroll wheel.

**Outliner** - the layer panel on the left, showing all objects in the scene as a tree.
Click any object to select it.

**Inspector** - the properties panel on the right.
Shows transform (position, rotation, scale), material settings, and physics options for the selected object.

**Objects** - you add primitives (cube, sphere, cylinder, torus, text) from the toolbar at the top, or import GLTF/OBJ meshes from File > Import.

### Materials and Lighting

Spline uses a simplified PBR material system.
For each object you can set:
- **Color** - base surface color
- **Roughness / Metalness** - PBR controls that match Three.js and Blender
- **Normal map / Texture map** - image-based detail

Lighting in Spline works with directional lights, ambient light, and environment maps.
The default scene already has sensible lighting.
You rarely need to touch it unless you want a specific mood.

### Built-in Physics and Interactions

Spline's most powerful web feature is its event system.
Without writing any code, you can attach behaviors to objects:

1. Select an object.
2. In the Inspector, click the **Events** tab.
3. Add a trigger: `On Mouse Hover`, `On Mouse Down`, `On Click`, `On Scroll`, etc.
4. Add an action: `Scale`, `Rotate`, `Move`, `Change Color`, `Play Animation`, etc.

This creates interactive 3D UI entirely in the editor.
Spline also supports simple physics simulations (gravity, collisions, springs) from the Physics panel.

### Exporting for the Web

When your scene is ready, click **Export** in the top right.

**For plain HTML** - choose "Spline Viewer" to get a `<script>` + `<spline-viewer>` web component snippet.
The scene is hosted on Spline's CDN; you embed it in minutes.

**For React** - choose "Code Export > React" to get an npm-based component.
Install `@splinetool/react-spline` and use the `<Spline>` component with your scene URL.

**For production** - check "Build Export" to download a self-contained bundle you can host yourself, avoiding Spline's CDN.

### Web Performance Guidelines

Spline scenes load as a binary `.splinecode` file from Spline's CDN.
To keep load times fast:

- **Polygon budget** - aim for under 50k triangles for a hero section.
Check triangle count under View > Statistics.
High-poly meshes slow both download and rendering.

- **Texture size** - use 512x512 or 1024x1024 textures.
Each texture is loaded from CDN; large textures are the most common cause of slow Spline embeds.

- **Object count** - keep it under 30-50 objects for a hero.
Each object is a draw call; too many tanks mobile performance.

- **`loading="lazy"`** - add this attribute to the `<spline-viewer>` tag so it only loads when the user scrolls near it.

## Key Terminology

| Term | Definition |
|------|-----------|
| Spline | Browser-based 3D editor designed to output interactive web embeds |
| Spline Viewer | The `<spline-viewer>` web component that renders a Spline scene in any HTML page |
| `@splinetool/react-spline` | The React component package for embedding Spline scenes in React apps |
| Event | A trigger+action pair in Spline's no-code interaction system |
| .splinecode | Spline's binary export format, served from Spline's CDN by default |
| Polygon budget | The maximum triangle count you aim for to keep performance acceptable |
| PBR | Physically Based Rendering - material model shared by Spline, Three.js, and Blender |
| Scene URL | The CDN URL for a published Spline scene, used in embed code |
| Build Export | Spline's option to download a self-hosted bundle instead of using the CDN |
| Draw call | One GPU instruction to render an object - too many slow mobile performance |

## Code Examples

### Embed in Plain HTML (Spline Viewer)

Get the embed URL from Spline's Export > Spline Viewer panel, then:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>3D Hero</title>
  <style>
    body { margin: 0; }
    spline-viewer {
      width: 100vw;
      height: 100vh;
      display: block;
    }
  </style>
</head>
<body>
  <script type="module"
    src="https://unpkg.com/@splinetool/viewer@1.0.0/build/spline-viewer.js">
  </script>
  <spline-viewer
    url="https://prod.spline.design/YOUR-SCENE-ID/scene.splinecode"
    loading="lazy">
  </spline-viewer>
</body>
</html>
```

Replace `YOUR-SCENE-ID` with the ID from your Spline export URL.

### Embed in React

```bash
npm install @splinetool/react-spline
```

```jsx
import Spline from '@splinetool/react-spline';

export default function HeroSection() {
  return (
    <section style={{ width: '100%', height: '100vh' }}>
      <Spline
        scene="https://prod.spline.design/YOUR-SCENE-ID/scene.splinecode"
        style={{ width: '100%', height: '100%' }}
      />
    </section>
  );
}
```

### Accessing Spline Objects from React

```jsx
import Spline from '@splinetool/react-spline';
import { useRef } from 'react';

export default function HeroSection() {
  const splineRef = useRef();

  function onLoad(spline) {
    splineRef.current = spline;
  }

  function triggerAnimation() {
    const obj = splineRef.current.findObjectByName('Button');
    splineRef.current.emitEvent('mouseDown', obj);
  }

  return (
    <>
      <Spline
        scene="https://prod.spline.design/YOUR-SCENE-ID/scene.splinecode"
        onLoad={onLoad}
      />
      <button onClick={triggerAnimation}>Trigger</button>
    </>
  );
}
```

## Hands-On Exercises

1. Create a free Spline account at spline.design and open a new project.
Add a cube, sphere, and cylinder.
Arrange them in a small composition and apply different PBR materials to each.

2. Select the cube and add an event: Trigger = `On Mouse Hover`, Action = `Scale` to 1.2x.
Preview in the editor with the Play button and hover over the cube.

3. Export the scene using Spline Viewer.
Copy the snippet and paste it into a minimal HTML file on your machine.
Open it in a browser and confirm the scene loads.

4. Check the triangle count under View > Statistics.
If it is over 50k, try replacing a high-poly sphere with a lower-subdivision version (set Subdivisions to 16 or lower in the Inspector).

5. Create a React project using Vite, install `@splinetool/react-spline`, and embed your scene as a hero section with a headline and call-to-action button overlaid using CSS absolute positioning.

## Resources

- [Spline Official Site](https://spline.design/) - editor, documentation, and community gallery
- [Spline Tutorials](https://docs.spline.design/) - official docs covering all editor features
- [Spline React Package](https://www.npmjs.com/package/@splinetool/react-spline) - npm page with usage examples
- [Spline Community Gallery](https://spline.design/community) - real examples to remix and learn from
- [WebGL Stats](https://webglstats.com/) - browser WebGL support data - useful context for 3D on the web

## Reflection Questions

1. What are two situations where you would choose Spline over React Three Fiber for a web project?
2. Why should you keep the polygon count low in a Spline hero section, even though modern GPUs can handle millions of triangles?
3. What is the difference between the Spline Viewer CDN embed and the Build Export option?
4. How does Spline's event system let you add mouse interactions without writing any JavaScript?
5. If a Spline scene is loading slowly on mobile, what are the first two things you would check?

## Next Steps

Day 7 is the Week 1 capstone: you will combine everything from this week - Three.js, R3F, Spline, and Meshy-generated models - into a complete AI-powered 3D hero section.

---

[← Day 5: React Three Fiber](./day-05-react-three-fiber.md) | [Day 7: Week 1 Capstone →](./day-07-week1-capstone.md)
