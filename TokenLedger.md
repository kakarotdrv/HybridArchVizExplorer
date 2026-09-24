# Token Ledger: ArchVizExplorer

These are estimates. Unless a row says "measured", each figure is roughly how much a step added to the conversation. It is not the billed amount. Every API call re-sends the whole conversation (mostly from cache), so billed input tokens are much higher than these numbers. Run `/cost` (or `/usage` on a subscription) for real session totals.

---

# Session 1: Lift outer-door task

## Turn 1: "can you access Bp_Lift" (Sonnet 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 0 | Baseline: system prompt, tool definitions, first message | n/a | ~41,500 |
| 1 | ToolSearch: load Unreal MCP tools | ok | (in baseline) |
| 2 | list_toolsets | ok | ~670 |
| 3 | describe_toolset BlueprintTools (72K chars, saved to file) | ok | ~1,240 |
| 4 | Grep tool names from saved file | ok | ~870 |
| 5 | list_graphs BP_Lift (wrong param name) | error | ~2,400 |
| 6 | list_graphs BP_Lift | ok | ~500 |
| | **Turn total** | | **~47,200** |

## Turn 2: "now access BP_Lift_Door" (Sonnet 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | list_graphs BP_Lift_Door | ok | ~300 |
| | **Turn total** | | **~300** |

## Turn 3: "in" (incomplete message) (Sonnet 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Asked for clarification | n/a | <100 |

## Turn 4: Implementation request / investigation (Sonnet 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | list_variables, list_events, list_functions, list_event_dispatchers (BP_Lift) | ok | ~1,300 |
| 2 | read_graph_dsl EventGraph + DoorOpen | EventGraph failed, DoorOpen empty | ~2,900 |
| 3 | get_graph (wrong params) | error | ~470 |
| 4 | get_graph | ok | ~520 |
| 5 | read_graph_dsl (wrong params) | error | ~360 |
| 6 | find_nodes (missing title) | error | ~650 |
| 7 | read_graph_dsl EventGraph retry | error | ~820 |
| 8 | find_nodes EventGraph, all ~190 nodes | ok | ~320 |
| 9 | ls memory dir (bash quoting error) | error | ~12,500 (mostly the step 8 node list) |
| 10 | Glob memory dir | ok, empty | ~200 |
| 11 | get_node_infos: 5 custom events | ok | ~2,240 |
| 12 | Launch background agent (read-only trace) | running | ~8,150 |
| | **Turn total** | | **~30,400** |

## Background agent: BP_Lift door-logic trace

| Step | Status | Est. tokens |
|------|--------|------------:|
| Read-only trace of Lift_Call_A/B, door maps, BP_Lift_Door events | completed (48 tool calls, ~6 min 40 s) | **144,998** (measured by harness) |

The agent started with the full conversation above (~78K tokens, mostly cached).

**Finding:** the feature is already built. After the lift stops, both lifts look up the called floor in LiftADoorsByFloor / LiftBDoorsByFloor and call OuterDoorOpen on that door.

## Turn 5: Ledger request (Opus 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Write this ledger | ok | ~2,000 |

## Turn 6: Verify agent findings (Opus 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | get_node_infos: CallFunction_33/35/36/37 (Map Find + OuterDoorOpen) | confirmed | ~3,200 |
| 2 | Update this ledger | ok | ~1,000 |

## Turn 7: Implement the two fixes (Opus 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Back up both .uasset files to scratchpad | ok | ~600 |
| 2 | Verify Map Add / map variables / debug prints (10 nodes) | confirmed | ~6,900 |
| 3 | List BP_Lift_Door nodes + variables | ok | ~2,400 |
| 4 | Inspect OuterDoorOpen, Interact, Branch, Timeline (6 nodes) | toggle confirmed | ~7,600 |
| 5 | Extract edit-tool schemas (1 failed attempt, then JSON parse) | ok | ~4,700 |
| 6 | create_node x4 (1 failed: variable getter not creatable) | 3 created | ~3,200 |
| 7 | find_node_types lookup for variable getter | empty | ~1,800 |
| 8 | break_pins x3, connect_pins x9 | ok | ~6,200 |
| 9 | compile_blueprint x2 | no errors | ~900 |
| 10 | Verify the 3 new branches | confirmed | ~1,500 |
| | **Turn total** | | **~48,000** |

## Session 1 total

| Item | Est. tokens |
|------|------------:|
| Turns 1-7 (main conversation) | ~132,000 |
| Background agent | 144,998 |
| **Session total** | **~277,000** |
| Wasted on errors (steps 1.5, 4.3, 4.5-4.7, 4.9 tool overhead) | ~5,000 |

### Session 1 notes
- **Model:** Turns 1-4 ran on Sonnet 5 and turns 5-7 on Opus 5. Per-token prices differ.
- **Biggest single cost:** the full EventGraph node list (~12K tokens). Later steps should use targeted `find_nodes` title filters.
- **Error waste:** most of it came from guessing MCP parameter names.

---

# Session 2: Lift door sync, floor doors, side windows, kitchen and living room swap sets (Opus 5, Sep 16-19)

**Measured** (desktop app usage readout): after turn 15 the context held **544,235 tokens**; after turn 17 it held **772,533 tokens** (77% of a 1M window: 734,014 messages, 26,375 system tools, 4,349 skills, 4,179 MCP tools, 2,892 system prompt, 780 memory files). Plan: Pro. At the end, the 5-hour limit was 64% used and the weekly limit 49%.

