# 🏮 แผนพัฒนาเว็บลอยกระทง 3D
## Loy Krathong Interactive 3D Experience

> **วันที่วางแผน:** 24 พฤศจิกายน 2569  
> **Stack หลัก:** Three.js + PlayCanvas  
> **สไตล์:** Realistic — บรรยากาศกลางคืนไทยแท้  

---

## 🎯 Vision & Goal

สร้างประสบการณ์เว็บลอยกระทง 3D แบบ Interactive ที่ให้ผู้ใช้:
- **ปล่อยกระทง** ลงแม่น้ำ พร้อมเขียนคำอธิษฐาน
- **ชมฉาก** กลางคืนไทย ริมแม่น้ำ มีตัวละครสาวไทยถือกระทง
- **โคมลอย** ลอยขึ้นฟ้า พร้อม Particle ดาว
- **หมุนกล้อง** รอบฉากได้ด้วย Orbit Controls
- **ฟังเสียง** น้ำไหล + เสียงพลุ + เสียงกระทงกระทบน้ำ

---

## 🏗️ โครงสร้างเทคโนโลยี

```
📁 Loy_kratong/
├── index.html                  ← ไฟล์เดียว (HTML + CSS + JS inline)
├── LOY_KRATHONG_3D_PLAN.md     ← ไฟล์แผนนี้
└── assets/                     ← (optional) เสียง, texture
    ├── sounds/
    │   ├── water_flow.mp3
    │   ├── fireworks.mp3
    │   └── splash.mp3
    └── textures/
        └── waternormals.jpg
```

### เหตุผลที่เลือก Single HTML File
- Deploy ง่าย — แค่ upload ไฟล์เดียว
- ทดสอบได้ทันที — เปิด browser โดยตรง
- ไม่ต้อง build step หรือ npm install

---

## 🔧 Tech Stack รายละเอียด

| ส่วน | Technology | เหตุผล |
|------|------------|--------|
| **ฉากหลัก** | Three.js r165 (CDN) | ควบคุม Shader/Render Pipeline ได้เต็มที่ |
| **น้ำ** | Three.js `Water` addon | Built-in planar reflection + ripple |
| **ตัวละคร + Physics** | PlayCanvas Engine (CDN) | ECS system, built-in physics, 3D audio positional |
| **Post-Processing** | Three.js `EffectComposer` | Bloom + Color Correction |
| **เสียง** | Web Audio API + PlayCanvas Audio | 3D Positional sound |
| **Animation** | Three.js `AnimationMixer` + GSAP | Skeletal + UI tweens |
| **UI Overlay** | HTML/CSS | Form คำอธิษฐาน, ปุ่มปล่อยกระทง |

---

## 🎨 องค์ประกอบฉาก (Scene Elements)

### 1. 🌊 แม่น้ำ (Water Surface)
- **Three.js Water Object** — Planar Reflection แบบ Real-time
- **Normal Map** scrolling — คลื่นละริ้ว, Gerstner Waves
- **waterColor:** `#001533` (น้ำเข้มกลางคืน)
- **distortionScale:** `2.5` (ระริ้วน้ำนุ่มๆ)
- กระทงลอยบนน้ำ — Buoyancy Physics (Y position sync กับ Wave formula)

### 2. 🏮 กระทง (Krathong)
- **วัสดุ:** PBR Material — ใบตอง (Subsurface Scattering), ดอกไม้ (Normal Map)
- **เทียน:** Point Light กระพริบ — animate intensity ด้วย Noise function
- **Bloom Halo:** Emissive Material + UnrealBloomPass ทำให้แสงเทียนเรืองรอง
- **เงา:** Soft Shadow (PCFSoftShadowMap) จากแสงเทียน

### 3. 👘 ตัวละคร (Character) — via PlayCanvas
- **สาวไทย** ชุดไทยสีทอง ยืนริมน้ำ
- **แอนิเมชัน:** Idle breathing, turn head, วางกระทงลงน้ำ (trigger เมื่อ user ปล่อย)
- **Shadow Casting:** ตัวละครทอดเงาลงพื้น + น้ำ
- PlayCanvas ECS component ฝังใน Three.js scene ผ่าน Canvas element แยก หรือ GLTF export

### 4. 🌙 บรรยากาศกลางคืน
- **พระจันทร์:** `THREE.Mesh` Sphere + Emissive texture + Bloom
- **ดาว:** BufferGeometry Particle System 5,000 จุด — Shader ทำให้กระพริบ
- **Skybox:** Gradient shader สีน้ำเงินเข้ม → ม่วงเข้ม → ดำ
- **หมอก:** `THREE.FogExp2` สีกรมท่าเข้ม — ทำให้ขอบฟ้าละลาย

