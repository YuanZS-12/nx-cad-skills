# NX CAD Validation

Read this before reporting success for generated NXOpen journals.

## Principle

Validation is layered. Local checks can prove source structure, route
consistency, and wrapper runtime freshness. They cannot prove Siemens NX
execution, native `.prt` save, STEP export, solid validity, or visual
correctness. NX runtime validation is user-run only: agents do not start NX and
do not call `dc_run_journal`.

## Validation Hierarchy

### Wrapper Mode

1. Syntax/static check:

   ```bash
   skills/nx-cad/scripts/check-journal models/<journal>.py
   ```

2. Runtime wrapper sync when the journal imports `cadnx`:

   ```bash
   skills/nx-cad/scripts/sync-runtime --models-dir models
   skills/nx-cad/scripts/check-journal models/<journal>.py
   ```

3. Strict geometry static check when the journal has industrial/mechanical
   booleans, holes, ribs, bosses, slots, or patterned features:

   ```bash
   skills/nx-cad/scripts/check-journal models/<journal>.py --strict-geometry
   ```

4. User-run Siemens NX journal execution.
5. User-reported NX stdout, warnings, traceback, `.prt` save, and `.step`
   export paths.
6. Post-NX STEP inspection, snapshot, and CAD Viewer handoff of the actual
   NX-exported STEP.

### Raw NXOpen High-Fidelity Mode

1. Syntax/static check:

   ```bash
   skills/nx-cad/scripts/check-journal models/<journal>.py
   ```

2. MCP API-review evidence, or explicit `STATIC_ONLY_NXOPEN_REVIEW` if MCP
   tools were unavailable.
3. Check that `cadnx/` is not required by the journal.
4. User-run Siemens NX journal execution.
5. User-reported NX stdout, warnings, traceback, body/feature diagnostics,
   `.prt` save, and `.step` export paths.
6. Post-NX STEP inspection, snapshot, and CAD Viewer handoff of the actual
   NX-exported STEP.

## What Local Checks Prove

`skills/nx-cad/scripts/check-journal` proves only the checks it implements:

- Python syntax is valid.
- Forbidden local CAD kernels are absent.
- Wrapper journals use `NXBuilder` entry points and STEP export calls.
- Raw NXOpen journals include `MCP_API_REVIEW` or
  `STATIC_ONLY_NXOPEN_REVIEW`.
- Raw high-fidelity journals do not use `cadnx.NXBuilder`.
- Raw journals include basic diagnostics and builder cleanup guardrails.
- Referenced `cadnx/` runtime files compile when the journal needs them.

It does not prove NX can commit the features or export a STEP.

For raw NXOpen evidence preflight during development, use:

```bash
skills/nx-cad/scripts/check-nxopen-api models/<journal>.py
```

This helper checks only raw NXOpen imports, evidence markers, and obvious raw
high-fidelity route violations. It does not replace `check-journal`.

## Brief-Level Validation Plan

Before coding, record in the design ledger:

- expected body or assembly-like body count;
- bounding dimensions and main axes;
- critical hole diameters, axes, and locations;
- major feature positions;
- whether cosmetic fillets, chamfers, colors, PMI, or annotations are required
  or optional;
- expected `.prt` and `.step` path behavior;
- what user-run NX output should prove.

## Raw NXOpen Evidence

Raw NXOpen evidence is required for generated or repaired raw `NXOpen.*`
journals.

When a generated or repaired journal uses raw `NXOpen.*`, the final response
must report one of:

- `MCP_API_REVIEW`: exact `dc_*` tools used and the API facts checked;
- `STATIC_ONLY_NXOPEN_REVIEW`: MCP tools were unavailable or not exposed, plus
  the local static checks that ran.

Do not imply MCP was used unless the final answer lists the actual `dc_*` calls.
Do not claim NX runtime success unless the user reports a successful manual NX
run. Report PMI, annotation, color, or cosmetic failures separately from
primary solid-generation failures.

## User-Run NX Runtime Validation

After local checks, hand the journal to the user for manual execution in
Siemens NX. Ask the user to report:

- whether NX executed the journal;
- warnings, tracebacks, or diagnostic prints;
- committed feature/body counts when printed;
- whether native `.prt` save was reported;
- whether `.step` export was reported;
- output paths reported by the journal.

For wrapper mode, tell the user to copy the journal and sibling `cadnx/`.
For raw NXOpen high-fidelity mode, tell the user to copy only the journal unless
the generated source explicitly imports additional local files.

## Post-NX STEP Review

After Siemens NX reports that the native part saved and the NX-exported STEP
exists, treat that STEP as the primary review artifact. From the repository
root, run targeted CAD checks against the explicit output path:

```bash
skills/cad/scripts/inspect refs <nx-exported.step> --facts --planes --positioning
skills/cad/scripts/snapshot <nx-exported.step>
```

Then hand the same STEP to `$cad-viewer` when available. Snapshot and viewer
review are visual checks; convert visual concerns into measurements, source
changes, or explicit NX rerun requests before claiming them fixed.

## Reporting Shapes

### Wrapper Mode

```text
Mode: wrapper
Generated: models/<journal>.py
Runtime: models/cadnx/ synced or not required
Checks run: <commands and results>
Assumptions: <brief summary>
Copy to NX machine: models/<journal>.py and models/cadnx/
NX runtime: not verified locally; user must run manually in Siemens NX
```

### Raw NXOpen High-Fidelity Mode

```text
Mode: raw NXOpen high-fidelity
Generated: models/<journal>.py
Runtime: cadnx/ not required
Review evidence: MCP_API_REVIEW with <dc_* tools> or STATIC_ONLY_NXOPEN_REVIEW
Checks run: <commands and results>
Assumptions: <brief summary>
Copy to NX machine: models/<journal>.py
NX runtime: not verified locally; user must run manually in Siemens NX
```

### Post-NX STEP Review

```text
NX runtime evidence: <user-reported stdout/paths>
STEP: <nx-exported.step>
Inspection: <facts/planes/measurements>
Snapshot: <path or skipped reason>
CAD Viewer: <link or unavailable reason>
Remaining risks: <unchecked claims>
```

## Claims Not Allowed Without Evidence

Do not claim:

- NX execution;
- `.prt` creation;
- STEP export;
- watertight solids;
- visual correctness;
- manufacturability;
- tolerance compliance;
- structural safety;

unless those facts are supported by user-reported NX runtime output or
subsequent STEP inspection.