The per-turn figures below are rough splits of that measured total. No background agents were used.

| Turn | Request | Outcome | Est. tokens |
|---|---|---|---:|
| 0 | Baseline: system prompt, tools, skills | n/a | ~37,500 (measured) |
| 1 | "Do you have memory of what you were implementing?" | Memory dir empty; history recovered from this ledger | ~3,000 |
| 2 | "How could your memory be empty?" | Explained: memory holds preferences, not work state | ~2,000 |
| 3 | Explain how the door map finds the right door | Found BeginPlay bug: every door registered with both lifts | ~49,000 |
| 4 | Fix it, reusing existing variables and logic | Fixed with 1 new getter + 4 rewires; 12/12 doors verified in Simulate | ~39,000 |
| 5 | "What changes did you do?" | Listed the edits | ~2,000 |
| 6 | Fix call-button and floor-button glitches, keep doors in sync | Lift now owns both doors (see below) | ~196,000 |
| 7 | Remove dead Branch nodes | 3 removed, compiled, saved | ~9,000 |
| 8 | Read Outliner selection | 8 BP_Gate doors | ~13,000 |
| 9 | Rename them "6th Floor Doors" | 6th Floor Doors 1-8 | ~4,000 |
| 10 | Selected door is also 6th floor | 6th Floor Doors 9 | ~2,000 |
| 11 | Put floors 1-5 doors in the same closed positions | 45 doors moved; restore file written | ~51,000 |
| 12 | Rename by floor, fix the growing gap under doors | Floors measured 339.6 apart; 45 doors lowered and renamed | ~32,000 |
| 13 | Continue from phone | Remote Control on, keep-awake requested | ~4,000 |
| 14 | Copy 6th-floor side windows to floors 1-5 | 60 actors created and verified | ~83,000 |
| 15 | Update this ledger and memory | This section | ~17,000 |
| | **Subtotal after turn 15** | | **~544,000 (measured)** |
| 16 | Access and open the selected blueprint | BP_DinningTable opened and summarised | ~8,000 |
| 17 | Build kitchen and living room swap blueprints like BP_DinningTable | BP_KitchenCabinate, BP_LivingRoomFurniture, WBP_LivingRoom built; WBP_KitchenCabinate rewired | ~220,000 |
| | **Subtotal after turn 17** | | **~772,500 (measured)** |
| 18 | Shrink and spin each mesh about its own centre instead of the set root | 54 pivots added, SwapTick loops over them; shelf/sink flip fixed; level actors replaced | ~110,000 (estimated, after a context compaction) |
| 19 | Alt tall unit hides sink, microwave/oven and fridge: rebuild it in Blender and replace it | New open-frame unit modelled, baked, exported and swapped in; Alt sink lowered 15 cm | ~130,000 (estimated) |
| 20 | Check the Unreal connection, update this ledger, then wire the home button back to the explorer pawn | Editor had been restarted; MCP port 8000 open but the session's connector had failed at startup and could not be re-dialled from the session; ledger updated | ~6,000 (estimated) |
| 21 | "try now": connect and build the home button | Connected by calling the MCP endpoint directly over HTTP; traced the pawn-switch flow; home button wired to `Switch_Pawn(Main Explorer Pawn)`; removed 2 stale allow rules that caused startup warnings | ~45,000 (estimated) |
| 22 | Fix the taskbar highlight after returning from first person | Home click now also calls the master menu's `Update_TaskbarButton_Style(Button_Home)` | ~6,000 (estimated) |
| | **Session total** | | **~1,069,500** |

## Turn 3: Explain the door map

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Load Unreal MCP tools, describe BlueprintTools (72K chars to file), parse schemas | ok | ~7,000 |
| 2 | Variables, events and graphs of BP_Lift / BP_Lift_Door | ok | ~6,000 |
| 3 | read_graph_dsl on door EventGraph | error | ~500 |
| 4 | RegisterOuterDoorA and door BeginPlay subgraphs | ok | ~13,000 |
| 5 | Lift Map Find / OuterDoorOpen nodes, toolset list, SceneTools schema | ok | ~13,000 |
| 6 | Level scan: WhichLift / Floor of 12 doors, BP_LiftMenu wiring | bug found | ~9,500 |

## Turn 4: Fix door registration

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Check reusable getters and how Interact uses BP_Lift_A/B | safe to rewire | ~13,000 |
| 2 | Back up .uassets, add door WhichLift getter, rewire both branches | ok | ~4,000 |
| 3 | Compile and read back | ok | ~4,000 |
| 4 | Simulate, read both lifts' maps, check log | every floor maps to the right door | ~18,000 |

## Turn 6: Lift and door timing rework

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Graph-dump script via ProgrammaticToolset; bisect nodes that get_node_infos can't read | ok after ~10 runs | ~36,000 |
| 2 | Full flow dump of both blueprints | 5 root causes found | ~17,000 |
| 3 | Delay durations, timeline lengths, node-type lookups | ok | ~4,000 |
| 4 | Ask 3 design questions | all recommended options chosen | ~4,000 |
| 5 | Pin and position survey, schema lookups, second backup | ok | ~15,000 |
| 6 | Door: new SetOuterDoorAlpha function; removed door-side toggles and delays | ok | ~9,000 |
| 7 | Lift: bCallPending, reuse CallFromWhichFloor, ~40 new nodes (5 build attempts) | built | ~65,000 |
| 8 | Fix Select node pin typing, compile with warnings as errors | clean | ~9,000 |
| 9 | Read back the new flow | matches plan | ~13,000 |
| 10 | Simulate check: lift actors stuck on a REINST class | problem | ~9,000 |
| 11 | Save both blueprints, reload Main (had no unsaved changes), Simulate again | fixed | ~15,000 |

