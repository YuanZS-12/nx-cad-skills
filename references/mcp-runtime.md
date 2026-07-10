# Designcenter/NXOpen MCP Runtime

Read this when the Designcenter/NXOpen MCP server may be available. The MCP
server is the preferred path for NXOpen API discovery, signature lookup,
Designcenter pattern lookup, and code review. It must not run full NX journals,
start NX, or otherwise perform NX runtime execution. The user runs journals
manually in Siemens NX and reports tracebacks or success output back to the
agent.

## MCP Discovery Gate

At the start of an NX-machine task, determine whether these tools are exposed
in the current agent session:

- `dc_lookup_pattern`
- `dc_search`
- `dc_semantic_search`
- `dc_get_api_info`
- `dc_list_namespace`
- `dc_run_snippet`
- `dc_run_journal` (do not call; runtime execution is user-run only)

If any `dc_*` API lookup or review tools are available, enter MCP API-review
mode. If none are
available, say this before generating code:

```text
Designcenter/NXOpen MCP tools are not available in this agent session; using
static-only nx-cad workflow.
```

Do not wait until the final response to reveal missing MCP tools.

## Review Modes

**MCP API-review mode** is active when one or more `dc_*` API lookup or review
tools are exposed. In this mode, use MCP to read NXOpen API information, verify
signatures, inspect known Designcenter patterns, and review generated journal
code. Do not launch NX or execute the full journal.

**Static-only mode** is active when no `dc_*` tools are exposed. In this mode,
generate portable NX journals, run local static checks, and tell the user which
`.py` journal and sibling `cadnx/` folder to run on the NX machine. Do not claim
NX execution.

## Raw NXOpen Review Path

Raw NXOpen review path is the normal MCP-backed review flow for direct NXOpen
journal code.

This path is not limited to unsupported wrapper operations.

Raw NXOpen review is a normal path for NX journal generation, not limited to
unsupported wrapper operations. Use it whenever direct NXOpen builders,
collections, sections, curves, PMI objects, expressions, assemblies, or export
APIs are the clearest way to model the requested part.

The MCP server is the evidence layer. It helps the agent avoid hallucinated
classes, wrong enum names, missing submodule imports, invalid builder
lifecycle, and version-specific API assumptions. It is not a requirement to
wrap every confirmed API in `NXBuilder`.

Do not call `dc_run_journal`, start NX, or run full journals.
Use `dc_run_snippet` only for short probes when lookup is insufficient.

## Minimum MCP Use

In MCP API-review mode:

1. Use at least one discovery tool before generating or modifying NXOpen code:
   `dc_search`, `dc_semantic_search`, or `dc_lookup_pattern`.
2. Use `dc_get_api_info` before writing raw `NXOpen.*` calls or extending
   `cadnx.NXBuilder`.
3. Use `dc_run_snippet` only for short API probes or review snippets when a
   signature or behavior cannot be confirmed by lookup alone. Keep snippets
   small and avoid full model generation.
4. Do not call `dc_run_journal`. The user runs journals manually in Siemens NX.

Do not call all seven tools mechanically. Use the minimum set that proves API
shape and code correctness.

## Tool Triggers

| Tool | Use when |
| --- | --- |
| `dc_lookup_pattern` | You need known Designcenter/NXOpen journal pitfalls, color facts from `ugcolor.cdf`, color names/indices, or best-practice patterns. |
| `dc_search` | You know likely NXOpen class, method, builder, or enum names and need matching API entries. |
| `dc_semantic_search` | You know the modeling intent but not the API name, such as "create a chamfer", "export STEP", or "set body color". |
| `dc_get_api_info` | You are about to write or revise raw NXOpen code and must confirm constructors, methods, enum names, properties, return types, or builder lifecycle. |
| `dc_list_namespace` | Search results are broad or incomplete and browsing an API family such as `NXOpen.Features`, `NXOpen.UF`, or `NXOpen.Assemblies` would narrow the path. |
| `dc_run_snippet` | A short API probe or code-review snippet is needed to confirm behavior not clear from lookup. |
| `dc_run_journal` | Do not use. The user runs finished journals manually in Siemens NX. |

## API Research Flow

For existing `NXBuilder` calls, do not search raw NXOpen APIs unless an error,
unsupported feature, or compatibility uncertainty requires it.

When writing raw NXOpen code, reviewing generated raw journal code, or extending
`cadnx.NXBuilder`:

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

## API Review Flow

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

3. Use MCP lookup tools to review any raw NXOpen use, wrapper extension, export
   assumptions, color usage, or known fragile journal pattern.
4. Use `dc_run_snippet` only for short API probes when lookup is insufficient.
5. Do not run `dc_run_journal`. Ask the user to run the repaired journal in
   Siemens NX and send the full traceback or success output back for the next
   repair pass.

## MCP Evidence Report

When MCP API-review mode was used, the final response must list:

- exact `dc_*` tools actually called;
- API or pattern facts learned from MCP;
- snippet results, if any.

Only include NX traceback, success output, generated `.prt`, or generated
`.step` paths when the user reported them from a manual Siemens NX run.

When MCP tools were expected but unavailable, the final response must state
that static-only validation was performed and must not claim NX execution.

## Integration Test Prompt

Use a small model to test whether Copilot is actually calling MCP:

```text
/nx-cad Create a 60 x 40 x 8 mm NXOpen mounting plate with four M5 clearance
holes and export STEP.

Before writing code, check whether Designcenter/NXOpen MCP tools are available.
If available, call one API discovery tool before writing code and do not run
dc_run_journal. If unavailable, stop and report that before generating code.
```
