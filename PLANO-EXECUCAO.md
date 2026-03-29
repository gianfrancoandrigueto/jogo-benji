# Plano de Execucao: DEMON SLAYER 3D PRO FIGHTER

## Indice
1. [Contexto e Visao](#contexto)
2. [Estado Atual - O que ja foi feito](#estado-atual)
3. [Arquitetura Tecnica](#arquitetura)
4. [Proximas Etapas](#proximas-etapas)
5. [Guia Unreal Engine 5](#guia-ue5)

---

## Contexto

Jogo de luta 3D competitivo com tema **Demon Slayer (Kimetsu no Yaiba)**, rodando em um unico arquivo HTML com **Three.js v0.171** via CDN. Combina gameplay de fighting game profissional com ferramenta de treino (frame data, hitbox viewer, replay, combo counter).

**Plataforma dupla:**
- **Three.js (browser)**: Jogo jogavel no navegador com graficos semi-realistas PBR
- **Unreal Engine 5 (futuro)**: Guia de migracao para qualidade AAA

---

## Estado Atual - O que ja foi feito

### FASE 1: Engine 3D Base [COMPLETO]

| Item | Status | Detalhes |
|------|--------|----------|
| Three.js via CDN importmap | OK | v0.171 com ES modules |
| WebGL Renderer | OK | antialias, PCFSoftShadowMap, shadow map 2048x2048 |
| Tone Mapping | OK | `NeutralToneMapping` (Khronos PBR Neutral), exposure 1.5 |
| Post-processing | OK | EffectComposer: RenderPass + UnrealBloomPass + OutputPass + SMAAPass |
| Environment Map | OK | `RoomEnvironment` via PMREMGenerator para reflexos PBR, envMapIntensity 0.35 |
| Color Space | OK | SRGBColorSpace output |
| Camera 2.5D | OK | PerspectiveCamera(38), tracking ponto medio, zoom dinamico, shake no impacto |
| Game Loop | OK | requestAnimationFrame com fixed timestep 60fps, acumulador delta |
| Input System | OK | Buffer circular 30 frames, motion inputs (QCF, DP, Super QCFx2) |
| Fog | OK | FogExp2 density 0.022 |

### FASE 2: Iluminacao Cinematica [COMPLETO]

Fisicamente correta (r155+ requer intensidades x PI):

| Luz | Tipo | Cor | Intensidade | Posicao | Funcao |
|-----|------|-----|-------------|---------|--------|
| Key Light | DirectionalLight | 0xc0d0e8 | 5.0 | (6,12,6) | Luar principal, com sombras |
| Fill Light | DirectionalLight | 0x887766 | 1.6 | (-4,6,-3) | Preenchimento quente oposto |
| Front Light | DirectionalLight | 0xddddee | 2.5 | (0,4,10) | Ilumina personagens de frente |
| Ambient | AmbientLight | 0x445566 | 2.0 | global | Luz base |
| Hemisphere | HemisphereLight | sky:0x6688aa ground:0x332211 | 1.6 | global | Bounce natural ceu/chao |
| Rim Light | DirectionalLight | 0x6644aa | 1.6 | (-2,5,-6) | Backlight roxo Demon Slayer |
| Hit Light | PointLight | dinamico | 0-15 | dinamico | Flash de impacto, decay=0 |

### FASE 3: Cenario 3D [COMPLETO]

| Elemento | Implementacao |
|----------|---------------|
| Chao | PlaneGeometry 40x30 com textura procedural de terra (3000 pontos de ruido + rachaduras) + normal map procedural |
| Lua | SphereGeometry com 3 camadas de glow (solid + halo r=2.5 + aura r=4) |
| Montanhas | 3 camadas de profundidade (parallax) com 8 cones cada |
| Wisteria | 4 arvores com tronco + 5 galhos + 20 flores emissivas + 12 tendrils pendentes |
| Estrelas | 400 pontos com size attenuation |
| Particulas | 300 (wisteria petals roxas + fireflies amarelo-verdes) com additive blending |
| Fundo | Cor 0x080818 com fog combinante |

### FASE 4: Personagens 3D (PBR) [COMPLETO]

**Sistema de construcao `buildChar()`:**
- Hierarquia articulada: root > hips > torso > chest > head + arms > elbows > hands + legs > knees > feet
- `MeshStandardMaterial` PBR em todas as partes (roughness + metalness reais)
- Outline via inverted hull (BackSide, scale 1.04x)
- Olhos 3 camadas: esclera branca + iris colorida + pupila preta
- Cabelo: cap + volume lateral + franja (5 mechas) + spikes
- Armas detalhadas: katana (lamina + edge highlight + tsuba + handle + pommel), dual swords (serrilhadas), cutelos (com corrente), rapier (com glow na ponta)
- Haori/capa: painel traseiro + flaps laterais, com texturas procedurais
- Helper `mk()` para posicionamento correto de meshes

**10 Personagens implementados:**

| # | Nome | Texturas Especiais | Decoracoes | Arma |
|---|------|-------------------|------------|------|
| 0 | Tanjiro | Checker verde/preto (torso + haori) | Brincos hanafuda, cicatriz, caixa nas costas | Katana preta |
| 1 | Nezuko | Asanoha pattern rosa | Bambu na boca, pontas laranja no cabelo | Nenhuma (chutes) |
| 2 | Zenitsu | - | - | Katana dourada emissiva |
| 3 | Inosuke | - | Mascara javali (sphere), presas | Dual katanas serrilhadas |
| 4 | Giyu | Half-half haori (burgundy/geometrico) | Rabo de cavalo | Katana azul |
| 5 | Rengoku | Flame haori (branco + chamas gradiente) | Sobrancelhas bifurcadas vermelhas | Katana vermelha emissiva |
| 6 | Shinobu | - | Borboleta no cabelo | Rapier com glow |
| 7 | Tengen | - | Faixa com gemas (vermelho/verde), marca no olho | Cutelos duplos com corrente |
| 8 | Muzan | - | Fedora branco (3 meshes), gravata vermelha | Nenhuma (tentaculos) |
| 9 | Akaza | - | Marcas azuis geometricas (face + corpo), esclera azul | Nenhuma (punhos) |

### FASE 5: Sistema de Combate [COMPLETO]

| Sistema | Detalhes |
|---------|----------|
| Frame Data | Cada golpe: startup, active, recovery, damage, hitstun, blockstun, knockback |
| Hitbox/Hurtbox | THREE.Box3 collision, visualizacao wireframe toggleavel (F1) |
| Golpes normais | Light (J), Medium (K), Heavy (L) por personagem |
| Golpes especiais | QCF+btn, DP+btn, QCFx2+btn (super) |
| Combo System | Contador real (hitstun chain), damage scaling 10%/hit, display HUD |
| Blocking | Segurar back = bloqueia, blockstun separado de hitstun |
| Knockdown | Knockback Y > 0.08 derruba, timer de levantamento |
| Veneno | Shinobu aplica poison DoT |
| Super Meter | Ganha com hits (8) e blocks (3), gasta 100 no super |
| Hitstop | 3 frames normais, 6 frames em supers |

### FASE 6: VFX 3D [COMPLETO]

| Tipo | Implementacao |
|------|---------------|
| Water | TorusGeometry expandindo (Tanjiro, Giyu) |
| Fire | SphereGeometry + PointLight dinamico decay=0 (Rengoku, Tanjiro Hinokami) |
| Lightning | Line geometry com midpoint displacement (Zenitsu) |
| Blood | SphereGeometry rosa expandindo (Nezuko) |
| Shockwave | TorusGeometry horizontal (Akaza) |
| Butterfly | Planes flutuantes roxo/rosa (Shinobu) |
| Explosion | SphereGeometry laranja (Tengen) |
| Dark | SphereGeometry roxo-escuro (Muzan) |
| Slash | PlaneGeometry (Inosuke) |
| Hit particles | Reutiliza sistema de particulas ambientais (recolore ultimos 35) |
| Camera shake | Proporcional ao dano (0.05 normal, 0.1 pesado, 0.18 super) |

### FASE 7: Sistemas Pro Player [COMPLETO]

| Sistema | Status | Detalhes |
|---------|--------|----------|
| Training Mode | OK | Toggle F5, vida/super infinito, dummy configuravel (4 modos) |
| Frame Data Display | OK | Toggle F2, mostra startup/active/recovery + barra visual + frame advantage |
| Hitbox Viewer | OK | Toggle F1, wireframe verde (hurtbox) e vermelho (hitbox ativo) |
| Input Display | OK | Ultimos 20 inputs com direcoes e botoes |
| Combo Counter | OK | HUD com hits + damage total, damage scaling |
| Frame Step | OK | P pausa, ] avanca 1 frame |
| Replay | OK | F9 grava, F10 reproduz |
| Debug Panel | OK | Toggle F4, mostra FPS/frame/estado/move atual |
| Round System | OK | Best of 3, timer 99s, win indicators |

### FASE 8: HUD/UI [COMPLETO]

- Barras de vida com gradiente (verde > amarelo > vermelho)
- Super meter com glow azul
- Timer dourado centralizado
- Round display
- Combo display animado (scale bounce)
- Menu principal com selecao
- Character select com grid 2x5 + preview
- Announce system (FIGHT!, round wins, match end)
- Pause overlay com backdrop-filter blur
- REC indicator

---

## Arquitetura Tecnica

```
index.html (arquivo unico)
├── CSS (HUD, menus, overlays)
├── Three.js v0.171 (via CDN importmap)
│   ├── EffectComposer (bloom + OutputPass + SMAA)
│   ├── RoomEnvironment (PBR env map)
│   └── NeutralToneMapping
├── ENGINE
│   ├── Renderer (WebGL, PBR, sombras)
│   ├── Camera (2.5D tracking + shake + zoom)
│   ├── Lighting (6 luzes + 1 hit light dinamico)
│   └── Game Loop (fixed timestep 60fps)
├── SCENE
│   ├── Ground (textura procedural + normal map)
│   ├── Moon (3-layer glow)
│   ├── Mountains (3 camadas parallax)
│   ├── Wisteria trees (4, com flores emissivas)
│   ├── Stars (400 pontos)
│   └── Particles (300, wisteria + fireflies)
├── CHARACTERS (10)
│   ├── buildChar() - construtor articulado PBR
│   ├── Fighter class - state machine + physics + combat
│   ├── Texturas procedurais (checker, asanoha, giyu, flame)
│   └── Decoracoes unicas por personagem
├── COMBAT
│   ├── Hitbox (Box3 collision)
│   ├── Frame data por golpe
│   ├── Combo system + damage scaling
│   ├── Motion inputs (buffer 30 frames)
│   └── VFX (9 tipos)
├── TRAINING
│   ├── Dummy modes (stand/block/counter)
│   ├── Frame data display
│   ├── Hitbox viewer
│   └── Frame step + replay
└── UI
    ├── HUD (HTML overlay)
    ├── Menus (HTML)
    └── Announces
```

### Controles

| Tecla | Acao |
|-------|------|
| WASD | Movimento P1 |
| JKL | Ataques P1 (light/medium/heavy) |
| Setas | Movimento P2 |
| Numpad 1-3 | Ataques P2 |
| F1 | Toggle Hitbox Viewer |
| F2 | Toggle Frame Data |
| F4 | Toggle Debug Panel |
| F5 | Toggle Training Mode |
| P | Pausar |
| ] | Frame Step (quando pausado) |
| R | Reset posicao (training) |
| 1-4 | Dummy mode (training) |
| 5 | Toggle HP infinito (training) |
| 6 | Toggle Super infinito (training) |
| F9 | Gravar/parar replay |
| F10 | Reproduzir replay |

---

## Proximas Etapas

### ETAPA A: Melhorias Visuais [COMPLETO]
- [x] Animacoes mais suaves com interpolacao (lerp entre poses)
- [x] Haori/capa com simulacao simples de pano (vertex deformation)
- [x] Trail effect nas armas durante ataques (ribbon geometry)
- [x] Dust particles ao andar/cair
- [x] Screen flash branco em impactos criticos
- [x] Letterbox (barras pretas) + camera zoom em supers

### ETAPA B: Gameplay [COMPLETO]
- [x] IA avancada para P2 (aproximar, defender, contra-atacar, variar comportamento)
- [x] Dash forward/backward (double-tap, 8f forward / 6f back com i-frames)
- [x] Throw/grab move (forward + heavy quando perto, 80 damage, lanca oponente)
- [x] Air combos (max 2 hits por pulo, juggle com gravidade crescente)
- [x] Juggle gravity scaling (previne combos infinitos no ar)

### ETAPA C: Polish [COMPLETO]
- [x] Contagem regressiva "3, 2, 1, FIGHT!" (240 frames, com SFX e flash)
- [x] Tela de vitoria com pose (bracos erguidos, letterbox, camera zoom, 3s)
- [x] Sound design (Web Audio API sintetizado: hit, heavy hit, block, dash, throw, super, KO, countdown, select, confirm)
- [x] Move list/command list (M durante pause, tabela com frame data + advantage)
- [x] SFX em menus (select, confirm)

### ETAPA D: Performance + Conteudo [COMPLETO]
- [x] VFX object pool (20 meshes pre-criados, reutilizaveis)
- [x] Shared geometries (sphere6/8/12, cyl6/8, box, cone4, plane, torus)
- [x] Stage system modular (clearStage/buildStage com hot-swap)
- [x] Segundo cenario: Snowy Mountain (neve no chao, montanhas com snow caps, pinheiros com neve, rochas de gelo, lua azulada, particulas de neve, fog azul)
- [x] Stage select no menu (A/D para trocar, preview em tempo real)
- [x] Particulas adaptadas por cenario (petalas roxas vs flocos de neve)

### ETAPA E: Refinamento Final [COMPLETO]
- [x] 3 modos de jogo: VS CPU, VS Local (2P), Training
- [x] Dificuldade da IA selecionavel (Easy/Medium/Hard) com Q/E no menu
- [x] Crouch attacks (down+attack = versao low de cada golpe, hitbox mais baixa)
- [x] Wake-up attack (ataque durante recovery de knockdown = reversal invencivel)
- [x] AI desativada em VS Local (2P) - ambos jogadores controlam com teclado
- [x] Fix: training mode agora e gMode=2 (nao 1)
- [x] Pause mostra hint do Move List (M)

---

## Guia Unreal Engine 5

### Visao Geral da Migracao

O jogo Three.js serve como **prototipo funcional**. A migracao para UE5 eleva para qualidade AAA com:
- Lumen (iluminacao global em tempo real)
- Nanite (geometria virtualizada)
- Niagara (VFX avancados)
- Modelos high-poly com PBR textures
- Physics-based combat

### Pipeline de Producao

#### 1. Setup do Projeto UE5

```
Novo Projeto > Games > Third Person > Blueprint
Project Settings:
  - Rendering > Global Illumination: Lumen
  - Rendering > Reflections: Lumen
  - Rendering > Shadow Map Method: Virtual Shadow Maps
  - Rendering > Anti-Aliasing: TSR
  - Rendering > Nanite: Enabled
```

Estrutura de pastas:
```
Content/
├── Characters/
│   ├── Tanjiro/
│   │   ├── Mesh/        (SK_Tanjiro.uasset)
│   │   ├── Materials/   (M_Tanjiro_Body, M_Tanjiro_Haori...)
│   │   ├── Textures/    (T_Tanjiro_BaseColor, Normal, Roughness...)
│   │   ├── Animations/  (AM_Tanjiro_Idle, Attack_L, Attack_M...)
│   │   └── Blueprint/   (BP_Tanjiro)
│   ├── Nezuko/
│   └── ... (10 personagens)
├── Environment/
│   ├── WisteriaForest/
│   │   ├── Meshes/
│   │   ├── Materials/
│   │   └── Textures/
│   └── SnowyMountain/
├── VFX/
│   ├── Niagara/
│   │   ├── NS_WaterSlash
│   │   ├── NS_FireBreathing
│   │   ├── NS_LightningFlash
│   │   └── NS_BloodExplosion
│   └── Materials/
├── Combat/
│   ├── DataTables/
│   │   └── DT_MoveList.uasset
│   ├── Blueprint/
│   │   ├── BP_CombatComponent
│   │   ├── BP_HitboxComponent
│   │   └── BP_ComboManager
│   └── AnimMontages/
├── UI/
│   ├── Widgets/
│   │   ├── WBP_HUD
│   │   ├── WBP_HealthBar
│   │   └── WBP_FrameData
│   └── Materials/
└── Core/
    ├── BP_GameMode_Fighter
    ├── BP_PlayerController_Fighter
    └── BP_GameState_Fighter
```

#### 2. Pipeline de Personagens

**Modelagem (Blender 3.6+ ou Maya)**
1. Modelar base mesh humanoide (~15k polys para game-ready)
2. Para cada personagem, customizar: proporcoes, rosto, cabelo, roupa
3. Usar Subdivision Surface para smooth, aplicar antes de exportar
4. UV unwrap com Smart UV Project + manual cleanup
5. Exportar como FBX com Apply Modifiers

**Escultura (ZBrush - opcional)**
1. Importar base mesh do Blender
2. Subdividir para ~2M polys
3. Esculpir detalhes: musculos, rugas, textura de tecido, padroes de roupa
4. Gerar Normal Map (Map > Create NormalMap)
5. Gerar Displacement Map
6. Exportar maps em 4K

**Texturizacao PBR (Substance Painter)**
1. Importar mesh + bake de normal/AO/curvature
2. Criar materiais por parte: pele (SSS), tecido, metal, cabelo
3. Pintar: Base Color, Normal, Roughness, Metallic, Emissive
4. Para Tanjiro: checker pattern no haori via procedural fill
5. Para Akaza: marcas geometricas via stencil/projection
6. Exportar texturas no preset "Unreal Engine 4 (Packed)"

**Rigging (Blender)**
1. Usar Rigify addon para skeleton humanoide
2. Gerar rig automatico > ajustar bone positions
3. Weight paint: auto weights > cleanup manual
4. Verificar: IK chains nos bracos/pernas, bone constraints
5. Exportar skeleton com mesh como FBX > "Include Armature"

**Animacao (Mixamo + Manual)**
1. Upload mesh para Mixamo > auto-rig
2. Baixar animacoes base: idle, walk, run, jump
3. Em Blender: criar animacoes de ataque customizadas
   - Light attack: 12 frames (rapido)
   - Medium attack: 18 frames
   - Heavy attack: 28 frames
   - Especiais: 30-45 frames
4. Exportar cada animacao como FBX separado
5. Na UE5: criar Animation Montages para cada ataque

**Importacao na UE5**
1. Arrastar FBX para Content Browser > Import Settings:
   - Skeleton: Create New (primeiro personagem) ou Assign Existing
   - Material Import Method: Create New Materials
   - Normal Import Method: Import Normals and Tangents
2. Atribuir texturas PBR aos Material Instances
3. Configurar Physics Asset para ragdoll
4. Criar Animation Blueprint com State Machine

#### 3. Sistema de Combate (Blueprints)

**Game Mode (BP_GameMode_Fighter)**
```
Variables:
  - RoundNumber (int)
  - Player1Wins, Player2Wins (int)
  - RoundTimer (float)
  - CurrentState (enum: Menu, Select, Fight, RoundEnd, MatchEnd)

Event BeginPlay:
  → Set State = Menu

Custom Event: StartRound
  → Spawn Player1 at SpawnPoint1
  → Spawn Player2 at SpawnPoint2
  → Reset HP, Super
  → Set Timer = 99
  → Set State = Fight
```

**Combat Component (BP_CombatComponent)**
```
Variables:
  - HP, MaxHP (float)
  - SuperMeter (float, 0-100)
  - CurrentState (enum: Idle, Walking, Jumping, Attacking, Hitstun, Blockstun, Knockdown)
  - CurrentMove (struct: MoveData)
  - ComboCount (int)
  - IsBlocking (bool)
  - InputBuffer (array of InputFrame)

Functions:
  - ExecuteMove(MoveData) → Play Montage, Set State=Attacking, Reset HitFlag
  - TakeHit(Damage, Knockback, Hitstun) → Apply Damage, Set State
  - CheckBlock(Direction) → Return bool
  - UpdateFrameData() → Track startup/active/recovery
```

**Move Data (DataTable: DT_MoveList)**
```
Row Structure (Struct: S_MoveData):
  - MoveName (FName)
  - CharacterID (int)
  - Startup (int frames)
  - Active (int frames)
  - Recovery (int frames)
  - Damage (float)
  - Hitstun (int frames)
  - Blockstun (int frames)
  - Knockback (FVector)
  - HitboxOffset (FVector)
  - HitboxSize (FVector)
  - AnimMontage (UAnimMontage*)
  - VFX_Niagara (UNiagaraSystem*)
  - IsSpecial (bool)
  - MotionInput (array of int: directions)
  - IsSuper (bool)
  - SuperCost (float)
```

**Hitbox System (BP_HitboxComponent)**
```
On Attack Frame (Active frames):
  → Spawn Box Collision at HitboxOffset (relative to character)
  → On Overlap with enemy Hurtbox:
    → If enemy.IsBlocking: Apply Blockstun, Pushback, Spawn Block VFX
    → Else: Apply Damage, Hitstun, Knockback, Spawn Hit VFX, Camera Shake
    → Increment Combo if enemy was in Hitstun
    → Add to SuperMeter
  → On Recovery: Destroy Hitbox
```

**Animation Blueprint (ABP_Fighter)**
```
State Machine:
  Idle ←→ Walking (speed > 0)
  Idle → Jumping (is in air)
  Idle → Crouching (crouch input)
  Any → Attacking (via Montage)
  Any → Hitstun (received hit)
  Any → Blockstun (blocked hit)
  Any → Knockdown (heavy knockback)
  Knockdown → GetUp → Idle

Blend Spaces:
  BS_Locomotion: Speed + Direction → Walk/Run anims

Montage Slots:
  DefaultSlot: Attack montages override state machine
```

#### 4. VFX com Niagara

**Slash Effect (NS_WaterSlash)**
```
Emitter: Ribbon Renderer
  - Spawn Rate: 60/s for 0.3s
  - Color: Lerp(Cyan, Blue, Age)
  - Width: Curve (grow then shrink)
  - Lifetime: 0.4s
  - Material: Translucent, Additive, Custom UV scroll

Emitter: Sprite Particles (droplets)
  - Spawn Burst: 30
  - Velocity: Cone, spread 45deg
  - Color: Cyan with alpha decay
  - Size: 2-5cm
```

**Fire Breathing (NS_FireBreathing)**
```
Emitter: Sprite (flames)
  - Spawn Rate: 200/s
  - Initial Velocity: Forward + Upward drift
  - Color over Life: Yellow → Orange → Red → Black
  - Size over Life: Grow then shrink
  - Turbulence Noise Force
  - Light Renderer: Dynamic point light per particle

Emitter: Ember Sprites
  - Spawn: 50/s
  - Small orange dots with long lifetime
  - Gravity: slight upward
```

**Lightning (NS_LightningFlash)**
```
Emitter: Beam Renderer
  - Source: Character hand
  - Target: Impact point
  - Noise: High frequency, amplitude 30cm
  - Regenerate beam shape each frame (flicker)
  - Color: White core, Yellow glow
  - Lifetime: 0.15s (very fast)

Post-process: Bloom intensity spike for 2 frames
```

#### 5. Cenario com Lumen

**Wisteria Forest**
```
Nivel de Iluminacao:
  - Directional Light (Moon): Intensity 3 lux, Color: Cool Blue
    - Enable: Cast Ray Traced Shadows
    - Atmosphere Sun Light: true
  - Sky Light: Intensity 0.5, Cubemap capture
  - Fog: Exponential Height Fog
    - Density: 0.02
    - Volumetric Fog: Enabled
    - Albedo: Dark Purple

Lumen Settings:
  - Lumen Global Illumination: Enabled
  - Lumen Reflections: Enabled
  - Software Ray Tracing (or Hardware if RTX available)
  - Final Gather Quality: High

Assets:
  - Wisteria trees: Nanite-enabled static meshes
  - Flowers: Instanced Static Mesh Component (thousands)
  - Ground: Landscape with 3-layer material (dirt, stone, moss)
  - Particles: Niagara system for floating petals + fireflies
```

#### 6. Camera System

**BP_CameraManager**
```
Variables:
  - TargetFOV, CurrentFOV (float)
  - ShakeIntensity (float)
  - IsInCinematic (bool)
  - LetterboxAmount (float)

Tick:
  → MidPoint = (Player1.Location + Player2.Location) / 2
  → Distance = |Player1.Location - Player2.Location|
  → TargetFOV = Clamp(40 + Distance * 3, 35, 65)
  → CurrentFOV = FInterp(CurrentFOV, TargetFOV, DeltaTime, 3)
  → Camera.SetFOV(CurrentFOV)
  → Camera.Location = Lerp(Camera.Location, MidPoint + Offset, 0.08)

Custom Event: ApplyShake(Intensity, Duration)
  → Start Camera Shake (Perlin Noise based)
  → Decay over Duration

Custom Event: SuperCinematic
  → Set IsInCinematic = true
  → Lerp FOV to 25 (zoom in)
  → Apply Letterbox (animate top/bottom black bars)
  → Set Time Dilation = 0.3
  → After 1s: Restore all
```

#### 7. IA Inimiga

**BP_AIController_Fighter**
```
Behavior Tree:
  Root (Selector)
  ├── Sequence: Defensive
  │   ├── Condition: Enemy is attacking
  │   ├── Task: Block (hold back direction)
  │   └── Task: Counter-attack after blockstun ends
  ├── Sequence: Approach
  │   ├── Condition: Distance > attack range
  │   ├── Task: Walk toward enemy
  │   └── Task: Dash if enemy is far
  ├── Sequence: Attack
  │   ├── Condition: Distance < attack range
  │   ├── Task: Choose attack (weighted random: 50% light, 30% medium, 15% heavy, 5% special)
  │   └── Task: Execute combo if hit confirmed
  └── Sequence: Punish
      ├── Condition: Enemy whiffed heavy attack
      └── Task: Full combo punish

Difficulty Scaling:
  - Reaction time: 4 frames (hard) to 20 frames (easy)
  - Block probability: 90% (hard) to 30% (easy)
  - Combo execution: Full combos (hard) to single hits (easy)
```

#### 8. UI System

**Widget Blueprint: WBP_HUD**
```
Layout:
  ┌─────────────────────────────────────────┐
  │ [P1 Name]     [99]     [P2 Name]       │
  │ [████████]             [████████]       │
  │ [▓▓▓▓]                    [▓▓▓▓]       │
  │                                          │
  │                                          │
  │                                          │
  │                                          │
  │ 5 HITS              [Frame Data]        │
  │ DMG: 234                                │
  └─────────────────────────────────────────┘

Health Bar: Progress Bar with Material Instance
  - Gradient: Green → Yellow → Red based on %
  - Glow pulse when low HP
  - Smooth interpolation on damage

Super Meter: Progress Bar
  - Fill: Blue gradient with pulse animation
  - Flash when full (ready to use super)
```

#### 9. Otimizacao

| Tecnica | Aplicacao |
|---------|-----------|
| Nanite | Meshes do cenario (arvores, pedras, chao) |
| LOD | Personagens: LOD0 (15k), LOD1 (5k), LOD2 (1k) |
| Virtual Shadow Maps | Automatico com Lumen |
| Occlusion Culling | Automatico |
| Niagara GPU Sim | Particulas ambientais |
| Instance Static Mesh | Flores de wisteria, pedras |
| Texture Streaming | Pool size adequado |
| TSR | Upscaling temporal (render 75%, display 100%) |

**Target Performance:**
- 60fps em GPU mid-range (RTX 3060 / RX 6700)
- 30fps com Lumen + ray tracing em GPU high-end

---

## Historico de Problemas Resolvidos

### Bug: Personagens invisiveis
**Causa**: `Object.assign(mesh, {position: new Vector3()})` falha silenciosamente em Three.js porque `position` e readonly (getter).
**Solucao**: Substituido todos os ~30 `Object.assign` pelo helper `mk()` que usa `.position.set()`.

### Bug: Cena muito escura
**Causa**: Three.js r155+ usa luzes fisicamente corretas - intensidades precisam ser ~3.14x maiores.
**Solucao**:
- Intensidades multiplicadas por PI
- Adicionado `RoomEnvironment` como environment map (PBR sem env map = preto)
- Trocado `ACESFilmicToneMapping` por `NeutralToneMapping` (menos compressao)
- Adicionado `OutputPass` (corrige gamma do bloom)
- PointLight `decay=0` (artistico, sem falloff fisico)
- Exposure 1.2 → 1.5

### Bug: Hex invalidos
**Causa**: `0xddd`, `0x777`, `0xaaa` sao hex de 3 digitos (invalidos).
**Solucao**: Corrigido para `0xEEEEEE`, `0x777777`, `0xaaaaaa`.
