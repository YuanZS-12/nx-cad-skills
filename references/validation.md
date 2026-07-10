# NX CAD Validation

Read this before reporting success for generated NXOpen journals.

## Principle

Local validation can prove source structure and runtime freshness. It cannot
prove Siemens NX execution, native `.prt` save, or STEP export unless the
journal was actually run inside NX and the user reports the result. Agents do
not start NX and do not call `dc_run_journal`; NX runtime validation is
user-run only.

## Local Static Validation

Always run after generating or modifying a journal:

```bash
skills/nx-cad/scripts/sync-runtime --models-dir models
skills/nx-cad/scripts/check-journal models/<journal>.py
```

These checks prove:

- the journal is valid Python syntax;
- wrapper-assisted journals import `cadnx.NXBuilder` and define
  `build(output_path: str = None)`;
- raw NXOpen journals carry `MCP_API_REVIEW` or `STATIC_ONLY_NXOPEN_REVIEW`
  evidence and required diagnostics;
- prohibited local CAD kernels are absent;
- sibling `cadnx/` runtime files compile.

Use `scripts/bundle/bundle-skill.sh nx-cad --check` before handoff when the
repository `models/cadnx/` runtime should be current.

## Brief-Level Validation Plan

Before coding, list what should be checked in NX:

- expected body or assembly-like body count;
- bounding dimensions and main axes;
- critical hole diameters and axes;
- major feature positions;
- whether cosmetic fillets/chamfers are required or optional;
- expected `.prt` and `.step` paths.

## User-Run NX Runtime Validation

After local static checks, hand the journal to the user for manual execution in
Siemens NX. Ask the user to report:

- whether NX executed the journal;
- whether warnings or tracebacks were printed;
- whether `.prt` save was reported;
- whether `.step` export was reported;
- output paths reported by the journal.

Do not run NX through MCP, do not call `dc_run_journal`, and do not try to
start NX. The user runs journals manually in Siemens NX.

If `.step` exists, it can be inspected with the regular CAD skill tools and CAD
Viewer. Until then, report local checks only.

## Raw NXOpen evidence

When a generated or repaired journal uses raw `NXOpen.*`, the final response
must report one of:

- `MCP_API_REVIEW`: list the exact `dc_*` tools used and the API facts checked;
- `STATIC_ONLY_NXOPEN_REVIEW`: state that MCP tools were unavailable and list
  the local static checks that ran.

Do not claim NX runtime success unless the user reports a successful manual NX
run. Report PMI or annotation failures separately from primary solid-generation
failures.

## Post-NX CAD Handoff

After Siemens NX reports that the native part saved and the NX-exported STEP
exists, treat that STEP as the primary review artifact. From the repository
root, run targeted CAD checks against the explicit output path:

```bash
skills/cad/scripts/inspect refs models/<nx_output>.step --facts --planes --positioning
skills/cad/scripts/snapshot models/<nx_output>.step
```

Use the inspection results for scale, body/assembly facts, major planes, and
placement-ready references. Use the snapshot as visual review only, then hand
the same NX-exported STEP to `$cad-viewer` and include the returned link in the
final report. If CAD inspection, snapshot, or `$cad-viewer` is unavailable,
state that explicitly and report the NX runtime evidence that was available.

## Post-NX STEP Review Checklist

After the user confirms NX wrote a `.step`, run CAD checks against the
NX-exported STEP:

```bash
skills/cad/scripts/inspect refs <nx-exported.step> --facts --planes --positioning
skills/cad/scripts/snapshot <nx-exported.step>
```

Then hand the same STEP to `$cad-viewer` when available. Treat snapshot and
viewer review as visual checks; convert visual concerns into measurements,
source changes, or explicit NX rerun requests before claiming them fixed.

## Reporting

Final responses for NX CAD generation should include:

- generated `.py` path;
- synced `cadnx/` path;
- static checks actually run;
- NX runtime result when available;
- exact `dc_*` MCP tools called when MCP API-review mode was used;
- CAD `scripts/inspect refs`, `scripts/snapshot`, and `$cad-viewer` results for
  any NX-exported STEP that exists;
- assumptions and important dimensions;
- exact files to copy to the NX machine;
- a clear statement that NX execution was not verified locally.

Do not claim:

- `.prt` creation;
- STEP export;
- watertight solids;
- visual correctness;
- manufacturability;

unless those facts are supported by NX runtime output or subsequent STEP
inspection.