## Turn 11: Align doors on floors 1-5

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Scan 57 BP_Gate doors and lift door heights | ok | ~7,000 |
| 2 | Match each door to its 6th-floor doorway, move 45 (large output) | ok | ~32,000 |
| 3 | Write restore file of original transforms | ok | ~12,000 |

## Turn 12: Fix the gap under doors

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Floor traces beside 54 doors | picked the wrong surface layer | ~9,000 |
| 2 | Multi-hit probes, lift stop heights | inconclusive | ~4,000 |
| 3 | Viewport capture (2.1M chars saved to file, decoded to PNG) | red-tinted, not usable | ~5,000 |
| 4 | Full-height probe | floors every 339.6 | ~3,000 |
| 5 | Lower 45 doors by 14.4-73.7, rename all 45 | ok | ~11,000 |

## Turn 14: Copy side windows

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Read the 12 selected actors | 6 BP_SideDoor + 6 static meshes | ~4,000 |
| 2 | Bounds scan of lower floors | interrupted (floors confirmed empty by user) | ~3,000 |
| 3 | BP_SideDoor components, slide variables, mesh overrides | ok | ~7,000 |
| 4 | Build attempts 1-2 | failed checks | ~17,000 |
| 5 | set_properties format tests (6 runs) | workarounds found | ~20,000 |
| 6 | Build attempts 3-4 | 60 created and verified | ~20,000 |
| 7 | Remove 16 leftovers from failed attempts, final count | 36 BP_SideDoor, 30 window meshes | ~12,000 |

## Turn 17: Kitchen and living room swap blueprints

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Trace BP_DinningTable and BP_BaseEditModeInteractions | DS1/DS2 = Timeline spin + shrink per set | ~8,000 |
| 2 | Widget, blueprint and imported-asset inventory | 9 kitchen alts; living set imported as separate _LOD0.._LOD4 meshes | ~14,000 |
| 3 | Widget graphs (WBP_KitchenSlab, WBP_KitchenCabinate) | swap pattern found; KitchenCabinate Construct unconnected | ~6,000 |
| 4 | Find originals in level (152 static mesh actors, 71 Datasmith actors) | furniture is Datasmith HISM, one instance per floor | ~12,000 |
| 5 | Read 6th-floor instance transforms | exact placements and scales | ~8,000 |
| 6 | Two question rounds; read Blender ROOM_SCENE layout | choices made; 11-piece layout read | ~25,000 |
| 7 | Explain manual HISM editing | n/a | ~3,000 |
| 8 | Back up Main.umap; remove 18 floor-6 instances; write restore file | done | ~14,000 |
| 9 | First blueprint build timed out at 300 s; the abort also undid the instance removal | redone and saved to disk | ~20,000 |
| 10 | BP_KitchenCabinate: 28 mesh components, variables, SwapTick timer, KS1/KS2 | compiled, saved | ~35,000 |
| 11 | BP_LivingRoomFurniture: 16 mesh components, same logic, LS1/LS2 | compiled, saved | ~25,000 |
| 12 | WBP_LivingRoom (duplicate of WBP_KitchenSlab) and WBP_KitchenCabinate rewiring | compiled, saved | ~20,000 |
| 13 | Place both actors, set widgets, verify placement (0.0 error), Simulate, save | ok | ~12,000 |
| 14 | Update this ledger | ok | ~10,000 |

## Turn 18: Per-mesh pivots for the swap animation

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Re-describe BlueprintTools, ActorTools, StaticMeshTools, ObjectTools, ProgrammaticToolset | ok | ~25,000 |
| 2 | Read SwapTick DSL, all component transforms and mesh bounds | shelf/sink found upside-down (mirrored originals) | ~15,000 |
| 3 | Back up both .uassets to `backup3_pivots/`; one-mesh pivot test | ok (1.3 s) | ~3,000 |
| 4 | Add pivots at bounds centres: Kitchen Set 1 (35 s), Set 2 (47 s), Living Set 1 (7 s), Set 2 (43 s); save after each | 54 pivots; shelf/sink fixed | ~15,000 |
| 5 | Rewire SwapTick in both blueprints (GetChildrenComponents + ForEachLoop), compile, save | clean | ~12,000 |
| 6 | Verify placed actors | stale per-instance values; reset_properties had no effect | ~10,000 |
| 7 | Replace both placed actors, re-apply Widget class and height, verify 18/18 vs Datasmith, save Main | exact | ~12,000 |
| 8 | Simulate with Target1/Target2 swapped on the CDO, sample pivots, revert, recompile, save | Set 1 shrank and spun in place, Set 2 grew; no runtime errors | ~10,000 |
| 9 | Update this ledger | ok | ~5,000 |