### 5. 🏮 โคมลอย (Sky Lanterns — Khom Loi)
- **InstancedMesh** — render 50+ โคมได้โดยไม่ lag
- **Custom Vertex Shader** — drift ซ้าย-ขวา + ลอยขึ้นเรื่อยๆ
- **Emissive Material** (สีส้มอบอุ่น) + Bloom → ทำให้เรืองแสง
- บางโคมมี particle trail ควันบางๆ

### 6. 🌴 ฉากรอบข้าง
- ต้นไม้ไทย (Low-poly silhouette) ริมฝั่ง
- วัด/ปรางค์ในระยะไกล (เงาดำ Silhouette)
- แสง Rim Light สีทองจากโคม กระทบขอบวัตถุ

---

## 💡 ระบบแสงและเงา (Lighting System)

```
🌙 AmbientLight (0x0a0a2e, intensity: 0.3)  — แสงกลางคืนฐาน
🕯️ PointLight × N (0xffaa44, flickering)   — แสงเทียนแต่ละกระทง
🌕 DirectionalLight (0xfff5cc, 0.8)         — แสงจันทร์
🔆 HemisphereLight (sky:#1a237e, ground:#0d47a1) — Ambient Bounce
```

**Shadow Settings:**
```javascript
renderer.shadowMap.type = THREE.PCFSoftShadowMap
shadowMapSize: 2048×2048
candleLight.shadow.bias = -0.001
```

---

## ✨ Post-Processing Pipeline

```
Render Scene
    ↓
UnrealBloomPass (strength:1.2, radius:0.8, threshold:0.3)
    ↓
(optional) FilmPass — grain นิดๆ เพิ่มความ cinematic
    ↓
OutputPass (ACES Filmic Tone Mapping)
    ↓
Display
```

**Bloom Parameters สำหรับลอยกระทง:**
- เทียนกระทง: `emissiveIntensity: 3.0` → bloom แรง
- โคมลอย: `emissiveIntensity: 2.0` → bloom นุ่ม
- ดวงจันทร์: `emissiveIntensity: 1.5` → glow อ่อนๆ

---

## 🔊 ระบบเสียง (Audio System)

| เสียง | Trigger | ระบบ |
|-------|---------|------|
| น้ำไหล (ambient) | ตลอดเวลา | Web Audio API loop |
| พลุ | Random interval 15-30s | PlayCanvas Audio |
| กระทงกระทบน้ำ | เมื่อกระทงแตะน้ำ | 3D Positional Audio |
| โคมลอย (ลม) | เมื่อปล่อยโคม | Web Audio API |

---

## 🎮 ระบบ Interactive

### ฟีเจอร์ที่ผู้ใช้ทำได้:

1. **เขียนคำอธิษฐาน** — UI Form overlay บน canvas
2. **ปล่อยกระทง** — คลิก/tap ปุ่ม → animation ตัวละครวางกระทง → กระทงลอยออกไป
3. **หมุนกล้อง** — `OrbitControls` (จำกัด angle ไม่ให้มองใต้น้ำ)
4. **ซูมเข้า-ออก** — scroll wheel / pinch
5. **กล้องอัตโนมัติ** — Cinematic mode ไล่กระทงที่ลอยออกไป

### Flow ผู้ใช้:
```
เปิดเว็บ → Loading Screen (ไทย)
    ↓
ฉากปรากฏ — ตัวละครยืนถือกระทง
    ↓
UI: "เขียนคำอธิษฐาน..." (input field)
    ↓
กด "ปล่อยกระทง" → ตัวละคร bow → กระทงวางน้ำ
    ↓
กระทงลอยออกไป — กล้อง follow
    ↓
โคมลอยขึ้นฟ้า — พลุระเบิด
    ↓
(ผู้ใช้หมุนดูฉาก / ปล่อยอีก)
```

---

## 📐 Three.js Skills ที่ใช้ในโปรเจกต์นี้

### ที่เรียนรู้และจะใช้:

| Skill | การใช้งาน | ระดับ |
|-------|-----------|-------|
| **WebGLRenderer** | Renderer หลัก + ToneMapping | 🔴 Core |
| **PCFSoftShadowMap** | เงานุ่ม จากเทียน/จันทร์ | 🔴 Core |
| **Water addon** | แม่น้ำ + Planar Reflection | 🔴 Core |
| **InstancedMesh** | โคมลอย 50+ ตัว | 🟡 Advanced |
| **ShaderMaterial** | ดาว, drift โคม, ไฟกระพริบ | 🟡 Advanced |
| **BufferGeometry** | Particle System ดาว/ประกาย | 🟡 Advanced |
| **UnrealBloomPass** | แสงเรืองรอง เทียน/โคม/จันทร์ | 🔴 Core |
| **AnimationMixer** | Animation GLTF ตัวละคร | 🟡 Advanced |
| **OrbitControls** | หมุนกล้อง | 🟢 Basic |
| **FogExp2** | หมอกกลางคืน | 🟢 Basic |
| **EffectComposer** | Post-Processing pipeline | 🟡 Advanced |
| **CubeCamera** | Environment Reflection บนน้ำ | 🔴 Core |
| **GSAP Tweens** | UI animation, กระทงลอย | 🟡 Advanced |

