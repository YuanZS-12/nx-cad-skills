---
name: nx-cad
description: Generate Siemens NXOpen Python journals from natural-language CAD specs. Use for NX-targeted parametric mechanical parts, assemblies, holes, ribs, bosses, fillets, chamfers, boolean operations, native .prt creation, and STEP export. This skill emits NXOpen Python only, never build123d/CadQuery/OCC.
---

> **Runtime**: Generated `.py` files are not executed locally.
> Copy the generated journal to a machine with Siemens NX installed, then run
> via NX: File -> Execute -> NX Open. Wrapper-mode journals also need the
> sibling `cadnx/` folder. Raw NXOpen high-fidelity journals do not use
> `cadnx/` unless the generated source explicitly imports it, which this mode
> should not do.
> On NX machines where Designcenter/NXOpen MCP tools are expected, start every
> task by checking whether the `dc_*` tools are exposed in the current agent
> session. If they are available, use MCP API-review mode for NXOpen lookup and
> static code review. Do not run NX through MCP, do not call `dc_run_journal`,
> and do not try to start NX; the user runs journals manually in Siemens NX.
> If MCP tools are unavailable, say so before continuing in static-only mode.

# NX CAD Generation

Provenance: maintained in [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad).
Use the installed local skill files as the runtime source of truth; the
repository link is only for provenance and release review.

## Purpose

Create NXOpen Python journals from natural-language CAD requirements. This skill
follows the text-to-cad CAD workflow style: natural-language brief, parametric
source, explicit assumptions, generated artifacts, and repair-loop discipline.
The implementation target is Siemens NXOpen Python, not build123d.

Generated journals under `models/` are task source outputs. Native `.prt` files
and `.step` exports are Siemens NX runtime artifacts; do not claim they exist,
loaded, saved, exported, or validated unless the user reports a manual NX run or
a downstream check inspects the actual NX-created artifact.

Generated files should be portable NX journals. They may use raw NXOpen
directly, or may use a small local wrapper package when wrapper operations are
the right fit:

```text
models/
    <model_name>.py
    cadnx/
        __init__.py
        builder.py
```

NXBuilder is optional. `cadnx.NXBuilder` is a local wrapper around official
Siemens `NXOpen` APIs for simple robust primitives, guards, and STEP export
convenience.

## Use This Skill When

Use this skill when the user asks for Siemens NX, NXOpen, UG NX, NX journal
code, NX-compatible Python CAD, NX `.prt` creation, STEP export through NX, or
mechanical CAD features such as holes, counterbores, countersinks, slots,
pockets, bosses, standoffs, ribs, gussets, fillets, chamfers, and boolean
operations.

Do not use this skill for local build123d generation, local STEP validation,
render-only concept art, CAM toolpaths, engineering certification, FEA
conclusions, architectural BIM, or freehand illustration.

## Default Assumptions

Use these defaults unless the user specifies otherwise:

- Units: millimeters.
- Origin: center of the main part footprint unless a mating interface suggests
  a clearer datum.
- Base plane: XY.
- Up/extrusion axis: positive Z.
- Output geometry: closed, positive-volume solids.
- Native output: `.prt` saved by NX during execution.
- Exchange output: `.step` exported next to the generated journal.
- Small plastic enclosure wall: 2.0-3.0 mm when unspecified.
- Cosmetic fillet: 1.0-3.0 mm when safe for local geometry.
- M3/M4/M5 normal clearance holes: 3.4/4.5/5.5 mm unless another standard is
  requested.
- Socket-head counterbores and tapped pilot holes should use `NXBuilder`
  standard metric helpers when possible instead of ad hoc cutter diameters.
- Industrial detail level: refined but conservative. Add practical details
  that fit the requested part class, such as chamfers, fillets, counterbores,
  tapped pilots, pockets, ribs, bosses, pads, alignment features, and lightening
  slots only when named parameters leave positive walls and edge distance.

Ask one focused clarification question only when missing information makes the
model impossible, fit-critical, safety-critical, or compliance-bound. Otherwise
proceed with explicit assumptions.

## Industrial Detail Policy

Generated parts should look like plausible industrial CAD, not just primitive
solid sketches. Classify the request before modeling and choose a conservative
detail budget:

