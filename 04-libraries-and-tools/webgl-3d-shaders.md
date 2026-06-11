# WebGL, 3D & Shaders for the Web
> How to design and ship WebGL/3D experiences that load fast, hold 60fps on real devices, degrade gracefully, and respect accessibility — without turning the site into a battery-draining gimmick. Covers Three.js, React Three Fiber + drei, Spline, GLSL, postprocessing, performance budgets, fallbacks, and scroll integration.

**Apply when:** a design calls for 3D scenes, shader effects, particle/fluid/distortion visuals, or 3D product viewers · **Related:** [Web Animation Libraries & Stack](animation-libraries-stack.md) · [Scroll-Driven Experiences](../03-site-types/scroll-driven-experiences.md) · [Web Performance & Core Web Vitals](../06-marketing-and-conversion/web-performance-core-web-vitals.md) · [Avoiding Generic 'AI' Aesthetics](../07-aesthetics-and-trends/avoiding-generic-ai-aesthetics.md)

## TL;DR — rules at a glance
- **Justify the 3D.** Use it where it carries information (product config, spatial data, brand signature moment) — not as decoration that costs 2s of load and 200ms of jank.
- **Budgets are hard limits:** <100 draw calls/frame desktop, <50 mobile; <50MB texture VRAM; hero objects 50K-100K tris, total scene ~500K tris.
- **Cap pixel ratio:** `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))`. 3x DPR = 9x pixels for marginal benefit.
- **Never `useState` per frame.** In R3F mutate `ref.current` inside `useFrame`; allocate Vector3/Matrix4 once with `useMemo`, then `.copy()`.
- **Shadows are expensive:** ≤3 shadow-casting lights; 3 of them ≈12ms/frame (over budget). Disable shadow maps on mobile; bake instead.
- **Instance everything repeated:** 5,000 identical trees → `InstancedMesh` → 1 draw call (not 5,000).
- **Compress assets:** Draco (80-95% smaller meshes), KTX2/Basis (~10x less GPU texture memory than PNG/JPEG).
- **Test WebGL support before downloading assets;** ship a static-image/2D fallback when it fails.
- **Honor `prefers-reduced-motion`:** pause rotation, fly-throughs, and particle motion.
- **Disable postprocessing by default on mobile.** SSAO/DoF/motion-blur each add a full-screen pass and can double frame time.
- **On-demand render** (`invalidate()`) for non-animated scenes to save battery and avoid thermal throttling.
- **Never expose free OrbitControls** on a marketing/portfolio site — curate camera paths.
- **Test sustained load,** not just first frame: 60fps can drop to 20fps after ~30s from GPU thermal throttling.

---

## 1. When 3D is worth it vs. a gimmick

3D on the web costs: bundle weight, GPU/battery, thermal headroom, accessibility surface, and engineering time. Spend it only where it returns value.

**Worth it (3D carries the message):**
- **Configurable products** — color/material/angle the user controls (furniture, sneakers, cars, watches). Interactive 3D product viewers consistently reduce return rates (the mechanism is genuine; treat specific percentages from vendor case studies as directional, not universal).
- **Spatial/technical data** — architecture, molecular, maps, mechanical assemblies where rotation aids comprehension.
- **A single signature brand moment** — one memorable hero, not 3D scattered across every section.
- **Data viz** where the third axis is meaningful (rare; 2D is usually clearer).

**Gimmick (cut it):**
- Decorative floating blobs that convey nothing and block the LCP element.
- Full-screen WebGL background behind text that already reads fine on a flat color.
- Spinning logo that exists only because the tool made it easy.
- Anything that pushes Time-to-Interactive past ~3s on an average connection for a content-first page. Google's own research pegs bounce rate increasing significantly beyond 3s; heavy 3D is a top offender.

**Decision rule:** if you can remove the 3D and the user still completes the task with equal clarity *and* the page feels intentional, it was decoration. Keep it only if it earns its frame budget and load cost.

## 2. The stack: pick the right tool

| Need | Use | Notes |
|---|---|---|
| Full custom 3D scenes, React app | **React Three Fiber (R3F)** + **drei** | Declarative; `useFrame` is the render-loop entry. |
| Full custom 3D, no React | **Three.js** (0.170.x-era; ships `WebGPURenderer` w/ WebGL2 fallback) | Most docs/examples; largest community. |
| Minimal WebGL, bundle-size critical | **OGL** | Tiny; no scene-graph luxuries. |
| No-code 3D + interactivity for designers | **Spline** | Physics, event state machine, Webflow-native embed. See §12. |
| Shader effects on existing `<img>`/`<video>` | **VFX-JS** (Codrops) | No full scene needed. |
| Designer-driven shader composition | **Unicorn Studio** (~29KB gz) / **Cables.gl** | Layer/node based; built-in perf scoring. |
| Postprocessing | **pmndrs/postprocessing** + **@react-three/postprocessing** | Composable Bloom/SSAO/DoF/etc. `OutputPass` required as final pass (Three.js 0.155+). |
| GPU compute / draw-call-heavy scenes | **WebGPURenderer** (Three.js) | Chrome 113+, Safari 18+; Firefox: nightly only. 2–10x draw-call perf; adds compute shaders. See §13. |

