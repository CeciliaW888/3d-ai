# Day 20: Full AI-Assisted 3D Workflow

## Learning Objectives

- Map each tool covered in this course to its stage in a complete production pipeline.
- Apply a decision framework for choosing between Meshy, Tripo3D, Blender, Three.js, R3F, and Spline.
- Trace a real end-to-end case study from concept to deployed website.
- Identify when AI accelerates work and when human judgment is still necessary.
- Estimate time and effort for a typical 3D web project using AI tools.

## Core Concepts

### The Six-Stage AI-Assisted Pipeline

A complete 3D web project moves through six stages.
Each stage has an AI-first tool and a fallback for when you need more control.

**Stage 1 - Generate (Days 3 and 8)**
Start with Meshy.ai or Tripo3D.
A well-written text prompt or reference image gets you a usable base mesh in 2-5 minutes.
This replaces hours of manual modeling for most web assets.
Use Meshy for organic shapes (characters, creatures, stylized objects) and when quality matters more than speed.
Use Tripo3D when you need a fast turnaround on simpler hard-surface objects.

**Stage 2 - Refine (Day 19)**
Import your AI-generated mesh into Blender.
Apply the Decimate modifier to bring polygon count down to web-friendly levels (2,000-10,000 faces for most assets).
UV unwrap for texture baking.
Fix any obvious mesh issues - non-manifold edges, flipped normals.
This stage typically takes 15-30 minutes with practice.
Skip Blender entirely if the AI output is clean enough for your use case.

**Stage 3 - Texture (Day 9)**
Use AI texture generation tools to create or refine surface materials.
Tools like Leonardo.ai and Stable Diffusion with ControlNet can generate tileable textures from a text description.
Apply in Blender's material editor and bake to a texture atlas if needed.
Export with textures packed into the GLB.

**Stage 4 - Animate (Day 10)**
Upload your clean, UV-unwrapped mesh to Mixamo.
Auto-rigging runs in about 30 seconds.
Browse the animation library (2,000+ free motions) and download with skin as FBX.
For non-humanoid shapes, Mixamo cannot help - use manual keyframe animation in Blender instead.
Re-export as GLB with animations included.

**Stage 5 - Render on the Web (Days 4-5, 11-12, 15)**
Load your optimized GLB into Three.js or R3F.
Apply the performance patterns from Day 18: LOD for distance, instancing for repeated objects, texture compression with KTX2 for large scenes.
For simple embeds with no coding, use Spline (Day 6) or model-viewer.

**Stage 6 - Deploy**
For static sites: GitHub Pages (free, zero configuration, great for portfolios).
For dynamic apps: Vercel or Netlify (both have free tiers with instant deploys from Git).
Run gltf-transform before deploying - final file size optimization pays off in load time.

### Decision Framework: Which Tool When

Use this framework when starting a new project:

| Situation | Best Tool |
|-----------|-----------|
| Need a 3D model in 5 minutes | Meshy.ai or Tripo3D |
| Model needs precise control or custom rigging | Blender |
| Want to animate a humanoid character | Mixamo |
| Building a React app | React Three Fiber |
| No coding background, designer mindset | Spline |
| Maximum performance control | Three.js |
| Model file is over 5 MB | gltf-transform + Draco |
| Need game physics | Three.js + Rapier |

The most common mistake is over-engineering early.
Start with the fastest AI tool, verify it works in the browser, then refine.

### Case Study: A 3D Portfolio Website

Let us walk through building a portfolio page with an animated 3D character in the hero section.
Total time with AI tools: approximately 2-3 hours.

**What we are building**: A portfolio hero section with an animated cartoon developer character.
The character waves when the page loads, then idles.
There is a particle background behind the model.

**Step 1 - Generate the character (15 min)**
Open Meshy.ai and enter the prompt:
"cartoon developer character, wearing hoodie, clean topology, game-ready, 3000 polygons, T-pose"
Select the best result and download as GLB.

**Step 2 - Refine in Blender (20 min)**
Import the GLB into Blender.
Check polygon count (should be around 3,000-8,000 for a hero asset).
Apply a Decimate modifier if needed to bring it under 6,000 faces.
Apply all transforms (Ctrl+A > All Transforms).
Pack external data (File > External Data > Pack Resources).
Export as GLB with Draco compression and Apply Transformations enabled.

**Step 3 - Rig and animate with Mixamo (10 min)**
Upload the GLB to mixamo.com.
Use Auto-Rigger with the default settings.
Select the "Wave" animation from the library, then download as FBX with Skin.
Also download the "Idle" animation as FBX.
In Blender, use the NLA Editor to combine both animations into one GLB export.
The export should include both "Wave" and "Idle" action clips.

**Step 4 - Optimize the file (5 min)**
Run gltf-transform inspect to check size.
Run `npx gltf-transform optimize developer.glb developer-final.glb --draco`.
Target: under 3 MB for a hero character.

**Step 5 - Build the React Three Fiber scene (45 min)**
Set up a Canvas with an ambient light and a directional light.
Load the model with useGLTF.
Use useAnimations from @react-three/drei to play the "Wave" clip on mount, then transition to "Idle".
Add an OrbitControls with enableZoom=false and autoRotate at a slow speed.
Add a particle Points mesh for the background - simple, performant, visually impressive.

**Step 6 - Deploy to Vercel (5 min)**
Push the project to a GitHub repository.
Connect to Vercel and deploy.
The portfolio is live with a custom URL.

### AI as Accelerator, Not Replacement

