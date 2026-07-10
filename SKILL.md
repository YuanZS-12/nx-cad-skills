---
name: nx-cad
description: Generate Siemens NXOpen Python journals from natural-language CAD specs. Use for NX-targeted parametric mechanical parts, assemblies, holes, ribs, bosses, fillets, chamfers, boolean operations, native .prt creation, and STEP export. This skill emits NXOpen Python only, never build123d/CadQuery/OCC.
---

> **Runtime**: Generated `.py` files are not executed locally.
> Copy the generated journal plus its sibling `cadnx/` folder to a machine with
> Siemens NX installed, then run via NX: File -> Execute -> NX Open. NX builds
> the model, saves a native `.prt`, and exports a `.step`.
> On NX machines where Designcenter/NXOpen MCP tools are expected, start every
> task by checking whether the `dc_*` tools are exposed in the current agent
> session. If they are available, use MCP API-review mode for NXOpen lookup and
> static code review. Do not run NX through MCP, do not call `dc_run_journal`,
> and do not try to start NX; the user runs journals manually in Siemens NX.
> If MCP tools are unavailable, say so before continuing in static-only mode.

# NX CAD Generation

## Purpose

Create NXOpen Python journals from natural-language CAD requirements. This skill
follows the text-to-cad CAD workflow style: natural-language brief, parametric
source, explicit assumptions, generated artifacts, and repair-loop discipline.
The implementation target is Siemens NXOpen Python, not build123d.

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

`cadnx.NXBuilder` is optional. It is a local wrapper around official Siemens
`NXOpen` APIs for simple robust primitives, guards, and STEP export convenience.

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

Raw NXOpen is allowed when it is the better modeling path for the requested
part. NXBuilder is optional: use it for simple robust primitives, common
guards, and STEP export convenience, but do not force high-fidelity geometry
through wrapper operations when direct NXOpen builders are clearer or more
capable.

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
2. Run the MCP discovery gate from `references/mcp-runtime.md` when
   Designcenter/NXOpen MCP tools are expected, the user mentions MCP, or the
   task is running on an NX machine:
   - if any `dc_*` API lookup tools are exposed, enter MCP API-review mode;
   - if none are exposed, state that before writing code and continue in
     static-only mode.
3. Load only the needed references:
   - official Siemens API source mapping:
     `references/official-nxopen-sources.md`
   - natural-language brief: `references/natural-language-specs.md`
   - parameter and feature planning: `references/parameters.md`
   - NXOpen modeling calls: `references/nxopen-modeling.md`
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
4. Create a concise internal CAD-NX brief: independent parameters, derived
   parameters, coordinate convention, industrial intent, manufacturing style,
   feature order, external component ledger, output name, assumptions, local
   static validation, official API source needs, and NX runtime validation
   targets.
5. Plan before coding:
   - primary solids before holes/cutouts;
   - functional and manufacturability details before cosmetic-only details;
   - booleans before cosmetic detail;
   - recognizable industrial parts checked against
     `references/part-class-quality.md`;
   - named external components resolved through `$step-parts` before simplified
     cylinders, boxes, or envelope geometry;
   - fillets/chamfers last and conservative;
   - choose the highest-fidelity NXOpen modeling route that matches the
     requested geometry;
   - use `NXBuilder` only when its operation directly matches the intended
     feature;
   - use raw NXOpen builders directly when wrapper operations would downgrade
     the part into misleading primitive approximations;
   - record MCP API-review evidence for raw NXOpen in the final response.
6. In MCP API-review mode, use at least one discovery tool
   (`dc_search`, `dc_semantic_search`, or `dc_lookup_pattern`) before writing
   the journal. Use `dc_get_api_info` before any raw NXOpen API or wrapper
   extension. If MCP tools are unavailable, do not fabricate MCP evidence.
7. Generate a single NXOpen Python journal using
   `templates/nxopen_part_template.py`.
8. Save the journal under `<repo>/models/<task_name>.py`.
9. Sync the runtime wrapper:
   `skills/nx-cad/scripts/sync-runtime --models-dir models`.
10. Run local static checks:
   `skills/nx-cad/scripts/check-journal models/<task_name>.py`.
   For newly generated industrial/mechanical journals, also run strict geometry
   budget checks:
   `skills/nx-cad/scripts/check-journal models/<task_name>.py --strict-geometry`.
11. In MCP API-review mode, follow `references/mcp-runtime.md`: use
   `dc_run_snippet` only for short API probes or static review snippets that do
   not launch full NX model generation. Do not run `dc_run_journal`, start NX,
   or attempt agent-side runtime execution.
12. Do not claim NX execution. Tell the user exactly which journal and sibling
   `cadnx/` folder to copy to the NX machine and what the manual NX rerun
   should prove.
13. If MCP API-review tools are unavailable, do not claim MCP review. Tell the
   user exactly which journal and sibling `cadnx/` folder to copy to the NX
   machine.
14. If the user reports an NX traceback, repair the smallest responsible
   section of either the generated journal or `cadnx/builder.py`, sync
   `models/cadnx/`, use MCP API-review tools to inspect suspect NXOpen APIs
   when available, and ask the user to rerun manually in Siemens NX.

## NXOpen Code Rules

- Use `from cadnx import NXBuilder` and `b = NXBuilder()` when wrapper
  operations directly match the intended geometry. Use raw NXOpen directly when
  it produces better NX geometry and MCP API-review evidence is available or
  static-only uncertainty is explicitly recorded.
- Do not import build123d, CadQuery, OCC, FreeCAD, OpenSCAD, or local CAD
  kernels.
- Wrapper-assisted generated files must define `build(output_path: str = None)`
  and end `build()` with `b.export_step(output_path)`.
- Raw NXOpen generated files must define a clear `main()` entry point, explicit
  required NXOpen submodule imports, work-part handling, runtime diagnostics,
  and save/export handling when export is part of the task.
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
- Prefer simple robust primitives and booleans over fragile low-level NXOpen
  builder sequences in generated journals.
- Avoid generated calls to raw NXOpen APIs unless `NXBuilder` lacks the needed
  operation and the reference confirms the pattern.
- Keep `models/cadnx/` synchronized with `skills/nx-cad/cadnx/` after
  generating or changing an output script.
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
External assembly component placement is not a raw journal operation yet. Use
the guarded `NXBuilder` contract documented in
`references/external-step-parts.md`, and only replace those guards with working
NXOpen calls after official Siemens API research and real NX runtime
validation.

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
- Keep `.py` journal and `cadnx/` wrapper together when moving to NX.
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
- `references/parameters.md` - parameter naming, derived dimensions, feature
  grouping, and common parameter failures.
- `references/nxopen-modeling.md` - journal structure, wrapper operations,
  coordinate conventions, and robust modeling rules.
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

Final responses should include generated `.py` path, `models/cadnx/` sync
status, syntax checks actually run, Designcenter/NXOpen MCP API-review result
when available, the exact `dc_*` tools called, assumptions, exactly what to copy
to the NX machine, and a clear statement that the user must run the journal
manually in Siemens NX for runtime validation.
