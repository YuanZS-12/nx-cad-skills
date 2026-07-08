# NX Runtime Feedback Ledger

Record user- or automation-reported NX results here. Local static checks are
not enough to mark a helper production-ready.

## Entry Format

```markdown
## YYYY-MM-DD - <journal path> - <NX version if known>

- Result: success|failure
- Operation class: boolean|fillet|chamfer|revolve|sweep|loft|export|save
- Traceback or warning:
- Root cause:
- Patch made:
- Follow-up checks:
```
```

## 2026-07-06 - `models/high_precision_three_axis_drone_gimbal.py` - NX version unknown

- Result: failure, then locally repaired.
- Operation class: boolean.
- Traceback or warning: `rectangular_pocket currently supports Z-axis directions only`.
- Root cause: generated journal used `rectangular_pocket()` for side-facing Y
  pockets even though the wrapper only supported Z-axis pocket directions.
- Patch made: replaced side pockets with explicit `box()` cutters plus
  `boolean_subtract()`.
- Follow-up checks: strict journal check and nx-cad Python tests passed locally;
  user NX rerun still required for runtime success.

## 2026-07-06 - `models/high_precision_hydraulic_manifold_block.py` - NX version unknown

- Result: failure, then locally repaired.
- Operation class: boolean.
- Traceback or warning: NX reported the tool and target did not form a complete
  intersection or had a touch condition that would create zero wall thickness.
- Root cause: valve mounting counterbore was tangent to the pad boundary.
- Patch made: moved `valve_mount_y` inward and added a counterbore edge guard.
- Follow-up checks: strict journal check and nx-cad Python tests passed locally;
  user NX rerun still required for runtime success.

## 2026-07-06 - `/Users/albert/models/S2.py` - NX version unknown

- Result: failure, then locally repaired.
- Operation class: feature budget guard.
- Traceback or warning: `boolean_operations=210 exceeds budget 120`.
- Root cause: intentional impeller stress test exceeded the default budget
  without declaring a higher model-specific max.
- Patch made: added explicit `max_boolean_operations=240` and
  `max_patterned_features=160`.
- Follow-up checks: strict journal check, nx-cad Python tests, and runtime sync
  check passed locally; user NX rerun still required for runtime success.

## 2026-07-06 - `/Users/albert/models/S5.py` - NX version unknown

- Result: failure, then locally repaired.
- Operation class: parameter guard.
- Traceback or warning: `fork pin lug leaves -1.000 mm side wall; minimum is 1.500 mm`.
- Root cause: generated journal compared pin-hole diameter against the
  through-thickness direction instead of checking vertical wall and end edge
  distance.
- Patch made: replaced wrong-direction wall guard with vertical wall and
  end-distance guards.
- Follow-up checks: strict journal check, nx-cad Python tests, and runtime sync
  check passed locally; user NX rerun still required for runtime success.
