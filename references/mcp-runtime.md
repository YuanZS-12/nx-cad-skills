# Designcenter/NXOpen MCP Runtime

Read this when the Designcenter/NXOpen MCP server tools are available in the
agent environment. The MCP server provides NXOpen API discovery, pattern lookup,
snippet execution, and full journal execution through the local NX machine.

## Principle

Use MCP tools to close the NX feedback loop, but keep the `nx-cad` modeling
contract intact:

- Generate portable NXOpen Python journals that use `cadnx.NXBuilder`.
- Prefer wrapper calls over scattered raw NXOpen code.
- Use MCP API tools before adding unsupported raw NXOpen operations.
- Run local static checks before NX runtime execution.
- Report only MCP calls and NX runtime results that actually ran.

If MCP tools are unavailable, fall back to the normal static-only workflow in
`validation.md`.

## Tool Triggers

Use the seven Designcenter/NXOpen MCP tools by need, not mechanically on every
task.

| Tool | Use when |
| --- | --- |
| `dc_lookup_pattern` | You need known Designcenter/NXOpen journal pitfalls, color table facts from `ugcolor.cdf`, color names/indices, or best-practice patterns. |
| `dc_search` | You know likely NXOpen class, method, builder, or enum names and need matching API entries. |
| `dc_semantic_search` | You know the modeling intent but not the API name, such as "create a chamfer", "export STEP", or "set body color". |
| `dc_get_api_info` | You are about to write or revise raw NXOpen code and must confirm constructor shape, method signatures, enum names, properties, return types, or builder lifecycle. |
| `dc_list_namespace` | Search results are too broad or incomplete and browsing an API family such as `NXOpen.Features`, `NXOpen.UF`, or `NXOpen.Assemblies` would narrow the path. |
| `dc_run_snippet` | A small risky fragment should be tested before embedding it in a full journal: builder creation, collector setup, enum assignment, color lookup, save/export behavior, or a new wrapper helper. |
| `dc_run_journal` | The generated journal passed static checks and should be executed in NX to verify native runtime behavior, `.prt` save, and STEP export. |

## API Research Flow

For existing `NXBuilder` calls, do not search raw NXOpen APIs unless an error,
unsupported feature, or compatibility uncertainty requires it.

When extending `cadnx.NXBuilder` or writing unavoidable raw NXOpen code:

1. Start with `dc_semantic_search` for intent-driven discovery or `dc_search`
   when a likely name is known.
2. Use `dc_list_namespace` only when namespace browsing will materially narrow
   the API choice.
3. Use `dc_get_api_info` for every raw class, method, enum, or property whose
   exact shape will be written into code.
4. Use `dc_lookup_pattern` for known journal pitfalls, color-table behavior, or
   Designcenter-specific recommendations.
5. Record any durable compatibility lesson in
   `references/nxopen-common-errors.md` or
   `references/nx-runtime-feedback-ledger.md` when it will help future repairs.

## Runtime Validation Flow

After generating or repairing a journal:

1. Sync the sibling runtime:

   ```bash
   skills/nx-cad/scripts/sync-runtime --models-dir models
   ```

2. Run the local static checker:

   ```bash
   skills/nx-cad/scripts/check-journal models/<journal>.py
   ```

   For industrial/mechanical journals or boolean repairs, also run:

   ```bash
   skills/nx-cad/scripts/check-journal models/<journal>.py --strict-geometry
   ```

3. Use `dc_run_snippet` before the full journal only when a newly researched
   raw API fragment is risky enough to isolate.
4. Use `dc_run_journal` on the complete generated journal.
5. If the journal fails in NX, classify the traceback with `repair-loop.md`,
   patch the smallest responsible section, sync runtime again, rerun local
   checks, and rerun `dc_run_journal`.

## Snippet Policy

Keep snippets short and disposable. They should answer one API question, not
become an alternate source of truth for the model.

Good snippet targets:

- Does this builder/property/enum exist in the local NX version?
- Does a color lookup or color index behave as expected?
- Does a save/export method accept the expected path and options?
- Does a collector/rule pattern initialize without traceback?

Avoid snippets that create large permanent model state unless that is the
specific behavior being tested. Prefer a full generated journal for complete
geometry validation.

## Journal Execution Policy

`dc_run_journal` is the preferred NX runtime proof on machines where the MCP
server is configured. Treat its output as runtime evidence and include it in
the final response.

Report:

- whether NX executed the journal;
- whether `.prt` save was reported;
- whether STEP export was reported;
- any warning or traceback;
- the output paths produced by the journal.

Do not claim visual correctness, manufacturability, or watertight solids from
`dc_run_journal` alone. If a STEP exists, use the CAD inspection and viewer
handoff workflow from `validation.md` when available.

## Fallback Policy

If any MCP tool is unavailable, fails to start, or cannot reach NX:

- Continue with static validation when possible.
- State which MCP tool was unavailable or failed.
- Do not claim NX execution.
- Tell the user which `.py` journal and `cadnx/` folder to copy or rerun on the
  NX machine.

## Final Response Additions

When MCP runtime was used, include:

- API discovery tools used for unsupported raw NXOpen code;
- snippet checks that actually ran;
- `dc_run_journal` result and output paths;
- remaining NX warnings or repair notes;
- whether post-export STEP inspection/viewer review was performed.