- **Machined part**: use datum-like flat faces, chamfers on exposed edges,
  counterbores or countersinks for fasteners, tapped holes when requested,
  pockets, relief slots, and clear edge-distance guards.
- **Molded or enclosure part**: use consistent wall thickness, rounded outside
  edges, internal bosses, ribs, standoffs, lip/step features, and conservative
  screw clearances.
- **Structural bracket or mount**: use mounting pads, gussets/ribs, symmetric
  hole patterns, lightening slots when space allows, and reinforced local
  material around fasteners.
- **Shaft, pulley, spacer, or rotary part**: use axial datums, shoulders,
  through bores, set-screw or keyway-like features only when specified or safe
  to approximate, and chamfered ends.
- **Generic concept part**: add only low-risk refinements such as small
  chamfers, rounded corners, standard hole helpers, and named validation guards.

Do not add industrial details that contradict the user's function, consume
critical clearance, create zero-wall geometry, or require unsupported wrapper
operations. When a detail is inferred rather than requested, make it a named
parameter and report it as an assumption.

## Official NXOpen API Policy

All raw `NXOpen.*` classes, enums, builders, creators, collections, properties,
and method patterns must be grounded in the Siemens **NXOpen Python Reference
Guide 2512**:

`https://docs.sw.siemens.com/en-US/doc/209349590/PL20250429951538534.custom_api.nxopen_python_ref`

Use `references/official-nxopen-sources.md` as the local source map. Before
adding or revising raw NXOpen wrapper code, confirm the API object or pattern in
that official guide and update the source map with the relevant official page.
Do not rely on memory, forum snippets, translated examples, or non-Siemens
pages for raw NXOpen code shape.

This skill has two generation modes:

- **Wrapper mode** is the default when the user asks for `nx-cad` without
  requesting MCP-backed raw NXOpen. Use `NXBuilder` for simple robust
  primitives, common guards, and STEP export convenience.
- **Raw NXOpen high-fidelity mode** is required when the user asks to use MCP,
  `dc_server`, Designcenter, raw/bare `NXOpen`, `NXOpen lib`, or explicitly asks
  for high-fidelity NXOpen code. In this mode, generate direct `NXOpen.*`
  journal code, do not import or use `cadnx.NXBuilder`, and include
  `RAW_NXOPEN_HIGH_FIDELITY = True` plus either `MCP_API_REVIEW` or
  `STATIC_ONLY_NXOPEN_REVIEW` in the journal.

Raw NXOpen is therefore not a fallback for only special geometry. It is the
MCP-backed high-fidelity route for any part class when the prompt asks for that
route. Wrapper mode remains the stable default route when the prompt does not
request MCP-backed raw NXOpen.

Raw NXOpen is allowed when selected by the prompt route above or when explicit
static-only uncertainty is recorded for a repair or compatibility fixture.

When raw `NXOpen.*` code is written or revised and Designcenter/NXOpen MCP
tools are available, use MCP API-review mode and record MCP API-review
evidence. Use `dc_get_api_info` for raw classes, builders, enums, properties,
and methods whose exact shape appears in the journal. If MCP tools are
unavailable, state that before coding and continue in static-only mode with
explicit uncertainty.

When Designcenter/NXOpen MCP tools are expected or available, read
`references/mcp-runtime.md` before planning or coding. In MCP API-review mode,
use MCP discovery tools before generating or modifying NXOpen code, and use
`dc_get_api_info` before raw NXOpen calls. Do not call `dc_run_journal`, start
NX, or otherwise execute NX; the user runs journals manually in Siemens NX and
reports tracebacks or success output.

## Natural-Language Specs Only

Do not ask the user to provide JSON. Convert prose into an internal CAD brief
with dimensions, units, coordinate convention, features, output paths,
assumptions, and validation targets. Use `assets/design-brief-template.md` and
`references/natural-language-specs.md` for brief-writing patterns.

## External Catalog Parts

When an NX part or assembly includes named off-the-shelf actuators, servos,
motors, bearings, screws, connectors, electronics boards, or other purchasable
components, search `$step-parts` before creating simplified placeholder
geometry. If an exact or near-exact STEP model is available, prefer that
catalog part unless the task explicitly needs a simplified clearance envelope.
If the API is reachable and no suitable part is found, record the search miss
and then use a documented simplified envelope.