AI tools handle the tedious parts: raw mesh generation, auto-rigging, texture creation.
Human judgment is still required for: deciding what to build, adjusting what AI gets wrong, optimizing for performance, and writing the rendering code.
The goal is to compress 40 hours of traditional 3D work into 4 hours while maintaining quality.

## Key Terminology

| Term | Definition |
|------|-----------|
| Pipeline | The ordered sequence of tools and steps from raw concept to deployed product |
| NLA Editor | Blender's Non-Linear Animation editor for combining and sequencing animation clips |
| Auto-Rig | Automated skeleton creation; Mixamo applies a humanoid rig to an uploaded mesh |
| Particle system | Large number of small geometry instances (points, sprites) used for effects |
| Vercel | Hosting platform with automatic deploys from Git; free tier available |
| LOD | Level of Detail; showing simpler geometry for distant objects to save GPU |
| Animation clip | Named segment of keyframe data (e.g., "Wave", "Idle") stored in a GLTF file |
| Topology | The arrangement of polygons in a mesh; good topology deforms cleanly during animation |

## Code Examples

### R3F Scene with Animated Character

```javascript
import { Canvas } from '@react-three/fiber';
import { useGLTF, useAnimations, OrbitControls } from '@react-three/drei';
import { useEffect, useRef } from 'react';

useGLTF.preload('/models/developer-final.glb');

function Developer() {
  const { scene, animations } = useGLTF('/models/developer-final.glb');
  const { actions } = useAnimations(animations, scene);

  useEffect(() => {
    // Play "Wave" once, then transition to "Idle"
    const wave = actions['Wave'];
    const idle = actions['Idle'];
    if (wave && idle) {
      wave.reset().play();
      wave.clampWhenFinished = true;
      wave.loop = THREE.LoopOnce;
      wave.addEventListener('finished', () => {
        idle.reset().fadeIn(0.5).play();
      });
    }
  }, [actions]);

  return <primitive object={scene} scale={1} position={[0, -1, 0]} />;
}

function ParticleField({ count = 1500 }) {
  const positions = new Float32Array(count * 3);
  for (let i = 0; i < count; i++) {
    positions[i * 3]     = (Math.random() - 0.5) * 20;
    positions[i * 3 + 1] = (Math.random() - 0.5) * 20;
    positions[i * 3 + 2] = (Math.random() - 0.5) * 20;
  }
  return (
    <points>
      <bufferGeometry>
        <bufferAttribute attach="attributes-position" args={[positions, 3]} />
      </bufferGeometry>
      <pointsMaterial size={0.04} color="#0d9488" />
    </points>
  );
}

export function PortfolioHero() {
  return (
    <Canvas camera={{ position: [0, 0, 4], fov: 50 }}>
      <ambientLight intensity={0.5} />
      <directionalLight position={[5, 5, 5]} intensity={1} />
      <Developer />
      <ParticleField />
      <OrbitControls enableZoom={false} autoRotate autoRotateSpeed={0.5} />
    </Canvas>
  );
}
```

### Deploying to GitHub Pages (Static)

```bash
# Install gh-pages if not installed
npm install --save-dev gh-pages

# Add to package.json scripts
# "predeploy": "npm run build",
# "deploy": "gh-pages -d dist"

# In vite.config.js set base to your repo name
# base: '/your-repo-name/'

# Deploy
npm run deploy
```

## Hands-On Exercises

1. Draw (on paper or in a tool like Excalidraw) the 6-stage pipeline from this lesson.
   Map each day of this course to one of the stages.

2. Using the decision framework table, choose the right tool stack for each of these projects:
   - A 3D product showcase for an e-commerce site (the client has no coding team)
   - A first-person shooter browser game with 50 enemy characters
   - A portfolio site for a front-end developer with React skills

3. Follow Steps 1-4 from the case study using any character or object you like.
   Note how long each step actually takes and compare to the estimates.

4. Time yourself: from blank browser tab to a deployed R3F scene with a GLTF model, how long does it take?
   Use the provided case study as your guide.

5. Take any project from a previous week and identify one stage where AI could have saved more time.
   What would you do differently?

## Resources

- [Three.js Journey course](https://threejs-journey.com/) - Bruno Simon's gold-standard course for deep Three.js skills
- [React Three Fiber documentation](https://r3f.docs.pmnd.rs/) - official R3F docs with hooks, Canvas, and ecosystem
- [Mixamo animation library](https://www.mixamo.com/) - 2,000+ free motion-capture animations from Adobe
- [gltf-transform.dev](https://gltf-transform.dev/) - full documentation for the optimization CLI
- [Vercel deployment guide](https://vercel.com/docs/frameworks/vite) - deploy a Vite app to Vercel in minutes

## Reflection Questions

1. At which stage of the pipeline does AI help you the most?
   Where does it help the least?

2. For a freelance client project with a 1-week deadline, how would you prioritize the six stages?
   What would you cut if time ran out?

3. When is it better to use Spline instead of R3F?
   What are you giving up when you choose the no-code option?

4. Why is it important to optimize the GLB file before deploying, even if the model looks fine in your local dev environment?

5. Think about a 3D project you would like to build.
   Map it to the six-stage pipeline and estimate how long each stage would take with AI tools.

## Next Steps

Tomorrow is Day 21 - the final capstone.
You will choose one of four complete project options and build and deploy a full 3D web experience of your own.
Everything from this course comes together in a single project you can show to the world.

[Back to Day 19: Blender to Web](./day-19-blender-to-web.md)
