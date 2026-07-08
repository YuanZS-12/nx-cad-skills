# NX CAD Validation

Read this before reporting success for generated NXOpen journals.

## Principle

Local validation can prove source structure and runtime freshness. It cannot
prove Siemens NX execution, native `.prt` save, or STEP export unless the
journal was actually run inside NX and the user or automation reports the
result.

## Local Static Validation

Always run after generating or modifying a journal:

```bash
skills/nx-cad/scripts/sync-runtime --models-dir models
skills/nx-cad/scripts/check-journal models/<journal>.py
```

These checks prove:

- the journal is valid Python syntax;
- the journal imports `cadnx.NXBuilder`;
- the journal defines `build(output_path: str = None)`;
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

## NX Runtime Validation

When the user runs the journal in Siemens NX, ask for:

- whether geometry appears;
- whether warnings were printed;
- whether `.prt` saved;
- whether `.step` exported;
- the full traceback for failures.

If `.step` exists, it can be inspected with the regular CAD skill tools and CAD
Viewer. Until then, report local checks only.

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
