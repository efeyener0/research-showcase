# The Meridian: 5-Layer Engine Monorepo, Biophysical Anatomy (`meridian::anat` v2), Fight World & Lyric (`1.0`)

**The Meridian** unifies four domain-specific procedural engines and **10+ custom domain-specific languages (DSLs)** under a strict 5-layer Directed Acyclic Graph (DAG) architecture.

> **Repository Status:** The unified monorepo (`efeyener0/the-meridian-monorepo`) and standalone domain repositories (`Procedurel-Anatomy`, `lyrical-legend`) are maintained privately. This document details the architectural invariants, cross-domain contract design, and mathematical solvers governing each subsystem.

---

## 1. 5-Layer Monorepo DAG Architecture

To prevent circular dependencies and cross-domain coupling across large procedural and runtime systems, `The-Meridian` enforces a strict 5-layer build and dependency hierarchy:

```text
The-Meridian/
├── third_party/raylib/            # L0: Single canonical raylib-internal-fork
├── foundation/                    # L0–L1: Shared low-level mathematical & compute infrastructure
│   ├── core/                      #   L0: Diagnostics, SourceLocation, physical units, RNG, Eigen wrapper
│   ├── geom/                      #   L1: Manifold mesh, RMF curves, sweep, SDF, 6-RoSy / Morse-Smale fields
│   ├── ir/                        #   L1: Content-addressed Binary Merkle DAG & SHA-256 cook cache
│   ├── gltf/                      #   L1: Unified GLB / glTF 2.0 serializer & reader
│   └── gpu/                       #   L1: CUDA / WebGPU / Work-Stealing CPU compute abstraction
├── lang/core/                     # L2: Shared lexer/parser combinators, unit literals & diagnostics
├── domains/                       # L3: Strictly isolated domain engines (ZERO domain-to-domain #include/link)
│   ├── anatomy/                   #   meridian::anat v2 (.anth / .antsrc / .antsc biophysical anatomy engine)
│   ├── atelier/                   #   Atelier V (.a5 parallel solid modeling & CUDA path-tracing engine)
│   ├── lyric/                     #   Lyric 1.0.0 (.lyr spectral music & acoustic synthesis engine)
│   └── fw/                        #   Fight World runtime engine (canon-core, ui, map, shader, weapon-anim)
├── bridges/                       # L4a: Cross-domain contract adapters & round-trip verifiers (C1–C6)
│   ├── anatomy-to-fw/             #   Body GLB + 106-segment FWTensor rig adapter
│   └── atelier-to-fw/             #   Equipment GLB + SE(3) attachment sockets + weapon collision capsules
└── apps/                          # L4b: Runtime executables & unified CLI compiler drivers
```

### Core Architectural Invariants
1. **Strict `L3` Domain Isolation:** `domains/anatomy`, `domains/atelier`, `domains/lyric`, and `domains/fw` are forbidden from `#include`-ing or linking against one another directly. All cross-domain exchange occurs exclusively through `foundation/*` (`L0–L1`) and verified `bridges/*` (`L4a`) contracts (`C1–C6`).
2. **Legacy Quarantine & Clean-Sheet Migration:** Four incompatible early anatomy prototypes were consolidated by quarantining v1 data under a non-compiled donor tree and architecting `meridian::anat` v2 from a clean mathematical specification with zero hardcoded prototype magic numbers.

---

## 2. Biophysical & Procedural Humanoid Anatomy (`meridian::anat` v2 / `.antsc`)

`meridian::anat` v2 synthesizes anatomically grounded humanoid skeletons, musculature, subcutaneous tissue, and manifold skin from declarative `.anth` / `.antsrc` / `.antsc` specifications:

- **Log-Space Allometric Dimensional Solver:** Solves coupled relational proportion graphs (`parts -> templates -> configs`) in logarithmic coordinate space, ensuring positive-definite bone/segment scaling and biological allometric invariants across diverse somatotypes (`GynoidHumanoid`, `AndroidHumanoid`).
- **Topological Branching Macros (`TriHubSpine` & `CatalanAutopod<R,P>`):** Models axial skeletal hubs (pelvis, ribcage, cranium) and appendicular hand/foot branching ($1 \to 2 \to 5 \to 14$ phalangeal/ray hierarchies) via parameterized Catalan tree topologies.
- **Isochoric Superelliptic Multi-Fiber Musculature:** Represents skeletal muscles as asymmetric superelliptic cross-section sweeps along origin-to-insertion action lines. Under joint articulation, muscle bellies deform under an **isochoric (volume-preserving)** constraint ($\lambda_1 \lambda_2 \lambda_3 = 1$), coupling longitudinal shortening to transverse belly bulging.
- **Sparse $8 \times 8 \times 8$ `BrickGrid` SDF Skin & 6-RoSy Retopology:** Evaluates smooth multi-layer union/blend fields across narrow-band sparse $8^3$ voxel bricks and extracts curvature-aligned meshes via intrinsic hexagonal / 6-RoSy and Morse-Smale field retopology.
- **Verification Telemetry:** Full anatomical build, topological invariant checks, and allometric solver suites execute in **0.75 s** (`4/4` test suites passing).

---

## 3. Fight World Runtime Engine (`domains/fw`)

A modular cross-platform runtime engine targeting Desktop, Android (JNI), and WebAssembly:
- **`canon-core`:** 106-segment articulated biomechanical rig, contact and capsule collision solvers, energy/flow graph (`EnergyBodyGraph`), and 21 standalone verification probe executables.
- **6+ Custom Game DSLs:** Dedicated declarative and behavioral languages for animation choreography (`fwani`), type schemas (`fwtype`), configuration (`fwconfig`), combat scripting (`fwscript`), material/UI styling (`fwms` / `fwshader`), and CSG spatial map authoring (`fwmap` / `fwobject`).

---

## 4. Lyric 1.0 (`domains/lyric`, `.lyr`) — Spectral Music & Acoustic Synthesis DSL

**Lyric** is a zero-dependency domain-specific language and acoustic synthesis engine that compiles compact musical score and spectral physics notation directly into **16/24-bit or float WAV audio at 44.1, 48, or 96 kHz**.

```lyric
import std.instruments

tempo 84
meter 3/4
key D dorian

spectrum glass = harmonic(1..24) |> amp(k => k^-1.3) |> stretch(0.0008)
               |> formant(1.2kHz, q: 4, gain: 6dB) |> decay(k => 4s / k^0.7)

let motif  = [ mf D4 e F4 A4 q G4 e. F4 s | E4 h. ]
let rising = seq(for i in 0..3 => motif |> step(i))

score {
    track lead : violin, pan: -0.2 { rising + (motif |> retrograde |> transpose(-M2)) }
    track pad  : glass,  pan:  0.2 { [ pp D3:m7 h. G2:7 C3:maj7 A2:7b9 ] * 2 }
}
```

### Key Capabilities
- **Exact Rational Rhythm & Diatonic Spelling:** Represents metric durations as exact rationals and preserves enharmonic/diatonic interval algebra (`F4 - M2 = Eb4`).
- **Microtonal & Historical Tunings:** Native support for `edo(n)`, Pythagorean, meantone, and 5-limit just intonation (distinguishing enharmonic comma shifts such as $C\sharp \neq D\flat$).
- **Spectral Sound Design & Spectral Harmony:** Ships with **24 spectral instrument models** written in Lyric itself and **~25 spectral operators** (inharmonic stretch, resonant formants, per-partial decay curves, spectral morphing, and chord-to-spectrum / spectrum-to-chord duality).
- **DSP Engine:** Alias-guarded additive phasor oscillator banks, Short-Time Fourier Transform (STFT) filtered noise synthesis, and high-band wavelet denoising.