Maintain an external component ledger in the internal CAD-NX brief whenever
catalog or user-supplied STEP parts are involved. Record the part id/name,
source URL or local path, checksum when known, units assumption, placement
datum, transform, and whether the generated journal imports a real STEP part or
uses an envelope. See `references/external-step-parts.md`.

## Root Model

Keep these roots separate:

- **Skill source root**: `<repo>/skills/nx-cad`
- **Generated output root**: `<repo>/models`
- **NX runtime root**: the copied model folder on the Windows NX machine

Generated journals must not depend on absolute paths from this Mac workspace.
They should resolve outputs from `__file__` and export `.step` next to the
journal on the NX machine.

## Required Workflow

1. Classify the request: new part, assembly, modification, NX repair, or export
   issue.
2. Select generation mode before planning:
   - use **raw NXOpen high-fidelity mode** when the prompt requests MCP,
     `dc_server`, Designcenter, raw/bare `NXOpen`, `NXOpen lib`, or high
     fidelity NXOpen code;
   - otherwise use **wrapper mode** unless the user explicitly forbids
     `NXBuilder`.
3. Run the MCP discovery gate from `references/mcp-runtime.md` when
   Designcenter/NXOpen MCP tools are expected, the user mentions MCP, or the
   task is running on an NX machine:
   - if any `dc_*` API lookup tools are exposed, enter MCP API-review mode;
   - if none are exposed, state that before writing code and continue in
     static-only mode.
4. Load only the needed references:
   - official Siemens API source mapping:
     `references/official-nxopen-sources.md`
   - natural-language brief: `references/natural-language-specs.md`
   - design ledger: `references/design-ledger.md`
   - parameter and feature planning: `references/parameters.md`
   - NXOpen modeling calls: `references/nxopen-modeling.md`
   - raw NXOpen high-fidelity route:
     `references/raw-nxopen-quality.md`
   - advanced geometry roadmap:
     `references/advanced-geometry-roadmap.md`
   - aerospace casing/frame modeling:
     `references/aerospace-frame-modeling.md`
   - aerospace linkage and bellcrank modeling:
     `references/aerospace-linkage-modeling.md`
   - part-class quality:
     `references/part-class-quality.md`
   - validation reporting: `references/validation.md`
   - STEP export issues: `references/nxopen-export-step.md`
   - Designcenter/NXOpen MCP runtime:
     `references/mcp-runtime.md`
   - repair loop: `references/repair-loop.md`
   - repeated API failures: `references/nxopen-common-errors.md`
   - benchmark regression work: `references/benchmark-workflow.md`
   - catalog STEP components: `references/external-step-parts.md`
5. Create a concise internal CAD-NX brief: independent parameters, derived
   parameters, coordinate convention, industrial intent, manufacturing style,
   feature order, external component ledger, output name, assumptions, local
   static validation, official API source needs, and NX runtime validation
   targets. Use `references/design-ledger.md` when the part has raw NXOpen API
   use, nontrivial frames/datums, external parts, PMI, assemblies, or
   user-reported NX runtime failures.
6. Plan before coding:
   - primary solids before holes/cutouts;
   - functional and manufacturability details before cosmetic-only details;
   - booleans before cosmetic detail;
   - recognizable industrial parts checked against
     `references/part-class-quality.md`;
   - named external components resolved through `$step-parts` before simplified
     cylinders, boxes, or envelope geometry;
   - fillets/chamfers last and conservative;
   - in raw NXOpen high-fidelity mode, use direct NXOpen builders, collections,
     curves, sections, expressions, PMI, assembly, save, and export APIs as
     appropriate for the requested model;
   - in raw NXOpen high-fidelity mode, do not use `NXBuilder` operations;
   - in wrapper mode, use `NXBuilder` only when its operation directly matches
     the intended feature;
   - record MCP API-review evidence for raw NXOpen in the final response.
7. In MCP API-review mode, use at least one discovery tool
   (`dc_search`, `dc_semantic_search`, or `dc_lookup_pattern`) before writing
   the journal. Use `dc_get_api_info` before any raw NXOpen API or wrapper
   extension. If MCP tools are unavailable, do not fabricate MCP evidence.
8. Generate a single NXOpen Python journal:
   - wrapper mode uses `templates/nxopen_part_template.py`;
   - raw NXOpen high-fidelity mode uses
     `templates/raw_nxopen_part_template.py`.
