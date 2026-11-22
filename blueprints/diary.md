# learn_AI_CAD_design: Project Diary & Engineering Narrative

> **Rule Reference**: `RULE-DIARY-001` (`dvs_skills/rules/project_diary.md`)  
> **Status**: **Append-Only & Strictly Immutable** (Historical Record)  
> **Target Media**: Source Material for the *Acquired*-Style Project Retrospective Podcast  

---

## Day 1 — 2026-11-21: Origins — Locking the Headless Remote-SSH Tier 1 Python B-Rep & MCP CAD Plan

### 1. The Goal & Hypothesis
The goal of `learn_AI_CAD_design` is to build a 100% headless, open-source, script-reproducible AI-assisted 3D CAD and 3D-printing pipeline on Ubuntu 26.04 LTS (`Python 3.14.4`)—supporting interactive 3D rotation, zooming, and cross-sectioning directly inside Antigravity over Remote-SSH, covering involute gears, printable screw threads, and KiCad `.step` PCB enclosures (`threaded_fit_gauge` -> `parametric_gear_drive` -> `pcb_snapfit_enclosure`) before applying the workflow to microphone array frames and acoustic baffles (`mic_arr_v2`, `mic_arr_spatial`, `realtime_anc`).

Our working hypothesis is that an AI agent fails when asked to drive brittle GUI click-ops chains or emit raw `.stl` polygon coordinates, and succeeds reliably when generating parameterized B-Rep `build123d` scripts behind an MCP code/inspection loop (`build123d-mcp`), standard gear/thread libraries (`py-gearworks`, `bd-warehouse`), automated solid collision checks (`intersect().volume == 0`), headless `PrusaSlicer` printability verification, and interactive in-Antigravity WebGL 3D inspection (`o3dv-viewer` + `OCP CAD Viewer`).

### 2. Actions & Technical Decisions
* Audited the host workstation (`Ubuntu 26.04.1 LTS`, `Python 3.14.4`, `NVIDIA GeForce GTX 1070 with Max-Q Design 8192 MiB`, `snap` runtime with no `flatpak`, `docker`, or `wine` installed) and verified APT + PyPI package availability.
* Surveyed all 8 major CAD ecosystems (`build123d`, `OpenSCAD + BOSL2`, `Onshape`, `FreeCAD 1.0/1.1`, `SolidCraft`, `Zoo/KCL`, `Autodesk Fusion`, `Blender`) and 6 CAD MCP servers (`build123d-mcp`, `openscad-mcp`, `freecad-mcp`, `onshape-mcp`, `solidcraft-cli mcp`, `fusion360-mcp`) in [cad_software_landscape_3dp.md](research/cad_software_landscape_3dp.md) and [cad_mcp_pros_cons_concerns.md](research/cad_mcp_pros_cons_concerns.md).
* Stripped out all structural/thermal FEA simulation (`gmsh`, `calculix-ccx`) and CNC/sheet-metal overhead so the pipeline focuses purely on FDM/SLA 3D printing.
* Locked the architecture on **Tier 1 (`Python 3.14`-only, `~430 MB` unpacked)**: `build123d 0.13.0`, `build123d-mcp 0.3.91`, `cadquery 2.8.0`, `cadquery-ocp 8.0.1.1.0`, `py-gearworks 0.0.24`, `bd-warehouse 0.4.0`, `manifold3d 3.5.5`, `trimesh 5.2.1`, `ocp-vscode 4.2.0`, `f3d 3.2.0`, `librsvg2-bin`, `prusa-slicer 2.9.4` CLI, pre-installed `xvfb`, and Antigravity extensions `nahueltaibo.o3dv-viewer` + `ocp-cad-viewer`.
* Published the finalized plan, research comparisons, `.gitignore`, root [readme.md](../readme.md), and this diary to a public GitHub repository while explicitly deferring `hw/` and `sw/` directory scaffolding until Stage 1 kickoff.

### 3. Failures, Anomalies & Rabbit Holes
* **Premature PCB-Filter Assumption:** Our very first draft excluded `Fusion 360`, `Onshape`, and `SolidCraft` automatically and included CNC/FEA checks; stepping back to evaluate the full 8-tool landscape for 3D printing + gears/screws clarified the true trade-offs.
* **The Ubuntu 26.04 / Remote-SSH GUI Wall:** `Autodesk Fusion` has no native Linux client; `freecad` is absent from Ubuntu 26.04 `apt` and `freecad-mcp` requires a live desktop Qt window whose main thread freezes on heavy helical thread booleans; `Onshape` lives outside Antigravity with a strict `2,500 API calls/year` Free limit and public-only files.
* **The Rust Kernel Thread+Fillet Bug:** `SolidCraft` (`tangency-labs/solidcraft`) offers an exciting built-in `solidcraft-cli mcp`, but its young `truck`/`solidcore` kernels still suffer documented multi-edge fillet hangs, bugs where adding modeled threads breaks subsequent fillets, and lack an involute gear library.
* **The `agentcad` `Python 3.14` Pin Trap:** Inspecting live PyPI metadata caught that `agentcad 0.8.0` pins `Python <3.13` (failing on Ubuntu 26.04's `Python 3.14.4`), whereas `build123d-mcp 0.3.91` (`<3.15,>=3.11`) installs cleanly.

### 4. Root Causes & Breakthrough Moments
* **In-Antigravity WebGL 3D Viewport Discovery:** Realized we do not need a heavy host X11 CAD GUI over Remote-SSH—`nahueltaibo.o3dv-viewer` renders `.step`/`.3mf` files natively inside Antigravity tabs with auto-reload, while `ocp-cad-viewer` (`port 3939`) adds a part tree and live X/Y/Z cross-section sliders to inspect internal gear backlash and screw engagement.
* **PyPI Wheel AST Audit Eliminated Dual-Engine Complexity:** Inspecting the actual `bd-warehouse 0.4.0` and `py-gearworks 0.0.24` wheels on PyPI proved pure Python already ships `SpurGear`, `HelicalGear`, `RackGear`, `Worm`/`WormWheel`/`WormGear`, `Sprocket`, `BevelGear`, `PlanetaryGearset`, `SpurRingGear`, `HelicalRingGear`, `CycloidGear`, `IsoThread`, `AcmeThread`, `MetricTrapezoidalThread`, and `PlasticBottleThread`—making **Tier 1 (`Python 3.14`-only)** sufficient without splitting the stack across `OpenSCAD`/`BOSL2` (which cannot export `.step`).

### 5. Daily Verdict & Key Metrics
* **Status:** Plan (`v2.1.0`) locked on **Tier 1 (`~430 MB` unpacked)** and published to GitHub (`public`); zero host packages installed and zero `hw/`/`sw/` scaffolding created yet.
* **Artifacts Published:** Root [readme.md](../readme.md), `.gitignore`, [overall.md](overall.md), [cad_software_landscape_3dp.md](research/cad_software_landscape_3dp.md), [cad_mcp_pros_cons_concerns.md](research/cad_mcp_pros_cons_concerns.md), and Day 1 of [diary.md](diary.md).
* **Deferred Roadmap:** `toolchain_setup` -> `threaded_fit_gauge` -> `parametric_gear_drive` -> `pcb_snapfit_enclosure`.