**Heuristics:**
- Reach for R3F when the page is already React; reach for raw Three.js for non-React or perf-surgical scenes.
- Don't pull in Three.js (~600KB min, ~150KB gz) for an effect VFX-JS or a CSS/canvas shader can do.
- Spline is fastest to *author* but you trade bundle control and fine-grained perf tuning; bake heavy materials to textures and keep web export under 5MB.

## 3. React Three Fiber: the performance-critical patterns

The single biggest R3F mistake is driving per-frame transforms through React state. Don't.

```jsx
// ❌ WRONG — setState every frame triggers React reconciliation, tanks fps
function Box() {
  const [y, setY] = useState(0)
  useFrame(() => setY(v => v + 0.01))      // never do this
  return <mesh position-y={y} />
}

// ✅ RIGHT — mutate the ref directly; React never re-renders
function Box() {
  const ref = useRef()
  useFrame((state, delta) => {
    ref.current.rotation.y += delta         // frame-rate independent via delta
  })
  return <mesh ref={ref} />
}
```

**Never allocate inside `useFrame`** — it produces garbage every frame and triggers GC pauses:

```jsx
const v = useMemo(() => new THREE.Vector3(), [])   // allocate once
useFrame(() => {
  v.set(x, y, z)                                    // reuse, no allocation
  ref.current.position.copy(v)
})
```

**On-demand rendering** for static/interaction-only scenes (huge battery + thermal win):

```jsx
<Canvas frameloop="demand">{/* renders only when invalidate() is called or props change */}</Canvas>
// inside a component: const invalidate = useThree(s => s.invalidate); invalidate() after a change
```

**Asset loading in R3F — always use Suspense:**

```jsx
// Correct pattern: Suspense boundary + useGLTF.preload for critical assets
import { Suspense } from 'react'
import { useGLTF } from '@react-three/drei'

useGLTF.preload('/model.glb')   // fires the fetch before the component mounts

function Model() {
  const { scene } = useGLTF('/model.glb')   // safe — fetch already in flight
  return <primitive object={scene} />
}

export function Scene() {
  return (
    <Suspense fallback={<FallbackMesh />}>
      <Model />
    </Suspense>
  )
}
```

Never render without a `<Suspense>` boundary — the canvas will throw on first render before the asset resolves. `useGLTF.preload` fires the network request early; without it, the model fetch begins only when the component first renders.

**drei helpers worth knowing:** `<Detailed>` (LOD — swaps geometry by camera distance, +30-40% fps in large scenes), `<Instances>/<Instance>` (instancing), `<Environment>` (IBL maps), `<MeshSurfaceSampler>`/`MeshSurfaceSampler` (particle seeding on a mesh), `<Text>`, `<OrbitControls>` (use sparingly — see §11), `<Bvh>` / `three-mesh-bvh` (fast raycasting; set `firstHitOnly: true` for particle picking), `<ScrollControls>` + `useScroll()` (scroll-driven scenes without GSAP — see §11), `<Trail>` + `<Sparkles>` (ready-made particle helpers).

## 4. GLSL shader basics (the mental model)

A shader runs **massively in parallel** on the GPU. Two stages matter for the web:
- **Vertex shader** — runs once per vertex. Cheap relative to fragments.
- **Fragment shader** — runs once per covered pixel. At 1080p that's ~2,000,000 invocations/frame; at 4K ~8,000,000. **Move work to the vertex shader whenever the result can be interpolated.**

**WebGL1 vs WebGL2 GLSL:** Three.js 0.163+ defaults to WebGL2 where available. WebGL2 shaders declare `#version 300 es` and gain: `in`/`out` instead of `attribute`/`varying`, `texture()` instead of `texture2D`, integer types (`ivec2`, `uint`), layout qualifiers, and `texelFetch`. Three.js `ShaderMaterial` auto-injects the version string based on context — don't add `#version 300 es` manually or it will conflict.

```glsl
// Minimal fragment shader (Three.js ShaderMaterial — WebGL1 compatible)
precision mediump float;        // mobile: ~2x faster than highp; use highp only when needed
uniform float uTime;
varying vec2 vUv;               // keep varyings ≤ 8 (WebGL1 minimum guarantee is MAX_VARYING_VECTORS ≥ 8)
void main() {
  float wave = sin(vUv.x * 10.0 + uTime);
  gl_FragColor = vec4(vec3(wave), 1.0);   // output alpha 1.0 (see alpha note in §7)
}
```

**Branch-free is faster.** GPUs hate divergent branches; prefer builtins:

```glsl
// ❌ if (x > 0.5) c = a; else c = b;
vec3 c = mix(b, a, step(0.5, x));     // hardware-optimized, no branch divergence
```

**Precision gotchas (silent data corruption):**
- `mediump float` (WebGL1) range ≈ (-2^14, 2^14), relative precision 2^-10; `lowp float` only (-2, 2).
- **iOS** requires `highp sampler2D` when sampling float textures — without it values clamp to ±2.0 silently.
- WebGL2 ints passed vertex→fragment need `highp int`; `mediump int` on Android silently drops high bits.

**Noise functions are not built-in GLSL.** `simplex` / `perlin` noise must be inlined or imported (e.g., `glsl-noise` npm package or Stefan Gustavson's public-domain GLSL snippets). Three.js does not include them. Use `snoise` or `cnoise`; include only what you use.

**Common building blocks:** `smoothstep` (anti-aliased edges), `fract`/`floor` (tiling), `snoise`/`cnoise` (organic motion — include separately), `length(uv - center)` (radial distortion), `texture2D` / `texture` (WebGL2) displacement (ripple/refraction).

## 5. Postprocessing — powerful, expensive, opt-in

Each effect is a **full-screen render pass**. SSAO, depth-of-field, and motion blur can **double frame time**. Chain order in Three.js: `RenderPass → effect passes → OutputPass`.

```jsx
// R3F
import { EffectComposer, Bloom, ChromaticAberration } from '@react-three/postprocessing'
<EffectComposer disableNormalPass>
  <Bloom luminanceThreshold={0.9} intensity={0.6} mipmapBlur />
  <ChromaticAberration offset={[0.0008, 0.0008]} />
</EffectComposer>
```

**`OutputPass` is required as the final pass** (Three.js 0.155+). It applies tone-mapping and sRGB conversion that `WebGLRenderer` previously handled automatically. Omitting it gives you blown-out or incorrectly gamma-encoded output. `@react-three/postprocessing` handles this internally; raw Three.js EffectComposer does not.

**Rules:**
- **Off by default on mobile.** Gate SSAO/DoF/motion-blur behind a desktop + capability check.
- Bloom is the cheapest high-impact effect (use `mipmapBlur`); SSAO and DoF are the most expensive.
- Prefer baking ambient occlusion into textures for static geometry over real-time SSAO.
- Measure: add each pass while watching frame time (`stats-gl` or `r3f-perf`'s `<Perf>`); if a pass adds >2-3ms on your target device, cut or simplify it.

## 6. Performance budgets (the numbers you must hit)

| Metric | Desktop | Mobile |
|---|---|---|
| Draw calls / frame | <100 (>500 hurts even strong GPUs) | <50 |
| Pixel ratio | `min(DPR, 2)` | `min(DPR, 2)` (often 1.5) |
| Texture VRAM total | <50MB | <50MB (target less) |
| Shadow map size | 1024-2048 (4096 only if critical) | 512-1024 (prefer baked) |
| Texture size | 2048 | 1024 (half of desktop) |
| Triangles total | ~500K | lower; LOD aggressively |

**Lighting frame cost (budget against ~16.6ms/frame for 60fps):**
- Ambient + Hemisphere ≈ 0.5ms
- 1 Directional + shadows ≈ 3ms
- 3 shadow-casting lights ≈ 12ms — **over budget by itself.** PointLight shadows cost 6 renders each (one per cube face).

**Asset pipeline targets:** raw GLB 10MB → Draco 1.5MB → Draco+gzip ~600KB. Draco decode adds 50-200ms CPU — apply only to models >1MB. **Meshopt** (Three.js `GLTFLoader` supports it natively) gives similar compression ratios to Draco with faster decode (~3–5ms vs 50-200ms) — prefer Meshopt when decode latency matters. KTX2 gives ~10x GPU memory reduction vs PNG/JPEG. A single 4096×4096 RGBA uncompressed texture = **64MB VRAM** (over your whole budget). **RGBA8 beats RGB8** — RGB formats are commonly emulated as RGBA internally, so RGB8 is surprisingly slow.

**Pipeline-stall killers (never call these in the render loop):** `getError()`, `getShaderParameter()`, `readPixels()` — each forces a GPU flush + CPU↔GPU round-trip (1ms+ stall). Check `LINK_STATUS` once after `linkProgram`, not `COMPILE_STATUS` after each shader (lets the driver compile in parallel). Use `KHR_parallel_shader_compile` + poll `COMPLETION_STATUS_KHR` non-blocking. Upload video textures *before* `useProgram`/`drawArrays`, not mid-pipeline.

## 7. Mobile performance (the real battleground)

- **Cap DPR** aggressively; many phones report DPR 3 → 9x fill cost.
- **`mediump` precision** in mobile shaders (~2x faster than `highp`); keep varyings ≤8 (WebGL1 spec minimum; fewer = better on old Adreno/Mali).
- **No shadow maps on mobile** — bake shadows or use SSAO; disable all postprocessing by default.
- **`gl.invalidateFramebuffer()`** to discard depth/stencil after rendering — tiled mobile GPUs pay bandwidth for depth buffers not explicitly discarded.
- **Don't set `alpha:false`** on context creation (expensive on some platforms); instead output `alpha 1.0` in the fragment shader.
- **Enable vertex attrib 0 as an array, bind location 0 to `position`** — otherwise macOS falls into slow desktop-GL emulation.
- **WebGL2:** prefer `texStorage2D` over `texImage2D` (avoids driver allocating the full mip chain, +30% memory, and catches mismatched mip sizes at allocation).
- **Capability checks before rendering to float textures:** `EXT_color_buffer_float` (render-to-float fails on most mobile WebGL1 — fall back to half-float); check `EXT_float_blend` separately (float32 render ≠ float32 blend). Always read `MAX_TEXTURE_SIZE` — only **4096** is guaranteed across devices.
- **Thermal throttling is the silent killer:** a scene at 60fps can fall to 20fps after ~30s sustained. **Test sustained load** and design a step-down (drop DPR, disable postFX, reduce particle count) when frame time degrades.

**Guaranteed-minimum limits to design against:** `MAX_TEXTURE_IMAGE_UNITS` ≥ 8, `MAX_VERTEX_ATTRIBS` ≥ 16, `MAX_FRAGMENT_UNIFORM_VECTORS` ≥ 64.

## 8. Loading strategy & fallbacks

```js
// Detect WebGL BEFORE downloading any 3D asset — failing after a big download is a severe UX failure
function detectWebGL() {
  try {
    const c = document.createElement('canvas')
    // Prefer WebGL2; fall back to WebGL1
    const ctx = c.getContext('webgl2') || c.getContext('webgl') || c.getContext('experimental-webgl')
    if (!ctx) return { supported: false, version: 0 }
    return { supported: true, version: ctx instanceof WebGL2RenderingContext ? 2 : 1 }
  } catch { return { supported: false, version: 0 } }
}
const gl = detectWebGL()
if (!gl.supported) renderStaticFallbackImage()   // ship a curated still / 2D version
// gl.version === 1: disable float FBOs, WebGL2-only features
```

- **Defer below-the-fold 3D** — async-load assets not visible at first paint; don't block LCP.
- **Lazy-load the heavy modules** (Three.js, decoders) and the model only when the section approaches the viewport (IntersectionObserver).
- **Host Draco/KTX2 decoders on a CDN**; preconnect to it.
- **Cache large assets via service worker or `Cache-Control: immutable` + content-hash URL.** On return visits, a 600KB GLB should load from disk cache, not the network.
- **Show progress, allow skip.** Never a full-screen loader the user can't skip.
- **Target Time-to-Interactive <3s** for 3D viewers on average connections.
- **Always provide a static-image or simplified-2D fallback** for failed WebGL or extremely low-end GPUs — and for `prefers-reduced-motion` and crawlers.

## 9. Accessibility & reduced-motion (non-negotiable)

- **`prefers-reduced-motion: reduce`** → pause/disable continuous rotation, camera fly-throughs, and particle motion. Render a single composed static frame instead.

```jsx
const reduce = useReducedMotion()            // e.g. via a matchMedia hook
useFrame((_, delta) => { if (!reduce) ref.current.rotation.y += delta })
```

- **Audio off by default.** Autoplay audio without a visible mute is universally hated — put a sound toggle in the loading modal *before* audio starts.
- **Don't trap users** in full-screen loaders or non-skippable intros.
- WebGL is a `<canvas>` — invisible to assistive tech. Provide a **text alternative / accessible description** of the content, and ensure the *task* (e.g., product configuration) is also completable via accessible DOM controls, not gestures inside the canvas only.
- Keep motion subtle; large-field continuous motion can trigger vestibular discomfort even without `prefers-reduced-motion` set.

## 10. Particles, fluid & distortion effects

**CPU vs GPU:** CPU-based particle updates bottleneck around **~50,000** on a mid-range device; GPU (FBO/GPGPU or compute) handles **millions**. Rule of thumb: 16,384 particles × 120fps = ~2M position evaluations/sec — CPU handles that only if each evaluation is trivial; any per-particle collision or noise lookup shifts the crossover point well below 50K.

**GPGPU (FBO) pattern** — simulate on the GPU by rendering into textures:
- Store positions in a `DataTexture` (`FloatType`, RGBA), sampled with **`NearestFilter`** (position accuracy; `LinearFilter` only for display-facing shaders).
- Run the simulation fragment shader on an **OrthographicCamera + screen quad** (FBO render-to-texture) for max parallelism.
- Use Three.js `GPUComputationRenderer` for the ping-pong setup.
- Display material: `AdditiveBlending` + `depthWrite:false` + `depthTest:false` for glow accumulation without depth artifacts.

```jsx
// Particle display material (R3F)
<points>
  <bufferGeometry /* positions seeded via MeshSurfaceSampler */ />
  <pointsMaterial size={2} sizeAttenuation
    blending={THREE.AdditiveBlending} depthWrite={false} depthTest={false} />
</points>
```

**Distortion / displacement:** drive a `ShaderMaterial` with a noise/texture lookup and a `uProgress`/`uTime` uniform; do the heavy displacement in the vertex shader where possible. For image/video distortion on existing DOM elements, **VFX-JS** avoids building a whole scene.

**Batching:** combine 1000 sprites into one `drawArrays` (use degenerate triangles to join discontinuous strips) rather than 1000 calls.

## 11. Integrating 3D with scroll

Two patterns for scroll-driven 3D:

**Option A — GSAP ScrollTrigger** (works outside R3F too; pairs with Lenis):

```js
gsap.to(material.uniforms.uProgress, {
  value: 1,
  scrollTrigger: {
    trigger: '#section',
    scrub: 1.5,          // fractional scrub = smooth easing independent of scroll speed (not `true`)
    fastScrollEnd: true, // snap to end state if the user scrolls past very fast
  },
})
```

**Option B — drei `<ScrollControls>` + `useScroll()`** (R3F-native; no extra library):

```jsx
import { ScrollControls, useScroll } from '@react-three/drei'

function Scene() {
  const scroll = useScroll()       // scroll.offset: 0→1 across the entire scroll length
  useFrame(() => {
    ref.current.rotation.y = scroll.offset * Math.PI * 2
  })
  return <mesh ref={ref} />
}

<Canvas>
  <ScrollControls pages={4} damping={0.1}>
    <Scene />
  </ScrollControls>
</Canvas>
```

Use `<ScrollControls>` when the entire page layout is driven by the canvas; GSAP ScrollTrigger when the 3D scene coexists with normal DOM sections.

- **Curate the camera, don't free it.** Define preset camera views tied to scroll positions that guide attention. **Never expose free OrbitControls on a marketing/portfolio site** — it disorients non-technical users and breaks the narrative. Don't add free-roam when there's limited content to show.
- Drive **uniforms** (progress, morph, color) from scroll for the smoothest effect; reserve camera moves for major beats.
- Keep the R3F render loop and scroll library on the same RAF where possible to avoid double-driving; with on-demand rendering, call `invalidate()` from the ScrollTrigger `onUpdate`.

## 12. Spline: no-code 3D — patterns & gotchas

Spline competes with code-based 3D only on authoring speed. Trade-offs are real:

**Embedding options:**
- **`<spline-viewer>` web component** — simplest embed; ~150KB custom element + the scene bundle. Works in any HTML including Webflow.
- **`@splinetool/react-spline`** — React wrapper; same bundle weight.
- **Iframe** — isolates Spline's runtime from your bundle; slightly heavier. Use when you want full isolation.

**Export size control (the #1 problem):**
- Spline defaults to exporting full-resolution meshes and uncompressed textures. Typical naive exports are 15–40MB.
- **Bake materials to textures** before export: in Spline, select object → Export → Bake Materials. Reduces geometry + shader complexity.
- Run the exported GLB through `gltf-transform optimize` (from `@gltf-transform/cli`) — applies Draco + KTX2 in one command; often drops 80%+.
- **Hard target: web export <5MB.** If you can't hit it after baking, the scene is too complex for Spline.

**Watermark / plan:**
- Free tier shows Spline branding in the bottom-right corner of the embed.
- Starter ($16/mo, billed annually as of 2025) removes watermark.
- Pro ($32/mo) adds code export, no watermark, and mobile-optimized export option.

**When Spline breaks down:**
- You need fine-grained postprocessing (custom bloom thresholds, depth of field).
- Scene must respond to complex app state (auth, cart, API data).
- Bundle budget is <200KB — pull in R3F + drei instead; Spline's runtime is not tree-shakeable.
- You need server-side rendering or static export — Spline requires browser canvas.

**Spline physics & events:** built-in; works for hover/click state machines and spring-based animations. For anything with collision groups or game-like logic, switch to code.

## 13. WebGPU: when to reach for it

Three.js ships `WebGPURenderer` (same scene graph, different backend). You can swap it in one line:

```js
import WebGPURenderer from 'three/addons/renderers/webgpu/WebGPURenderer.js'
const renderer = new WebGPURenderer({ antialias: true })
await renderer.init()   // async — await before first render
```

**Browser support (2026 baseline):** Chrome 113+ (full), Edge 113+, Safari 18+ (desktop + iOS 18+), Firefox desktop (nightly / flag only — not production-ready). **Do not use `WebGPURenderer` as the primary renderer without a WebGL2 fallback** — Firefox stable has no WebGPU.

**When it's worth it:**
- Draw-call heavy scenes: WebGPU's indirect draw + render bundles cut CPU overhead dramatically; measurable at >500 draw calls/frame.
- GPGPU compute: replaces FBO ping-pong with `ComputeNode` — cleaner code, no UV-based hacks.
- Scenes bottlenecked on shader compilation (WebGPU compiles WGSL async natively).

**When to stay on WebGL2:**
- Any scene where Firefox coverage matters.
- Simple scenes (<200 draw calls) — the switch adds complexity with no measurable gain.
- Legacy tooling (Three.js shaders using GLSL in `ShaderMaterial`) — `WebGPURenderer` uses TSL (Three.js Shading Language / WGSL-based node material) natively; raw GLSL `ShaderMaterial` works but bypasses the new material system.

**Fallback pattern:**

```js
import { isAvailable } from 'three/addons/renderers/webgpu/WebGPURenderer.js'
const renderer = isAvailable()
  ? new WebGPURenderer({ antialias: true })
  : new THREE.WebGLRenderer({ antialias: true })
if (renderer.init) await renderer.init()
```

## Concrete specs & numbers
- **Draw calls:** <100/frame desktop, <50 mobile; >500 problematic on any GPU.
- **Pixel ratio:** `Math.min(window.devicePixelRatio, 2)`; 3x DPR = 9x pixels.
- **Texture VRAM:** <50MB total; 4096×4096 RGBA uncompressed = 64MB.
- **Texture sizes:** 2048 desktop / 1024 mobile. Shadow maps: 512-1024 mobile, 1024-2048 desktop, 4096 only when critical. Mipmaps add +30% memory — only when objects scale down.
- **Polygons:** hero 50K-100K tris, props 500-5K, total ~500K.
- **Lighting cost:** Ambient+Hemisphere ~0.5ms; 1 Directional+shadows ~3ms; 3 shadow-casters ~12ms (over budget). ≤3 shadow-casting lights.
- **Compression:** Draco 80-95% smaller (50-200ms decode; >1MB models only). KTX2 ~10x GPU memory reduction. Pipeline: 10MB → Draco 1.5MB → +gzip ~600KB.
- **Particles:** CPU bottleneck ~50K/frame on a mid-range device; GPU (FBO/GPGPU) handles millions. 16,384 particles × 120fps = ~2M evaluations/sec — GPU mandatory if each evaluation involves noise or collision.
- **Precision:** `mediump float` WebGL1 range (-2^14, 2^14) @ 2^-10; `lowp float` (-2, 2). MAX_VARYING_VECTORS ≥ 8 (WebGL1 spec minimum); keep varyings ≤8, fewer on old Adreno/Mali.
- **GL minimums:** texture units ≥8, vertex attribs ≥16, fragment uniform vectors ≥64, guaranteed `MAX_TEXTURE_SIZE` 4096.
- **Instancing:** 5,000 trees → 1 draw call. LOD via `<Detailed>` → +30-40% fps.
- **Postprocessing:** each pass = one full-screen render; SSAO/DoF/motion-blur can double frame time.
- **Targets:** TTI <3s for 3D viewers; >2s load loses ~60% of visitors.
- **WebGPU:** 2-10x improvement in draw-call-heavy / compute-heavy scenes vs WebGL; Chrome 113+, Safari 18+, Firefox nightly only (2026). See §13.
- **Spline:** free tier shows watermark; Starter ~$16/mo removes it; Pro ~$32/mo adds code + mobile export. Web export target <5MB. Unicorn Studio runtime ~29KB gz.

## Tools & libraries
- **Three.js** — core WebGL abstraction (0.170.x-era); ships `WebGPURenderer` (Chrome 113+, Safari 18+; Firefox nightly only — always provide WebGL2 fallback). Use for any non-trivial 3D.
- **React Three Fiber** — declarative Three.js for React; `useFrame` is the render-loop entry. Use in React apps; **don't** use `useState` for per-frame values.
- **drei** — R3F helpers: `<Detailed>` (LOD), `<Instances>`, `<Environment>`, `<Text>`, `MeshSurfaceSampler`, `<Bvh>`/`three-mesh-bvh`, `<ScrollControls>`/`useScroll()`, `<Trail>`, `<Sparkles>`.
- **pmndrs/postprocessing** + **@react-three/postprocessing** — composable Bloom/SSAO/DoF/MotionBlur/ChromaticAberration/GodRays. Opt-in, desktop-gated.
- **GPUComputationRenderer** / **MeshSurfaceSampler** — GPGPU particle sim + surface seeding.
- **Draco / KTX2+Basis Universal / Meshopt** — mesh and texture compression. Meshopt decode ~3-5ms (faster than Draco's 50-200ms) at similar ratio; prefer Meshopt when decode latency matters.
- **GSAP + ScrollTrigger / Lenis** — scroll→uniform/camera bridge + smooth scroll.
- **Spline** — no-code 3D (physics, event state machine, Webflow-native, 7M+ scenes). Fast to author; bake materials, keep export <5MB.
- **OGL** — minimal WebGL when bundle size dominates. **VFX-JS** — WebGL effects on existing `<img>`/`<video>`. **Unicorn Studio / Cables.gl** — designer shader tools.
- **stats-gl** (frame-time overlay) + **r3f-perf** (`<Perf>` component, includes memory/gpu stats) + **Spector.js** (frame capture / shader debug) — dev only.
- **`@react-three/xr`** — VR/AR/immersive sessions in R3F; niche but relevant for product showcases.
- **Learning/inspiration:** Bruno Simon's *Three.js Journey*; `mesh3d.gallery` (curated Three.js sites).
- **When NOT to use:** don't load Three.js for an effect VFX-JS/CSS/canvas can do; don't use Spline when you need fine bundle/perf control; don't use postprocessing on mobile by default; don't use OrbitControls on marketing sites.

## Do / Don't
**Do**
- Mutate refs in `useFrame`; `useMemo` your Vector3/Matrix4; use `delta` for frame-rate independence.
- Wrap R3F async assets in `<Suspense>`; call `useGLTF.preload` before the component mounts.
- Instance repeated geometry; merge static geometry; use LOD (`<Detailed>`).
- Cap DPR at 2; compress with Draco or Meshopt + KTX2; bake lighting/shadows for static elements.
- Detect WebGL2/WebGL1 before downloading assets; ship a static fallback; cache large assets.
- Honor `prefers-reduced-motion`; provide a sound toggle before audio; allow skipping loaders.
- On-demand render static scenes; design a thermal step-down for sustained load.
- Curate camera paths; drive scroll via uniforms with fractional `scrub`.
- Add `OutputPass` as the final EffectComposer pass (Three.js 0.155+).
- Include noise functions explicitly (not built-in GLSL); prefer `mediump`; keep varyings ≤8.

**Don't**
- Use `useState`/`setState` for per-frame transforms, or allocate objects inside `useFrame`.
- Exceed draw-call / texture / polygon budgets; use >3 shadow-casting lights; ship 4K uncompressed textures.
- Call `getError()`/`readPixels()`/`getShaderParameter()` in the render loop.
- Enable shadow maps or postprocessing on mobile by default; set `alpha:false` on context.
- Use branches in shaders where `mix()`/`step()` work; use `mediump`/`highp` carelessly (iOS float textures need `highp sampler2D`).
- Expose free OrbitControls on marketing/portfolio sites; autoplay audio; trap users in non-skippable intros.
- Block LCP with above-the-fold 3D you could defer.
- Assume `WebGPURenderer` works in Firefox stable (it doesn't as of 2026).
- Omit `OutputPass` in Three.js EffectComposer (broken tone-mapping/gamma).
- Rely on GLSL built-ins for noise — simplex/perlin must be included manually.
- Render R3F async components without a `<Suspense>` boundary.

## Common pitfalls & how to avoid them
- **React re-render per frame** — `useState` in `useFrame`. *Fix:* mutate `ref.current` directly.
- **GC stutter** — `new Vector3()`/`new Matrix4()` each frame. *Fix:* `useMemo` once, `.copy()`/`.set()`.
- **VRAM leaks** — not calling `texture.source.data.close?.()` on `ImageBitmap` textures after upload. *Fix:* close the bitmap.
- **Pipeline stalls** — `getError()`/`readPixels()` in the loop. *Fix:* never query GPU state per frame; use `KHR_parallel_shader_compile`.
- **Silent shader value clamping** — missing `highp` for float textures (iOS) or `highp int` for WebGL2 varyings (Android). *Fix:* set correct precision explicitly.
- **Render-to-float failures on mobile** — assuming float FBOs work. *Fix:* check `EXT_color_buffer_float`; fall back to half-float; check `EXT_float_blend` separately.
- **Over-budget shadows** — multiple shadow-casting (esp. point) lights. *Fix:* ≤3, bake the rest.
- **Mip-chain memory blowup** — `texImage2D` allocating full mip chain. *Fix:* `texStorage2D` (WebGL2); mipmaps only when downscaling.
- **Heavy asset before WebGL check** — large download then a black canvas. *Fix:* detect first, lazy-load, fallback.
- **First-frame-only testing** — passes at start, throttles to 20fps after 30s. *Fix:* test sustained load; implement step-down.
- **Spline weight creep** — unbaked materials, oversized export. *Fix:* bake to textures, run `gltf-transform optimize`, keep <5MB.
- **Missing `OutputPass`** — tone-mapping and sRGB conversion lost when using EffectComposer (Three.js 0.155+). *Fix:* always add `OutputPass` as the final pass.
- **R3F without Suspense** — canvas throws before async asset resolves. *Fix:* wrap all async R3F components in `<Suspense fallback={...}>`.
- **Late model fetch** — model fetch starts only when component renders, adding 200-500ms. *Fix:* call `useGLTF.preload('/model.glb')` at module scope.
- **Assuming noise is built-in GLSL** — `simplex`/`perlin` don't exist in GLSL; shader silently errors. *Fix:* include a noise GLSL snippet or `glsl-noise` package.
- **WebGPU without fallback** — works in Chrome/Safari, black canvas in Firefox stable. *Fix:* use `isAvailable()` guard; fall back to `WebGLRenderer`.

## What real users complain about
- **"It melted my phone / drained my battery."** Sustained full-screen WebGL with continuous animation and no on-demand rendering — classic thermal-throttle + battery complaint.
- **"I got lost spinning the model and couldn't find my way back."** Free OrbitControls with no reset on a marketing site.
- **"Blasted music with no mute."** Autoplay audio in WebGL intros — near-universally hated.
- **"Staring at a loader I couldn't skip."** Non-skippable full-screen 3D intros.
- **"Beautiful but I left before it loaded."** Heavy 3D blocking first paint; >2s loads shed ~60% of visitors.
- **"It made me dizzy."** Large continuous camera motion ignoring `prefers-reduced-motion`.
- **"Janky scroll."** 3D re-rendering at full DPR while scrolling, fighting the scroll library; fix with DPR cap, on-demand render, and fractional `scrub`.
- **"Blank box on my older phone/laptop."** No WebGL/extension check and no fallback.

## Senior-level checklist
- [ ] 3D is justified — it carries information or is one signature moment, not scattered decoration.
- [ ] Draw calls <100 desktop / <50 mobile; verified with `stats-gl`.
- [ ] `setPixelRatio(min(DPR, 2))`; mobile may cap lower.
- [ ] Repeated geometry instanced; static geometry merged; LOD via `<Detailed>` for large scenes.
- [ ] Meshes Draco-compressed (>1MB), textures KTX2; decoders on CDN; texture VRAM <50MB.
- [ ] ≤3 shadow-casting lights; shadow maps off + baked on mobile.
- [ ] No `useState`/allocations in `useFrame`; refs mutated; `delta`-based motion.
- [ ] No `getError()`/`readPixels()`/per-shader `COMPILE_STATUS` in the loop; parallel compile used.
- [ ] Shader precision correct (`mediump` mobile default; `highp sampler2D` for iOS float textures; `highp int` for WebGL2 varyings); varyings ≤8; branches replaced with `mix`/`step`.
- [ ] WebGL + needed extensions detected *before* asset download; static/2D fallback ships.
- [ ] Below-the-fold 3D deferred; modules + model lazy-loaded; TTI <3s.
- [ ] `prefers-reduced-motion` pauses rotation/fly-through/particles; static frame fallback.
- [ ] Audio (if any) off by default with a toggle before it starts; loaders skippable.
- [ ] Postprocessing off by default on mobile; desktop passes measured (<2-3ms each on target device).
- [ ] On-demand rendering for non-animated scenes; thermal step-down for sustained load.
- [ ] Tested on a real mid-range phone for 30s+ — sustained fps, not just the first frame.
- [ ] Camera curated (no free OrbitControls on marketing/portfolio); scroll drives uniforms with fractional `scrub` + `fastScrollEnd`.
- [ ] If using Spline: export <5MB, materials baked, watermark plan confirmed, fallback for zero-WebGL.
- [ ] If using WebGPURenderer: WebGL2 fallback in place; `await renderer.init()` before first render; Firefox stable not assumed.
- [ ] `OutputPass` is last in EffectComposer chain (Three.js 0.155+); postprocessing measured per-pass.
- [ ] `<Suspense>` boundary wraps R3F async assets; `useGLTF.preload` fires before component mounts.
- [ ] Large assets cached (immutable `Cache-Control` or service worker) to eliminate re-download on return visits.

## Sources
- React Three Fiber — performance & scaling docs: https://r3f.docs.pmnd.rs/advanced/scaling-performance
- Three.js manual: https://threejs.org/manual/
- Three.js WebGPURenderer migration guide: https://github.com/mrdoob/three.js/wiki/Migration-Guide
- pmndrs/postprocessing: https://github.com/pmndrs/postprocessing
- drei: https://github.com/pmndrs/drei
- WebGL best practices (MDN): https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/WebGL_best_practices
- Khronos WebGL public wiki — performance: https://www.khronos.org/webgl/wiki/WebGL_and_OpenGL_Differences
- KTX2 / Basis Universal: https://github.com/KhronosGroup/KTX-Software
- Draco: https://github.com/google/draco
- Meshopt / gltf-transform: https://gltf-transform.dev/
- GSAP ScrollTrigger: https://gsap.com/docs/v3/Plugins/ScrollTrigger/
- Lenis: https://github.com/darkroomengineering/lenis
- Spline: https://spline.design/
- glsl-noise (Patricio Gonzalez Vivo): https://github.com/patriciogonzalezvivo/glsl-noise
- Bruno Simon — Three.js Journey: https://threejs-journey.com/