9. Save the journal under `<repo>/models/<task_name>.py`.
10. Sync the runtime wrapper only for wrapper mode or when checking repository
   runtime freshness:
   `skills/nx-cad/scripts/sync-runtime --models-dir models`.
11. Run local static checks:
   `skills/nx-cad/scripts/check-journal models/<task_name>.py`.
   For newly generated industrial/mechanical journals, also run strict geometry
   budget checks:
   `skills/nx-cad/scripts/check-journal models/<task_name>.py --strict-geometry`.
12. In MCP API-review mode, follow `references/mcp-runtime.md`: use
   `dc_run_snippet` only for short API probes or static review snippets that do
   not launch full NX model generation. Do not run `dc_run_journal`, start NX,
   or attempt agent-side runtime execution.
13. Do not claim NX execution. Tell the user exactly which journal to copy to
   the NX machine. For wrapper mode, also copy the sibling `cadnx/` folder. For
   raw NXOpen high-fidelity mode, do not require `cadnx/`.
14. If MCP API-review tools are unavailable, do not claim MCP review. Tell the
   user exactly which journal to copy to the NX machine, and whether `cadnx/`
   is needed for the selected mode.
15. If the user reports an NX traceback, repair the smallest responsible
   section of either the generated journal or `cadnx/builder.py`, sync
   `models/cadnx/`, use MCP API-review tools to inspect suspect NXOpen APIs
   when available, and ask the user to rerun manually in Siemens NX.

## NXOpen Code Rules

- Use `from cadnx import NXBuilder` and `b = NXBuilder()` only in wrapper mode.
- In raw NXOpen high-fidelity mode, do not import `cadnx` or instantiate
  `NXBuilder`; use direct `NXOpen.*` APIs and include
  `RAW_NXOPEN_HIGH_FIDELITY = True`.
- Do not import build123d, CadQuery, OCC, FreeCAD, OpenSCAD, or local CAD
  kernels.
- Wrapper-assisted generated files must define `build(output_path: str = None)`
  and end `build()` with `b.export_step(output_path)`.
- Raw NXOpen generated files must define a clear `main()` entry point, explicit
  required NXOpen submodule imports, work-part handling, runtime diagnostics,
  and save/export handling when export is part of the task.
- Raw NXOpen generated files that use builders must wrap builder lifecycles in
  `try/finally` and call `Destroy()`. Load/status objects returned by part
  creation or open calls must be disposed when present.
- If an output path is generated, set it to the generated journal basename with
  `.step`.
- Put all named dimensions near the top of `build()` or `main()`.
- Separate independent prompt parameters from derived dimensions.
- Derive centers, pitches, cut depths, and repeated feature positions from
  named parameters instead of duplicating numeric constants.
- Use floats or numbers only for dimensions; `NXBuilder` normalizes types for
  NXOpen.
- Use named parameter guards for tight mechanical geometry:
  `b.require_positive(...)`, `b.require_min_wall(...)`, and
  `b.require_edge_distance(...)`.
- For aerospace linkage, bellcrank, control-arm, lug, and clevis parts, use
  semantic helpers such as `b.bearing_seat(...)`, `b.bushing_boss(...)`, and
  `b.clevis(...)` instead of only raw cylinders and holes.
- For complex patterned parts, especially aerospace cases, frames, struts, and
  housings, add `b.require_feature_budget(...)` near the parameter guards with
  named budgets for boolean operations, micro holes, and patterned features.
- Use named boolean robustness parameters such as `through_overcut`,
  `feature_overlap`, and `cutter_overlap` so subtractive and united tools form
  real volume intersections instead of tangent or face-only contact.
- In wrapper mode, prefer simple robust primitives and booleans over fragile
  low-level NXOpen builder sequences in generated journals.
- In raw NXOpen high-fidelity mode, use reviewed native NXOpen builders and
  collections directly; do not route the model through wrapper primitives.
- Keep `models/cadnx/` synchronized with `skills/nx-cad/cadnx/` after
  generating or changing a wrapper-mode output script.
- `b.slot_cut()` currently requires both `axis` and `direction` to be
  axis-aligned unit vectors. Do not generate radial, diagonal, or trig-derived
  slot directions. For angled cosmetic witness marks, traceability marks, or
  nonfunctional surface details, compose conservative `box()` cutters or
  `rectangular_pocket()` calls; for functional angled slots, extend
  `NXBuilder` first and add a static check before generating the journal.