## Turn 19: Rebuild the Alt tall kitchen unit

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Read the Alt handoff; list Blender shelf objects | original and Alt both in `KitchenAlt_DarkLuxe.blend` | ~8,000 |
| 2 | Ray-cast depth maps and vertex coordinates of the original unit | open frame: fridge alcove, appliance column with 2 cavities, sink niche; front is Blender -Y | ~12,000 |
| 3 | Convert appliance transforms from BP_KitchenCabinate into unit space | cavities match the microwave/oven; found the Alt sink counter 15 cm too high | ~10,000 |
| 4 | Read pipeline code (`kalt`, `kbake`, `kprev`), old material wiring, UE asset setup | ok | ~15,000 |
| 5 | Model the new unit (2 builds), clearance test, 3 preview renders | 0 overlaps with microwave, oven, fridge, taps, sink basin | ~20,000 |
| 6 | Back up exports, textures, .blend and UE assets; UVs, UCX, untextured FBX/GLB | ok | ~8,000 |
| 7 | 4K bake (OPTIX, one call), ORM pack, baked material, textured FBX/GLB, save .blend | ok | ~8,000 |
| 8 | UE: `import_file` refused to overwrite; renamed old assets to `_OLD`, imported new textures + mesh, repointed the material | ok | ~12,000 |
| 9 | BP: `S2_Shelf` mesh, `S2_Sink_Pivot` lowered; placed actor stale again, replaced | ok | ~8,000 |
| 10 | Simulate with Set 2 forced on, 4 viewport captures decoded from base64, revert, save | layout correct in engine; no log errors | ~25,000 |
| 11 | Update this ledger | ok | ~4,000 |

## What Session 2 changed in the project

- **BP_Lift_Door**
  - BeginPlay: each door registers only with the lift that matches its own WhichLift.
  - Interact: calls the lift, or opens its doors if the lift is already on that floor. The 2.2 s delay chain and the outer-door toggle are gone.
  - New function `SetOuterDoorAlpha(Alpha)` positions both outer door panels. OuterDoorOpen and the door's own timeline are no longer used.
- **BP_Lift**
  - Arrival fires from the movement timeline's Finished pin, followed by a 0.3 s delay.
  - Lift_Call_A/B share one gate: calls are ignored while the lift is busy (`bLiftMoving`). If the doors are open, the lift closes them, sets `bCallPending`, and moves once they're shut. The accepted floor is stored in `CallFromWhichFloor`.
  - InsideDoorOpen only opens doors (never closes). It is ignored while the lift is busy, and it resolves `Outer_LiftDoor_ref` for the current floor.
  - The inside-door timeline calls `SetOuterDoorAlpha` every tick, so both doors always move together.
  - Auto-close uses a 5 s Retriggerable Delay, started when the doors finish opening and restarted whenever they open. It only closes doors that are fully open.
  - Removed three dead Branch nodes.
- **Main level**
  - BP_Gate doors are named `<Nth> Floor Doors 1-9` on all six floors, in matching closed positions.
  - Floor surfaces (z): 2022.09, 1682.53, 1342.98, 1003.42, 663.86, 324.31, i.e. 339.6 apart. BP_Gate's pivot is at the bottom of the door.
  - Side windows copied to floors 1-5, labelled `<Nth> Floor <6th-floor label>`. The 6th-floor originals kept their old labels.
  - Deleted by the user: extra doors door55, door28 and door30.
