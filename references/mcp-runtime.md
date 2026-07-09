# Designcenter/NXOpen MCP Runtime

Read this when the Designcenter/NXOpen MCP server may be available. The MCP
server is the preferred path for NXOpen API discovery, snippet trials, and
executing finished journals on a local NX machine.

## MCP Discovery Gate

At the start of an NX-machine task, determine whether these tools are exposed
in the current agent session:

- `dc_lookup_pattern`
- `dc_search`
- `dc_semantic_search`
- `dc_get_api_info`
- `dc_list_namespace`
- `dc_run_snippet`
- `dc_run_journal`

If any `dc_*` tools are available, enter MCP runtime mode. If none are
available, say this before generating code:

```text
Designcenter/NXOpen MCP tools are not available in this agent session; using
static-only nx-cad workflow.
```

Do not wait until the final response to reveal missing MCP tools.

## Runtime Modes

**MCP runtime mode** is active when one or more `dc_*` tools are exposed. In
this mode, MCP is runtime evidence and `dc_run_journal` is the default
post-static-check execution path.

**Static-only mode** is active when no `dc_*` tools are exposed. In this mode,
generate portable NX journals, run local static checks, and tell the user which
`.py` journal and sibling `cadnx/` folder to run on the NX machine. Do not claim
NX execution.

## Minimum MCP Use

In MCP runtime mode:

1. Use at least one discovery tool before generating or modifying NXOpen code:
   `dc_search`, `dc_semantic_search`, or `dc_lookup_pattern`.
2. Use `dc_get_api_info` before writing raw `NXOpen.*` calls or extending
   `cadnx.NXBuilder`.
3. Use `dc_run_snippet` before embedding risky newly researched API fragments,
   such as builder setup, collectors, enum values, color lookup, save, or STEP
   export behavior.
4. Run `dc_run_journal` after `sync-runtime` and `check-journal` pass unless
   the user explicitly asks not to execute NX.

Do not call all seven tools mechanically. Use the minimum set that proves API
shape and runtime behavior.

## Tool Triggers

| Tool | Use when |
| --- | --- |
| `dc_lookup_pattern` | You need known Designcenter/NXOpen journal pitfalls, color facts from `ugcolor.cdf`, color names/indices, or best-practice patterns. |
| `dc_search` | You know likely NXOpen class, method, builder, or enum names and need matching API entries. |
| `dc_semantic_search` | You know the modeling intent but not the API name, such as "create a chamfer", "export STEP", or "set body color". |
| `dc_get_api_info` | You are about to write or revise raw NXOpen code and must confirm constructors, methods, enum names, properties, return types, or builder lifecycle. |
| `dc_list_namespace` | Search results are broad or incomplete and browsing an API family such as `NXOpen.Features`, `NXOpen.UF`, or `NXOpen.Assemblies` would narrow the path. |
| `dc_run_snippet` | A risky fragment should be tested before embedding it in a full journal. |
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
5. Record durable compatibility lessons in `references/nxopen-common-errors.md`
   or `references/nx-runtime-feedback-ledger.md` when they help future repairs.

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

3. Use `dc_run_snippet` only when a newly researched raw API fragment is risky
   enough to isolate.
4. Use `dc_run_journal` on the complete generated journal.
5. If NX fails, classify the traceback with `repair-loop.md`, patch the
   smallest responsible section, sync runtime again, rerun local checks, and
   rerun `dc_run_journal`.

## MCP Evidence Report

When MCP runtime mode was used, the final response must list:

- exact `dc_*` tools actually called;
- API or pattern facts learned from MCP;
- snippet results, if any;
- `dc_run_journal` result;
- NX traceback or success output;
- generated `.prt` and `.step` paths when reported.

When MCP tools were expected but unavailable, the final response must state
that static-only validation was performed and must not claim NX execution.

## Integration Test Prompt

Use a small model to test whether Copilot is actually calling MCP:

```text
/nx-cad Create a 60 x 40 x 8 mm NXOpen mounting plate with four M5 clearance
holes and export STEP.

Before writing code, check whether Designcenter/NXOpen MCP tools are available.
If available, call one API discovery tool and run dc_run_journal after static
checks. If unavailable, stop and report that before generating code.
```