## Supported Wrapper Operations

Prefer these `NXBuilder` calls:

- `b.box(length, width, height, origin=(x, y, z))`
- `b.oriented_box(length, width, height, center=(x, y, z), u_axis=(...), v_axis=(...), w_axis=(...))`
- `b.rounded_box(length, width, height, radius, origin=(x, y, z))`
- `b.cylinder(diameter, height, origin=(x, y, z), axis=(0, 0, 1))`
- `b.hole(diameter, depth, position=(x, y, z), direction=(0, 0, -1))`
- `b.annular_groove(target, outer_diameter, inner_diameter, depth, position=(x, y, z), direction=(0, 0, -1), cutter_overlap=...)`
- `b.bearing_seat(target, bore_diameter, seat_diameter, through_depth, seat_depth, position=(x, y, z), direction=(0, 0, -1), cutter_overlap=...)`
- `b.bushing_boss(target, boss_diameter, boss_height, bore_diameter, center=(x, y, z), axis=(0, 0, 1), feature_overlap=..., through_overcut=...)`
- `b.clevis(target, length, width, ear_thickness, gap, pin_hole_diameter, center=(x, y, z), u_axis=(...), v_axis=(...), w_axis=(...))`
- `b.counterbore_hole(target, hole_diameter, hole_depth, counterbore_diameter, counterbore_depth, position, direction)`
- `b.screw_clearance_hole(target, "M5", depth, position=(x, y, z), direction=(0, 0, -1))`
- `b.tapped_hole(target, "M5", depth, position=(x, y, z), direction=(0, 0, -1))`
- `b.socket_head_counterbore_hole(target, "M5", depth, position=(x, y, z), direction=(0, 0, -1))`
- `b.countersink_hole(target, "M5", depth, position=(x, y, z), direction=(0, 0, -1))`
- `b.slot_cut(target, length, width, depth, center, axis=(1, 0, 0), direction=(0, 0, -1))`
- `b.rectangular_pocket(target, length, width, depth, center, direction=(0, 0, -1))`
- `b.linear_pattern_points(start, count, spacing, direction=(1, 0, 0))`
- `b.circular_pattern_points(center, radius, count, start_angle_degrees=0.0)`
- `b.require_positive(base_length=base_length, plate_thickness=plate_thickness)`
- `b.require_feature_budget(boolean_operations=..., micro_holes=..., patterned_features=...)`
- `b.require_min_wall(local_width, feature_diameter, min_wall=1.5, label="counterbore")`
- `b.require_edge_distance(edge_distance, feature_diameter, min_ratio=1.0, label="mounting hole")`
- `b.boolean_subtract(target, tool)`
- `b.boolean_unite(target, tool)`
- `b.fillet(edges, radius)`
- `b.chamfer(edges, offset)`
- `b.get_all_edges(feature)`
- `b.get_top_edges(feature)`
- `b.get_bottom_edges(feature)`
- `b.get_edges_by_axis(feature, axis=(0, 0, 1))`
- `b.get_edges_near(feature, point=(x, y, z), tolerance=...)`
- `b.get_edges_in_box(feature, min_xyz=(...), max_xyz=(...))`
- `b.export_step(output_path)`
- External STEP import into the work part: `b.import_step_part(...)`.
- Guarded assembly component contract, not yet production placement:
  `b.add_component(...)` and `b.place_component(...)`

If a model needs an unsupported feature, either compose it from supported
solids/booleans or extend `cadnx.NXBuilder` first, then sync the runtime.
In raw NXOpen high-fidelity mode, unsupported wrapper features should become
direct NXOpen operations only after MCP API-review or explicit static-only
uncertainty. External assembly component placement may use raw NXOpen APIs in
that mode when the API shape is reviewed and the journal reports runtime
uncertainty honestly.

## Repair Loop

When NX reports an error:

1. Identify whether the failure is import/path, work part creation, geometry
   builder, boolean, edge selection, fillet/chamfer, save, or STEP export.
