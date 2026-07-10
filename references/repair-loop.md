# NX CAD Repair Loop

Read this when a generated NX journal fails locally, in Siemens NX, during
runtime wrapper import, during modeling, or during STEP export.

## Loop

1. Read the exact failing traceback or command output.
2. Classify the failure.
3. Patch the smallest responsible generated journal or `cadnx.NXBuilder`
   wrapper section.
4. Run local static gates:

   ```bash
   skills/nx-cad/scripts/sync-runtime --models-dir models
   skills/nx-cad/scripts/check-journal models/<journal>.py
   ```

5. Use MCP API-review tools, when available, to inspect suspect NXOpen APIs,
   known Designcenter patterns, or wrapper signatures involved in the failure.
6. Do not rerun the same journal in NX through `dc_run_journal`, do not start
   NX, and do not attempt agent-side runtime execution.
7. Ask the user to rerun the same journal manually in Siemens NX after the
   repair and send back the full traceback or success output.
8. Record new NX compatibility failures in `references/nxopen-common-errors.md`.

## Failure Classes

### Import Or Path Failure

Likely causes:

- copied the `.py` journal without the sibling `cadnx/` directory;
- ran the journal with normal Python instead of NX;
- stale `cadnx/builder.py` on the Windows machine;
- hardcoded workspace path in the generated journal.

Fix:

- rerun `sync-runtime`;
- copy the journal and `cadnx/` together;
- keep generated paths relative to `__file__`;
- do not claim local execution as NX execution.

### NXOpen API Compatibility Failure

Likely causes:

- NX version exposes a different builder method name;
- builder collector property is absent or `None`;
- enum name differs from examples;
- recorded journal API is not portable across NX versions.

Fix:

- patch `cadnx/builder.py`, not every generated journal;
- use `getattr` and fallback variants;
- prefer selection collectors and rules over topology index assumptions;
- keep the old path when it worked and add a fallback for the reported version.

### Raw NXOpen Journal Failure

Classify raw journal failures more specifically before patching:

- raw NXOpen import/submodule failure;
- raw builder property or enum mismatch;
- section/curve construction failure;
- primary feature commit failure;
- optional PMI/annotation failure after primary solid creation;
- save/export failure.

Fix:

- add explicit submodule imports such as `NXOpen.Features` or
  `NXOpen.Annotations`;
- use MCP API-review tools to confirm the exact builder, enum, property, and
  method shape;
- keep primary solid generation separate from optional PMI so annotation
  failures do not erase successful geometry;
- add runtime diagnostics for work part, body/feature count, and output path.

### Invalid Or Missing Geometry

Likely causes:

- zero or negative dimension;
- subtractive tool misses the target;
- boolean did not affect the intended body;
- feature order made topology fragile.

Fix:

- simplify the failing feature;
- overcut through tools by 1-3 mm;
- build primary solids before details;
- apply cosmetic fillet/chamfer operations last.

### Fillet Or Chamfer Failure

Likely causes:

- radius or offset exceeds local geometry;
- selected edge group includes tiny or unintended edges;
- boolean topology is too complex for one continuous operation.

Fix:

- reduce the radius/offset in the generated journal when it is functional;
- use narrower edge selectors;
- let wrapper safe operations skip cosmetic failures;
- avoid `get_all_edges()` on complex multi-boolean bodies unless cosmetic.

### Save Or STEP Export Failure

Likely causes:

- NX `SaveAs` check failed;
- output directory is not writable;
- STEP exporter properties differ by NX version;
- display part exists but native `.prt` save failed.

Fix:

- let `NXBuilder.export_step()` try normal save, `_cadnx_work/`, then display
  part export fallback;
- ensure output paths are near the journal and writable;
- if STEP still fails, simplify fragile detail features and rerun NX part
  checks.

## Patch Decision

| Failure | Patch first |
| --- | --- |
| NXOpen import, work part, save, or export compatibility | `cadnx/builder.py` |
| Boolean tangent/no complete intersection | generated journal geometry plan |
| Wrong wall guard or impossible parameter | generated journal parameters/checker guidance |
| Primitive-only high-precision part class | skill references and regenerated journal |
| STEP exists but looks wrong | source journal after CAD inspect/snapshot evidence |

Patch `cadnx/builder.py` when:

- two or more models can hit the same NXOpen compatibility issue;
- the failing API is inside a wrapper operation;
- the generated source used documented wrapper calls correctly.

Patch the generated journal when:

- the feature plan is too fragile or too ambitious;
- the selected edge set is wrong;
- a dimension or axis in the model brief was interpreted incorrectly.

## Report After Repair

Tell the user:

- which file changed;
- which runtime folder was synced;
- which local checks ran;
- which `dc_*` tools were used for API review, if MCP API-review mode was
  available;
- which file to copy to the NX machine;
- what the next NX rerun should prove.
