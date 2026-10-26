# 3D AI for the Web

> Build AI-powered 3D experiences for websites, web apps, and games — in 21 days.

**21 days · 30 min/day · English · Beginner-friendly**

You already know how to use AI tools.
This course teaches you 3D — and how to use AI to accelerate every step of the 3D workflow.

## Already took the Three.js + AI Portfolio course?

If you completed [`threejs-learning`](https://github.com/CeciliaW888/threejs-learning), the Week 1 Three.js foundations (Days 1-4) will feel familiar.
You can skim those days for the new AI-tool angles (Meshy, Tripo3D) and jump to Day 5 (React Three Fiber) where new territory begins.
This course is broader — it adds R3F, Spline, AI generation tools, game development with Rapier.js, and a Blender-to-web pipeline that the portfolio course doesn't cover.

---

## What you'll learn

- How 3D rendering works (meshes, materials, lighting, shaders)
- Three.js and React Three Fiber for embedding 3D on the web
- Spline for no-code 3D web design
- AI 3D generation tools: Meshy.ai, Tripo3D, Luma AI Genie
- AI texture generation and animation with Mixamo
- Building 3D website hero sections and scroll-driven experiences
- Building 3D product configurators and web app viewers
- Browser-based 3D games with physics
- GLSL shaders (with AI as your writing partner)
- Performance optimization, Draco compression, Blender-to-web pipeline
- A complete AI-assisted 3D workflow from idea to deployed experience

## Curriculum

### Week 1: 3D Foundations + AI Toolkit

| Day | Topic | Type |
|-----|-------|------|
| 1 | [How 3D Works: Meshes, Materials & Lighting](week-01/day-01-how-3d-works.md) | Foundations |
| 2 | [The 3D Software Landscape](week-01/day-02-3d-software-landscape.md) | Foundations |
| 3 | [Text-to-3D AI: Meshy & Tripo3D](week-01/day-03-text-to-3d-ai.md) | Concepts |
| 4 | [Three.js Fundamentals](week-01/day-04-threejs-fundamentals.md) | Foundations |
| 5 | [React Three Fiber: 3D in React](week-01/day-05-react-three-fiber.md) | Concepts |
| 6 | [Spline: No-Code 3D for Web](week-01/day-06-spline-for-web.md) | Concepts |
| 7 | [Week 1 Capstone: AI 3D Hero Section](week-01/day-07-week1-capstone.md) | Project |

### Week 2: AI-Powered 3D for Web & Apps

| Day | Topic | Type |
|-----|-------|------|
| 8 | [Image-to-3D AI Tools](week-02/day-08-image-to-3d.md) | Concepts |
| 9 | [AI Texture Generation](week-02/day-09-ai-textures.md) | Concepts |
| 10 | [AI Animation: Mixamo & Beyond](week-02/day-10-ai-animation.md) | Concepts |
| 11 | [3D for Websites: Scroll & Parallax](week-02/day-11-3d-for-websites.md) | Advanced |
| 12 | [3D for Web Apps: Configurators & Viewers](week-02/day-12-3d-for-web-apps.md) | Advanced |
| 13 | [AI as Your 3D Coding Partner](week-02/day-13-ai-coding-partner.md) | Advanced |
| 14 | [Week 2 Capstone: 3D Product Configurator](week-02/day-14-week2-capstone.md) | Project |

### Week 3: Games, Shaders & Final Project

| Day | Topic | Type |
|-----|-------|------|
| 15 | [Browser 3D Games with Three.js](week-03/day-15-browser-3d-games.md) | Foundations |
| 16 | [AI Game Asset Creation](week-03/day-16-ai-game-assets.md) | Concepts |
| 17 | [GLSL Shaders: Visual Magic](week-03/day-17-glsl-shaders.md) | Advanced |
| 18 | [Performance Optimization for Web 3D](week-03/day-18-performance-optimization.md) | Advanced |
| 19 | [Blender to Web: Export Pipeline](week-03/day-19-blender-to-web.md) | Advanced |
| 20 | [Full AI-Assisted 3D Workflow](week-03/day-20-full-ai-workflow.md) | Review |
| 21 | [Final Capstone: Your 3D Web Experience](week-03/day-21-final-capstone.md) | Project |

## Repository Structure

```
3d-ai/
├── README.md
├── week-01/           Daily markdown study guides (Days 1–7)
├── week-02/           Daily markdown study guides (Days 8–14)
├── week-03/           Daily markdown study guides (Days 15–21)
├── projects/          Project starter files
└── website/           Installable PWA course app
    ├── index.html
    ├── styles.css
    ├── main.js
    ├── manifest.json
    ├── sw.js
    ├── modules/       Day content (lazy-loaded HTML fragments)
    ├── data/          Quiz JSON + flashcards
    ├── diagrams/      SVG diagrams
    └── icons/         PWA icons
```

## Course Website

**Live site: https://ceciliaw888.github.io/3d-ai/**

The site is a PWA — installable to your home screen for offline access.

## Getting Started

No install required for the first week.
For Three.js days (Day 4+), you'll need Node.js:

```bash
npm create vite@latest my-3d-project -- --template vanilla
cd my-3d-project
npm install three
npm run dev
```

For React Three Fiber (Day 5+):

```bash
npm create vite@latest my-r3f-project -- --template react
cd my-r3f-project
npm install three @react-three/fiber @react-three/drei
npm run dev
```

## Tools Used

| Tool | Purpose | Free? |
|------|---------|-------|
| Three.js | WebGL 3D library | ✅ |
| React Three Fiber | Three.js for React | ✅ |
| Spline | No-code 3D editor | ✅ (free tier) |
| Blender | 3D modeling suite | ✅ |
| Meshy.ai | Text/image to 3D AI | ✅ (free tier) |
| Tripo3D | Fast AI 3D generation | ✅ (free tier) |
| Mixamo | Auto-rigging + animation | ✅ |
| Rapier.js | Physics engine | ✅ |