2. Patch the shared wrapper when the failure is an NXOpen compatibility issue.
3. Patch the generated journal when the feature plan is fragile or invalid.
4. Run `skills/nx-cad/scripts/sync-runtime --models-dir models`.
5. Run `skills/nx-cad/scripts/check-journal models/<task_name>.py` locally.
   For repairs involving booleans, also run
   `skills/nx-cad/scripts/check-journal models/<task_name>.py --strict-geometry`.
6. If Designcenter/NXOpen MCP tools are available, use API-review tools to
   inspect suspect NXOpen calls, signatures, patterns, or wrapper behavior.
7. Do not rerun the journal with `dc_run_journal`, do not start NX, and do not
   attempt agent-side NX runtime execution.
8. Tell the user exactly which files to copy to the NX machine and what the next
   manual NX rerun should prove.

## Non-Negotiables

- Output source must be NXOpen-compatible Python, not build123d.
- Generated journals must run inside Siemens NX, not normal Python.
- Keep `.py` journal and `cadnx/` wrapper together when moving wrapper-mode
  journals to NX. Raw NXOpen high-fidelity journals should not require
  `cadnx/`.
- Never claim a journal has run in NX unless the user reports a manual Siemens
  NX run.
- Do not call `dc_run_journal`; runtime evidence comes from the user's manual
  Siemens NX run.
- Never imply MCP was used unless the final answer lists the actual `dc_*`
  tools that were called.
- Never start or run NX through MCP. MCP is for NXOpen API lookup, signature
  checking, pattern lookup, and code review.
- Never hardcode Mac workspace paths into generated journals.
- Report only checks that actually ran.

## Progressive References

Load these files only when their trigger applies:

- `assets/design-brief-template.md` - internal brief scaffold.
- `references/official-nxopen-sources.md` - Siemens NXOpen Python Reference
  Guide pages used as the source for journal structure, builder calls, and STEP
  export APIs.
- `references/natural-language-specs.md` - prose-to-CAD brief conversion.
- `references/design-ledger.md` - internal ledger for generation mode,
  parameters, datums, API evidence, output paths, and NX runtime checks.
- `references/parameters.md` - parameter naming, derived dimensions, feature
  grouping, and common parameter failures.
- `references/nxopen-modeling.md` - journal structure, wrapper operations,
  coordinate conventions, and robust modeling rules.
- `references/raw-nxopen-quality.md` - direct NXOpen high-fidelity route used
  when the prompt asks for MCP, `dc_server`, Designcenter, raw/bare `NXOpen`,
  `NXOpen lib`, or high-fidelity NXOpen code.
- `references/advanced-geometry-roadmap.md` - staged research and acceptance
  gates for revolve, curves/splines, loft, sweep, and blade helpers.
- `references/aerospace-frame-modeling.md` - special guidance for aerospace
  casings, rear frames, ring frames, strut rings, diffuser frames, and other
  circular patterned high-detail parts.
- `references/aerospace-linkage-modeling.md` - special guidance for
  flight-control bellcranks, control arms, lugs, bushing bosses, and clevis
  interfaces.
- `references/part-class-quality.md` - expected features and forbidden
  primitive-only shortcuts for recognizable industrial part classes.
- `references/external-step-parts.md` - using `$step-parts`, recording an
  external component ledger, and deciding between real STEP parts and
  simplified envelopes.
- `references/validation.md` - local static gates, NX runtime validation
  boundaries, and final report contents.
- `references/repair-loop.md` - failure classification and source-vs-wrapper
  repair policy.
- `references/nxopen-export-step.md` - `.prt` save and STEP export behavior.
- `references/mcp-runtime.md` - MCP discovery gate, API-review-vs-static mode
  selection, required MCP evidence, tool triggers, API research flow, snippet
  probes, and the no-agent-runtime-execution policy.
- `references/nxopen-common-errors.md` - known NXOpen Python compatibility
  errors from this user's NX environment.
- `references/nx-runtime-feedback-ledger.md` - user- or automation-reported NX
  runtime results, root causes, patches, and follow-up checks.
- `references/benchmark-workflow.md` - repo benchmark prompts, local static
  gates, NX runtime reporting, and repair policy.

Final responses should include generated `.py` path, selected mode, whether
`models/cadnx/` is synced or not required, syntax checks actually run,
Designcenter/NXOpen MCP API-review result when available, the exact `dc_*`
tools called, assumptions, exactly what to copy to the NX machine, and a clear
statement that the user must run the journal manually in Siemens NX for runtime
validation.
