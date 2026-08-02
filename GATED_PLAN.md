# 3D Printing Scaffold — Gated Execution Plan

> **For the scaffold-3d-printing project terminal:** This is a gated execution plan. Complete each phase, then wait for user approval before proceeding to the next phase.

---

## Context

You are building the **first consumer** of AiTer's Extension Host mechanism (merged in PR #1). Your job is to create a 3D printing scaffold that demonstrates the extension mechanism end-to-end: CAD file preview/edit + slicer integration + printer dispatch.

**AiTer Extension Host provides:**
- 7 contribution points: fileTypes, previewHandlers, editors, monacoLanguages, tabKinds, ipcChannels, shortcuts
- Built-in extension: preview-builtin (25 fileTypes, 10 Node renderer handlers)
- Extension discovery: `extensions/builtin/<id>/extension.json` + `extensions/user/<id>/extension.json`
- Manifest validation: hand-rolled validator (schemaVersion gate + POSIX path + .js/.mjs handler suffix + native module ban)
- Two renderer modes: `renderer: 'node'` (Node module handler) + `renderer: 'iframe'` (HTML entry)

**Reference implementation:** `extensions/builtin/preview-builtin/` in the AiTer repo.

---

## Phase 1: Research & Design (GATE 1)

**Goal:** Understand the 3D printing domain and design the scaffold's extension manifest.

**Tasks:**
1. **Research 3D file formats:**
   - STL (mesh, universal slicer input)
   - 3MF (ISO/IEC 25422:2025, modern STL replacement)
   - STEP/IGES (B-rep, mechanical engineering)
   - GLB/glTF (web rendering, AI generation output)
   - OpenSCAD (.scad, text-based CSG)
   - CadQuery (.py, Python B-rep)

2. **Research slicer integration:**
   - OrcaSlicer CLI (headless slicing, `--slice --export-gcode`)
   - PrusaSlicer CLI (same flags, inherited from PrusaSlicer)
   - CuraEngine CLI (JSON settings required)

3. **Research printer dispatch:**
   - OctoPrint REST API (`/api/files`, `/api/job`)
   - Klipper Moonraker REST API (`/api/printer`, `/api/files`)
   - Bambu Lab MQTT (reverse-engineered, MITM proxy required)

4. **Design the extension manifest:**
   - Which fileTypes to register (STL, 3MF, STEP, GLB, SCAD, CadQuery)
   - Which previewHandlers to register (three.js mesh viewer for STL/3MF/GLB, Monaco editor for SCAD/CadQuery)
   - Which editors to register (Monaco with OpenSCAD syntax, CadQuery Python)
   - Which monacoLanguages to register (OpenSCAD monarch grammar)
   - Which ipcChannels to register (slicer:slice, printer:dispatch, printer:monitor)

5. **Write the design doc:** `docs/superpowers/specs/2026-08-02-3d-printing-scaffold-design.md`

**Deliverable:** Design doc committed to git. Report to user: "Phase 1 complete — design doc ready for review."

---

## Phase 2: Extension Implementation (GATE 2)

**Goal:** Implement the extension manifest + preview handlers + editors.

**Tasks:**
1. **Create extension manifest:** `extension.json`
   - Register 6 fileTypes: stl, 3mf, step, glb, scad, cadquery
   - Register 2 previewHandlers: mesh-viewer (iframe, three.js), cad-editor (monaco, custom)
   - Register 2 editors: scad-editor (monaco, openscad), cadquery-editor (monaco, python)
   - Register 1 monacoLanguage: openscad (monarch grammar)
   - Register 3 ipcChannels: slicer:slice, printer:dispatch, printer:monitor

2. **Implement mesh viewer (iframe):** `handlers/mesh-viewer.html`
   - three.js r185 (latest stable)
   - STL/3MF/GLB loader (three-stdlib loaders)
   - ClippingGroup for cross-section visualization
   - Rotation/zoom/measure controls
   - Unit display (mm/inch toggle)

3. **Implement CAD editor (Monaco):** `handlers/cad-editor.ts`
   - Monaco with OpenSCAD custom syntax (monarch grammar)
   - CadQuery Python syntax (standard Python monarch)
   - Customizer panel: parse `// Customizer` comments → render parameter UI
   - Live preview: on save, call `slicer:slice` IPC → show sliced result

4. **Implement slicer integration:** `handlers/slicer.ts`
   - Call OrcaSlicer CLI via `child_process.execFile`
   - Parse slicer output (G-code)
   - Return G-code path + metadata (layer count, print time, filament usage)

5. **Implement printer dispatch:** `handlers/printer.ts`
   - OctoPrint REST API client (upload G-code, start print, monitor progress)
   - Moonraker REST API client (same interface)
   - WebSocket for real-time progress updates

6. **Write tests:** `tests/`
   - Unit tests for each handler
   - Integration test: open STL file → preview shows mesh → slice → dispatch to printer

**Deliverable:** Extension implemented + tests passing. Report to user: "Phase 2 complete — extension ready for verification."

---

## Phase 3: Verification & Testing (GATE 3)

**Goal:** Verify the extension works end-to-end in AiTer.

**Tasks:**
1. **Install extension:** Copy `extension.json` + `handlers/` to `extensions/builtin/scaffold-3d-printing/` in AiTer repo
2. **Restart AiTer:** Verify extension loads (check ExtensionHost logs)
3. **Open STL file:** Verify mesh viewer renders correctly
4. **Open SCAD file:** Verify Monaco editor with OpenSCAD syntax works
5. **Edit CAD file:** Verify Customizer panel renders parameters
6. **Slice file:** Verify slicer integration works (G-code generated)
7. **Dispatch to printer:** Verify printer dispatch works (OctoPrint/Moonraker)
8. **Run full test suite:** `npm run test` in AiTer repo (must pass, no regressions)

**Deliverable:** All verification steps passing. Report to user: "Phase 3 complete — extension verified end-to-end."

---

## Phase 4: Publish (GATE 4)

**Goal:** Publish the scaffold to the AiTer scaffold marketplace.

**Tasks:**
1. **Create README.md:** Usage instructions, installation, features
2. **Create CHANGELOG.md:** Version history
3. **Tag release:** `git tag v0.1.0`
4. **Push to GitHub:** `git push origin main --tags`
5. **Publish to scaffold marketplace:** `aiter scaffold publish`
6. **Create PR:** Merge scaffold to AiTer's `extensions/builtin/` directory

**Deliverable:** Scaffold published to marketplace. Report to user: "Phase 4 complete — scaffold published."

---

## Key Constraints

- **Cross-platform:** All path handling must use POSIX forward slash in manifest, `path.join` in main process, `split(/[/\\]/)` in renderer
- **No native modules:** M1 phase bans native module dependencies (`.node` binaries)
- **Sandbox:** iframe preview uses existing sandbox, IPC bridge uses existing allowlist
- **Behavior preservation:** Existing AiTer preview/edit flows must not regress
- **Type safety:** All new code must pass `npm run type-check` and `npm run lint`

---

## Reference Files

- AiTer Extension Host spec: `docs/superpowers/specs/2026-07-23-extension-host-design.md`
- AiTer Extension Host plan: `docs/superpowers/plans/2026-08-02-extension-host.md`
- Built-in extension example: `extensions/builtin/preview-builtin/`
- Hello-world extension example: `extensions/builtin/example-text-styler/`

---

## Reporting

After each phase, report to the user via `aiter mail` with:
1. Phase completion status
2. Key decisions made
3. Any blockers or issues
4. Request approval to proceed to next phase

**Topic:** `scaffold-3d-printing:dev:3d-printing-scaffold`