- **Kitchen and living room swap sets (turn 17)**, both children of `BP_BaseEditModeInteractions` in `/Game/InteractionSystem/Blueprints/EditModeInteractions/`:
  - `BP_KitchenCabinate`: `Kitchen Set 1` = the original island, shelf, sink, freezer, cooktop, microwave, oven, 2 taps and 5 stools. `Kitchen Set 2` = the `Kitchen2ndSet` `_Alt` meshes in the same spots and scales. Actor anchored on the island at (-752.07, -225.02, 2025.04). Events `KS1` / `KS2`.
  - `BP_LivingRoomFurniture`: `Living Set 1` = the original sofa, 2 armchairs and side table. `Living Set 2` = the `LivingRoomSet` `_Alt_LOD0` meshes on the same spots, plus 8 `_JT` props placed from the Blender `ROOM_SCENE` layout (offsets from the sofa, scaled 1.5, on the floor at z 2023.66). Actor anchored on the sofa at (70, -310, 2030). Events `LS1` / `LS2`.
  - **How the swap works:** the tools can't author Timeline curves, and overriding Tick would skip the parent's widget-facing Tick. So BeginPlay starts a 0.016 s looping timer that calls `SwapTick`, which uses FInterpTo (speed `SwapSpeed` = 4) to ease `Alpha1` / `Alpha2` toward `Target1` / `Target2`. `KS1` / `LS1` retract Set 2, wait 0.8 s, then bring in Set 1; `KS2` / `LS2` do the reverse. `Set1_On?` / `Set2_On?` track the state.
  - **Per-mesh pivots (turn 18):** every mesh sits under its own SceneComponent `<mesh>_Pivot`, placed at the centre of the mesh's bounding box, with the mesh offset inside it so nothing moved. SwapTick runs `GetChildrenComponents(set root)` → `For Each Loop` and applies the existing scale (1 → 0.001) and spin (0 → 180 yaw) Lerps to each pivot, so each piece shrinks and turns about its own centre. The set roots stay at scale 1. Set 2 pivots default to scale 0.001 and yaw 180, so only Set 1 shows in the editor.
  - **Shelf and sink fix (turn 18):** the original shelf and kitchen-sink instances are mirrored (negative-determinant matrix). The turn-17 copies used rotation (0, -90, 180) with all-positive scale, which put them upside-down. They now use rotation (0, -90, 0) and scale (1.329693, **-1**, 1.215619), which matches the Datasmith matrix exactly (S1 and S2).
  - **Placed actors:** replaced in turn 18 (`BP_LivingRoomFurniture_C_1`) and again in turn 19 for the kitchen (`BP_KitchenCabinate_C_0`); labels unchanged. Instance overrides: Widget class and Widget height only.
  - **New Alt tall unit (turn 19):** the first `SM_Furniture__SM_Shelf_Alt` was a solid block that buried the sink, microwave, oven and fridge. The original `SM_Furniture__SM_Shelf` is an open frame, so the Alt was rebuilt to the same frame in Blender (`KitchenAlt_DarkLuxe.blend`, local cm, front = Blender -Y):
    - fridge alcove x -164.5..-54.7, z -124.25..53.7 (open through)
    - appliance column x -54.7..10.5 with an oven cavity (x -49.1..4.9, z -55.0..-9.6) and a microwave cavity (x -51.8..7.5, z -8.0..25.4), 57 cm deep, with smoked-bronze surrounds
    - sink base x 10.5..168.5 up to z -37.3 with an open top for the basin; open sink niche above with a dark marble backsplash on the back plane
    - upper fluted cabinet band z 53.7..124.25; fluted doors and bronze pull lines throughout
    - same outer bounds as the original (3.37 x 0.81 x 2.485 m), 6,108 tris, box UCX, UVmap_0 + LightMapUV, 4K BaseColor/Normal/ORM re-baked
  - In UE the new mesh and textures kept the original asset names; the old ones were renamed `*_OLD` (safe to delete once the new unit is approved). The material instance `M_SM_Furniture__SM_Shelf_Alt_Baked` now points at the new textures; Normal has Flip Green on, like before.
  - **Alt sink lowered:** the Alt sink's countertop was modelled at the top of its bounding box (15 cm above the original counter), which floated it above the base and buried the taps. `S2_Sink_Pivot` z went from 103.71 to 85.48 (-15 cm in the unit's scale) so the counter sits at the original height.
  - **Blender-side backups:** `Documents\KitchenAlt_DarkLuxe\_superseded_shelf_v1\` (old exports, textures and the .blend). The old Alt is kept in the .blend as `SM_Furniture__SM_Shelf_Alt_OLD`. UE backups: scratchpad `backup4_shelf_alt_ue\`.
  - **Widgets:**
    - `WBP_LivingRoom` is new (duplicate of `WBP_KitchenSlab`): Construct finds the living room blueprint, Button_1 calls `LS1`, Button_2 calls `LS2`.
    - `WBP_KitchenCabinate`: Construct is now connected (it finds the kitchen blueprint, then runs the existing stove and sink lookups), Button_3 calls `KS1`, Button_4 calls `KS2`. The old Material3 / Material4 SetMaterial nodes are left unconnected; Button_1 / Button_2 still swap materials.
  - **Widget class:** set on each placed actor (as BP_DinningTable does), not in the blueprints, so the base class is untouched.
  - **Level:** 18 sixth-floor HISM instances removed from 12 Datasmith actors (island, shelf, sink, freezer, cooktop, microwave, oven, 2 taps, 5 stools, sofa, 2 armchairs, side table). Floors 1-5 are untouched. Restore data is in `datasmith_removed_instances_floor6.json`, and `backup_level/Main.umap` holds the level from before the removal (both in the scratchpad).
  - **Tested (turn 18):** the swap animation itself, by temporarily setting Target1 = 1 / Target2 = 0 on the blueprint defaults and running Simulate.
  - **Tested (turn 19):** Set 2 kitchen layout in engine, with Set 2 forced on in Simulate and viewport captures: the sink niche, microwave/oven cavities and fridge alcove are all visible and filled.
  - **Not tested in play:** clicking the widget buttons. The living room props' positions also still need a visual check.
- **Backups:** the session scratchpad (outside the repo) has `backup/` (originals), `backup2/` (before the timing rework) and `door_restore_floors1-5.json`.
- **Not tested in play:** real lift rides. Test calls from other floors, pressing a floor button while the doors are open, and pressing buttons while the lift is moving.

- **Home button in first person (turn 21)**
  - **Flow traced:**
    - `BP_MasterMenu_Widget` `OnClicked(Button_UnitTour)` hides the POIs and calls `BP_Explorer_PC.Switch_Pawn("NewEnumerator2")`, then sets `IsinFP?` = true.
    - In `Switch_Pawn`, NewEnumerator2 calls the `Switch_to_FPP` function: it stores `OriginalPawn`, removes the master menu, possesses `BP_FirstPersonCharacter`, and sets Game-and-UI input on its `InteractingDotRef`.
    - `BP_FirstPersonCharacter` `EventPossessed` calls **Remove All Widgets**, then creates and adds a new `WBP_InteractingDot` and adds `IMC_Default`.
  - **Return path that already existed:** `Switch_Pawn(NewEnumerator0 = Main Explorer Pawn)` runs `Switch_to_MainExplorerPawn`, then possesses `Explorer_Main_Pawn` and restores the control rotation. Its second switch collapses the 360 menu, shows the master menu and shows the visible POIs. `Switch_to_MainExplorerPawn` does SetViewTargetWithBlend(OriginalPawn), UnPossess and AddToViewport(MasterMenu).
  - **Built in `WBP_InteractingDot`:** the existing `OnClicked(Main_Menu)` event (the home button) previously only printed "Hello". It now runs: GetOwningPlayer → Cast To BP_Explorer_PC → `Switch_Pawn(NewEnumerator0)` → `MasterMenu.IsinFP?` = false → `MasterMenu.Update_TaskbarButton_Style(MasterMenu.Button_Home)` (added in turn 22, so the taskbar highlight moves from Unit Tour to Home, as the menu's own Home button does) → Remove From Parent.
  - **Why the two extra steps:**
    - Nothing removed the dot widget when leaving first person, so the crosshair and buttons would stay on screen.
    - `IsinFP?` was never reset. The master menu's own `ExitUnitTour` checks it, so clicking Home, Surroundings or Amenities would have re-run the switch every time.
  - Compiled with warnings as errors and saved. Backup: scratchpad `backup5_home_button\`.
  - **Not tested in play:** the tools can't click UI.
  - **Known side effects, not changed:**
    - The first-person character's Remove All Widgets also removes the 360 menu, and nothing re-adds it, so the 360 view may have no menu after a first-person round trip.
    - Each dot widget's 0.01 s "Set Direction" timer keeps running after the widget is removed.

## Open items (as of Sep 19)

- **User tweaks pending:** the user will make some tweaks themselves and say if any need doing here.
- **Test the home button in play:** Unit Tour → Home → check the main menu, cursor, orbit input and POIs; then Unit Tour again.
- **Cleanup once approved:** delete the `*_OLD` shelf mesh and textures in `Kitchen2ndSet/SM_Furniture__SM_Shelf_Alt/`, and `SM_Furniture__SM_Shelf_Alt_OLD` in the Blender file.
- **Still to check in play:** kitchen and living room widget buttons, living room prop positions, relabelling the widget buttons, and lift rides.
- **Backups live in the session scratchpad** (a temp folder outside the repo). Copy anything worth keeping before the temp folder is cleaned.

## Session 2 notes: Unreal MCP quirks found
- **Unreadable nodes:** `get_node_infos` fails on some nodes (orphaned Self nodes, Cast, Create Widget, promotable operators). Skip them in batch reads.
- **Script errors roll back edits, but not always spawned actors:** a failing `execute_tool_script` usually rolls back its edits, but actors spawned with `add_to_scene_from_asset` were sometimes left behind. Check for leftovers after a failed run.
- **Don't compile inside a script that can fail:** a rolled-back compile left the level's BP_Lift actors on a `REINST_` class. Compile with direct calls. The fix was saving the blueprints and reloading the level.
- **Struct values:**
  - JSON struct values in `set_properties` only apply the first field. Use text, e.g. `(X=..,Y=..,Z=..)` or `(Pitch=..,Yaw=..,Roll=..)`, one property per call.
- **Array values:**
  - Arrays can only grow by one appended element per call.
  - Empty slots must be written as the string `"None"`.
- **Stale component references:** setting any property on a Blueprint instance re-runs its construction script. Look up component references again after every set.
- **Select node typing:** a new Select node keeps Wildcard option pins until it is rebuilt. Break one option link to force the rebuild, then connect `NewEnumerator0/1`.
- **Cross-blueprint calls:** a new function on another blueprint only shows up in `find_node_types` after the calling blueprint is compiled.
- **Biggest costs:** full graph dumps, long build scripts, and the 45-door move output. Return compact summaries instead.
- **Script timeouts:** an `execute_tool_script` running longer than 300 s is aborted, and the abort undid in-memory work from **earlier, completed** calls too (a blueprint created one call earlier, and the instance removal). Keep each script to about 60 s and **save to disk after every stage**.
- **Blueprint component templates:** `get_components` on a Blueprint needs its CDO (`.../BP_X.Default__BP_X_C`). Calling it repeatedly while adding components is very slow; use the reference `add_component` returns (`...BP_X_C:<Name>_GEN_VARIABLE`) instead. Inherited components come back as the **parent's** template, so editing them changes the parent class.
- **Parent calls:** "call parent function" nodes (`|Parent:Tick`) can't be created. Don't override an event the parent implements; use a timer or custom event instead.
- **Cross-blueprint custom event calls:** create them with `Class|<BPNameNoUnderscores>|<Event>` (e.g. `Class|BPKitchenCabinate|KS1`). `CallFunction|<Event>` only resolves with context pins.
- **Duplicated assets:** node IDs change when a Blueprint is duplicated, so look nodes up by listing the graph rather than by old IDs. A failed widget script left partial nodes behind, so check graph state after any failure.
- **Blueprint variables:** `set_properties` can't set them on a running (PIE / Simulate) instance, or on a placed instance unless they are Instance Editable. To test a state, set it on the CDO (`Default__BP_X_C`), run Simulate, then set it back.
- **Editing component templates doesn't reach placed actors:** after changing a Blueprint's component defaults with `set_properties` (and after reparenting), the actors already in the level kept their old transforms, and new components came in at SceneComponent defaults. `reset_properties` on the instance had no effect. Replacing the placed actor fixed it. Always verify placed instances after template edits.
- **Mirrored instances:** a Datasmith instance matrix with a negative determinant can't be reproduced with positive scale. Check `det(x, y, z rows)` and put the minus sign on one scale axis.
- **Reparenting components:** `ActorTools.set_parent_component` works on Blueprint templates. `add_component` with a SceneComponent template as `owner` attaches the new component under it.
- **Importing over an asset:** `StaticMeshTools.import_file` refuses to overwrite an existing asset. Rename the old one with `AssetTools.move` (references follow it), import under the original name, then repoint references.
- **Viewport captures:** `CaptureViewport` needs the `annotations` argument (all zeros, `classFilter: null`). It returns a base64 PNG; write it with `AssetTools.write_file` to `<Project>/Saved/*.txt` (only .txt/.json/.csv/.md/.py/.html are allowed) and decode it outside Unreal. Simulate lighting is dark, so brighten before judging.
- **Fresh texture imports** can report 32x32 from `get_size` until the texture finishes compiling.
- **Blender pipeline:** helper library `Documents\ArchvizAlt_Pipeline` (`kalt`, `kbake`, `kprev`); a 4K five-map bake of a 6K-tri mesh finished inside one 60 s MCP call on OPTIX.
- **When the `unreal-mcp` connector shows "failed"** (for example the editor was started after the session), the endpoint `http://127.0.0.1:8000/mcp` may still be up. Check with `Test-NetConnection 127.0.0.1 -Port 8000`. You can call it directly:
  1. POST `initialize` and read the `Mcp-Session-Id` response header.
  2. POST `notifications/initialized` with that header.
  3. POST `tools/call` for `list_toolsets`, `describe_toolset` or `call_tool`. The reply comes back as SSE, in a `data:` line.
  - Use **curl**. Python's urllib sends `Connection: close` and gets an empty body.
  - Session 2's helper for this was `scratchpad\ue.py` (commands `tools`, `call`, `tool`, `script <file>`). It's in a temp folder, so re-create it if it's gone.
- **Cross-BP custom event calls** show up in `find_node_types` without the underscore: `Class|BPExplorerPC|SwitchPawn` for `Switch_Pawn`. Enum pins take `NewEnumeratorN` strings via `set_pin_value`.
- **Datasmith furniture:** it is `HierarchicalInstancedStaticMeshComponent` instances on actors placed at the origin. Placement is in `PerInstanceSMData` (matrix rows). Shrinking that array with `set_properties` removes instances cleanly, as long as the remaining elements are unchanged.

---

# Session 3: Sheer drapes and BP_Curtain (Opus 5, Sep 20)

| Turn | Request | Outcome | Est. tokens |
|---|---|---|---:|
| 1 | `claude doctor` | Ran it; installation healthy, but the last auto-update attempt failed (`install_failed`) | ~2,000 |
| 2 | Design drapes + fixture in Blender, measured off the real windows | 5 meshes modelled, previewed and exported | ~150,000 |
| 3 | "are you connected to unreal now?" | Verified both connectors live | ~1,000 |
| 4 | Implement in Unreal as a child of `BP_InteractableComponentBaseClass`, like `BP_Gate` | `BP_Curtain` built, 6 actors placed on floor 6 | ~120,000 |
| | **Session total** | | **~273,000** |

## What Session 3 changed in the project

- **Window survey (floor 6).** Each apparent "window" is **two actors forming one glazed band**: a fixed `window*_polySurface*` pane plus an adjacent `BP_SideDoor` sliding unit. Six bands:

  | Band | Wall | Width | Head z | Sill z | Reveal |
  |---|---|---:|---:|---:|---:|
  | East | x ≈ 962–994 | 488.0 | 2212.7 | 2013.6 | 22.1 |
  | North-A | y ≈ 1587–1705 | 314.2 | 2216.2 | 2013.0 | 22.5 |
  | North-B | y ≈ 1587–1705 | 320.5 | 2211.2 | 2017.8 | 28.1 |
  | West | x ≈ −1008…−976 | 507.4 | 2188.8 | 2012.3 | 27.8 |
  | South-A | y ≈ −452…−340 | 308.0 | 2187.8 | 2016.2 | 6.6 |
  | South-B | y ≈ −452…−340 | 322.7 | 2186.5 | 2016.5 | 6.6 |

  Floor z **2022.09**, visible ceiling **2313.25** (slab soffit 2325.0, lintel underside 2222.9 on the East/North bands). Glazing runs below the floor line, so these are floor-to-head bands with no sill.

- **New Blender assets** (`C:\Users\pc\Documents\DrapesAlt\`, built with the existing `ArchvizAlt_Pipeline`, cm mesh data + 0.01 root scale, `UVmap_0` + `LightMapUV`, triangulated):
  - `SM_DrapeTrack_S` 320 × 5.4 × 2.95 cm, 208 tris — **dual-channel** (slots at y = ±1.3 cm) so a centre-parting pair does not interpenetrate
  - `SM_DrapeTrack_L` 500 cm version, 208 tris
  - `SM_DrapePanel_S` 161.2 × 15.1 × 193.9 cm, 15,800 tris, 17 carriers
  - `SM_DrapePanel_L` 251.2 × 15.1 × 193.9 cm, 24,080 tris, 26 carriers
  - `SM_DrapeBracket` 5.8 × 7.6 × 9.0 cm, 324 tris (arm needs ~1.7× X scale to reach past the 6.6 cm bedroom reveals)
  - Ripplefold geometry: carriers every 100 mm (±5.5% jitter), half-sine bows between carriers, amplitude 30 → 58 mm from heading to hem. Each fold also has a Y offset that is **zero at the carrier line and grows as it falls** — without it the gathered stack collapses into a flat slab.
  - Pivot is at the **top outer corner**; hem sits 194.46 cm below the origin.
- **New UE assets** in `/Game/ArchVizExplorer/Drapes/`:
  - `M_DrapeSheer` — **BLEND_Translucent, TwoSided, DefaultLit, SurfacePerPixelLighting, CastDynamicShadowAsMasked**. Opacity is a Lerp between `OpacityFacing` and `OpacityGrazing` driven by a Fresnel, which is what makes the folds read when backlit.
  - `MI_DrapeSheer` — OpacityFacing **0.82**, OpacityGrazing **0.995**, Roughness 0.90. **This is the one slider to touch if the drapes look too dense or too sheer.**
  - `M_DrapeTrack` / `MI_DrapeTrack` — matte black, metallic 0.85, roughness 0.34.
  - The existing `M_Master_Curtain` and `M_CurtainFinal` were **not** reused: both are `BLEND_Opaque` + `MSM_Subsurface` (thick drapes, not see-through), and flipping the master would have changed whatever already uses `MI_Curtain`.
- **`BP_Curtain`** in `/Game/InteractionSystem/Blueprints/PlayModeInteractions/`, created by **duplicating `BP_Gate`** so it keeps the parent `BP_InteractableComponentBaseClass` and the `BPI_InteractibleBP` interface.
  - Components: `Track`, `PanelL`, `PanelR` (added) alongside the inherited `DefaultSceneRoot`, `Widget`, `StaticMesh`.
  - Instance-editable variables: `BandWidth`, `Overlap` (6), `DropToFloor`, `PanelBaseWidth`, `TrackBaseWidth`, `OpenScaleX` (0.14), `OpenScaleY` (1.55), `SlideSpeed` (3), `TrackMesh`, `PanelMesh`. Computed: `ClosedScaleX`, `ScaleZ`. State: `Alpha`, `TargetAlpha`.
  - **ConstructionScript** assigns the meshes and lays the band out from `BandWidth`: track scaled to the opening, `PanelL` at (0, −1.3, 0), `PanelR` at (BandWidth, +1.3, 0) yawed 180°, both scaled `ClosedScaleX` × 1 × `ScaleZ`.
  - **Interact** (same shape as `BP_Gate`): ClickSound → FlipFlop → A sets `TargetAlpha` 1, B sets it 0, both start a 0.016 s looping timer on `CurtainTick`.
  - **`CurtainTick`** FInterpTo's `Alpha` toward `TargetAlpha` at `SlideSpeed`, lerps both panels' scale X (`ClosedScaleX` → ×0.14) and Y (1 → 1.55), and clears the timer once within 0.002. The Y figure is arc-length derived — at 1.35 the gather looked like a folded blind.
  - Note: `BP_Gate`'s own FlipFlop **B pin is empty**, so a gate opens once and never closes. `BP_Curtain` wires both.
- **Six actors placed on floor 6**, labelled `6th Floor Curtain <Band>`. Every hem lands at **z 2024.09** (2 cm above the floor). Panel X scales 0.995–1.038, Z scales 0.866–1.007, so the fold pitch stays correct on every band.
- **Backup:** `C:\Users\pc\Documents\DrapesAlt\backup_level\Main.umap` (taken before any actor was placed).
- **Not tested in play:** the interaction itself. The tools cannot drive first-person input, so open/close on look-and-interact is unverified.
- **Open question:** the drapes default to **closed**, which veils every view on floor 6. If the tour should start with the views visible, they need to default open.

## Session 3 notes: new MCP quirks found

- **`write_graph_dsl` cannot create Timeline nodes.** `read_graph_dsl` happily renders one as `(|Timeline 0.0)`, but writing it fails with `The node could not be created / |Timeline does not exist`. This confirms Session 2's finding from the other direction. Use the timer + `FInterpTo` pattern instead.
- **`write_graph_dsl` cannot wire getters for *inherited* components.** `(Variables|Default|GetStaticMesh)` resolves as a node type and the node is created, but its output never connects — the compile fails with "This blueprint (self) is not a SceneComponent, therefore 'Target' must have a connection", once per use. Getters for the Blueprint's **own** components work fine. Add your own components rather than driving inherited ones.
- Because of that, the FPP hover highlight (`SetRenderCustomDepth` on the interactable's `StaticMesh` component) has no target on `BP_Curtain` — the inherited `StaticMesh` is left empty. Assign a mesh to it by hand in the editor if the outline is wanted.
- **A failed `write_graph_dsl` rolls back cleanly** — the graph was byte-identical afterwards.
- **`CaptureViewport` needs BOTH `annotations` and `captureTransform` passed explicitly**, despite both being optional in the schema.
- **`FocusOnActors` does not take effect before a following `GetCameraTransform`** in the same script. Compute camera poses from `get_actor_bounds` instead.
- **`find_actors(name=...)` matches the actor LABEL, not the actor name.** Searching "Curtain_6F" found nothing; "Curtain" found all six.
- **`get_node_type_pins` really creates the node** in the target graph. The probes left `K2Node_CallFunction_8..12` behind; a later full `write_graph_dsl` cleared them.
- **`get_properties` raises if *any* requested property is unreadable**, aborting the whole script. Query per object type, or one property per call.
- **`StaticMeshTools.import_file` with `import_materials=False`** still preserves the FBX's material **slot names**, so slots can be filled afterwards with `set_material`.
- **Blender:** `kalt.ensure_uvs` wipes existing UVs. To keep a hand-authored `UVmap_0`, add `LightMapUV` yourself and smart-project only that channel.
- **`kprev.setup` hides every mesh not in its `show` list**, including `hide_set`. Unhide everything before any later `bpy.ops` work or operators silently fail.

---

# All sessions

| Session | Tokens |
|---|---:|
| Session 1 (main conversation ~132K + agent 145K) | ~277,000 |
| Session 2 (single conversation, compacted once) | ~1,069,500 (772,500 measured + ~297,000 estimated) |
| Session 3 (Blender drapes + BP_Curtain) | ~273,000 (estimated) |
| **Total** | **~1,619,000** |
