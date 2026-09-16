# Token Ledger: Lift outer-door task

These are estimates. Each figure is the drop in Claude's context counter after a step, so it shows roughly how much the step added to the conversation. It is not the billed amount. Every API call re-sends the whole conversation (mostly from cache), so billed input tokens are much higher than these numbers. Run `/cost` (or `/usage` on a subscription) for real session totals.

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
| Read-only trace of Lift_Call_A/B, door maps, BP_Lift_Door events | completed (48 tool calls, ~6 min 40 s) | **144,998** (reported by harness) |

This number is the harness's own count for the agent, not an estimate. The agent started with the full conversation above (~78K tokens, mostly cached).

**Finding:** the feature is already built. After the lift stops, both lifts look up the called floor in LiftADoorsByFloor / LiftBDoorsByFloor and call OuterDoorOpen on that door.

## Turn 6: Verify agent findings (Opus 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | get_node_infos: CallFunction_33/35/36/37 (Map Find + OuterDoorOpen) | confirmed | ~3,200 |
| 2 | Update this ledger | ok | ~1,000 |

## Turn 5: Ledger request (Opus 5)

| # | Step | Result | Est. tokens |
|---|------|--------|------------:|
| 1 | Write this ledger | ok | ~2,000 |

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

## Running total (main conversation, context growth)

| Item | Est. tokens |
|------|------------:|
| Turns 1-7 (main conversation) | ~132,000 |
| Background agent | 144,998 |
| **Task total** | **~277,000** |
| Wasted on errors (steps 1.5, 4.3, 4.5-4.7, 4.9 tool overhead) | ~5,000 |

## Notes
- **Model:** Turns 1-4 ran on Sonnet 5 and turn 5 on Opus 5. Per-token prices differ.
- **Biggest single cost:** the full EventGraph node list (~12K tokens). Later steps should use targeted `find_nodes` title filters.
- **Error waste:** most of it came from guessing MCP parameter names. The schemas are now known, so this shouldn't repeat.