### PlayCanvas Skills ที่ใช้:
| Skill | การใช้งาน |
|-------|-----------|
| **Entity-Component System** | ตัวละคร + Physics body |
| **AnimationComponent** | Skeletal animation ชุดไทย |
| **SoundComponent** | 3D Positional Audio |
| **RigidBodyComponent** | Buoyancy กระทง |
| **ScriptComponent** | AI behavior ตัวละคร |

---

## 📅 แผนการพัฒนา (Development Phases)

### Phase 1 — ฉากพื้นฐาน
- [ ] Setup HTML boilerplate + Three.js CDN + PlayCanvas CDN
- [ ] สร้าง Renderer + Camera + OrbitControls
- [ ] สร้างแม่น้ำ (Water plane + Normal Map)
- [ ] สร้าง Skybox + พระจันทร์ + ดาว Particle
- [ ] ระบบแสง (Ambient + Hemisphere + Directional moon)

### Phase 2 — กระทง & ตัวละคร
- [ ] สร้าง 3D กระทง (Geometry หรือ GLTF)
- [ ] ใส่เทียน (PointLight + flickering animation)
- [ ] Bloom Post-Processing pipeline
- [ ] ตัวละครสาวไทย (GLTF + AnimationMixer หรือ PlayCanvas)
- [ ] Soft Shadow setup

### Phase 3 — Particle & Effects
- [ ] โคมลอย InstancedMesh + Shader drift
- [ ] ดาว BufferGeometry Particle
- [ ] Bloom fine-tuning
- [ ] FogExp2 + ขอบฟ้า

### Phase 4 — Interactive & Audio
- [ ] OrbitControls จำกัด angle
- [ ] UI form คำอธิษฐาน
- [ ] ปุ่มปล่อยกระทง + animation trigger
- [ ] Web Audio API — เสียงน้ำ + พลุ + splash
- [ ] 3D Positional Audio (PlayCanvas)

### Phase 5 — Polish & Optimize
- [ ] Loading screen ไทย
- [ ] Mobile touch support
- [ ] Frustum Culling + PixelRatio cap
- [ ] Memory management (dispose)
- [ ] Performance profiling

---

## 🎨 Color Palette (บรรยากาศกลางคืนไทย)

| ชื่อ | HEX | ใช้ที่ |
|------|-----|--------|
| น้ำลึก | `#001533` | Water color |
| ท้องฟ้ากลางคืน | `#07090f` | Sky top |
| ม่วงขอบฟ้า | `#1a0533` | Sky horizon |
| แสงทอง | `#ffaa44` | เทียน, โคมลอย |
| แสงจันทร์ | `#fff5cc` | Directional Light |
| Bloom Glow | `#ff8800` | Emissive bloom |
| ใบตอง | `#2d5a1b` | กระทง |
| ดอกดาวเรือง | `#ff6600` | ดอกไม้กระทง |

---

## ⚠️ ข้อควรระวัง

> **[IMPORTANT]** PlayCanvas และ Three.js ใช้ Canvas แยกกัน — ต้อง overlay ให้ถูกต้อง หรือ export ตัวละครเป็น GLTF แล้วโหลดใน Three.js แทน

> **[WARNING]** Bloom + Water Reflection อาจทำให้ GPU หนัก — ต้อง cap `pixelRatio ≤ 2` และ water texture `512×512` สำหรับ mobile

> **[TIP]** ใช้ `THREE.InstancedMesh` สำหรับโคมลอย 50+ ตัว แทนการสร้าง Mesh ใหม่ทุกตัว — ประหยัด Draw Call ได้มาก

> **[NOTE]** เสียงต้องมี User Interaction ก่อน (Web Audio API Policy) — ใช้ Loading Screen click เพื่อ unlock Audio Context

---

## 🚀 การ Deploy

เนื่องจากเป็น Single HTML File:
- **ทดสอบ local:** เปิดไฟล์ใน browser โดยตรง (บาง feature อาจต้องใช้ local server)
- **Deploy:** อัพโหลดไปยัง GitHub Pages, Netlify, Vercel, หรือ Firebase Hosting
- **แนะนำ local server:** `python3 -m http.server 8080`

---

*สร้างโดย Antigravity AI · วางแผนครบก่อนลงมือ · ลอยกระทงปี 2569 🏮*
