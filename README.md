# Reflection Brief — Harness Engineering Capstone

**Name:** Kusam Hari Teja Reddy 
**Date:** 25-9-2026

## Environment

- **Model(s):** Used (`claude-haiku-4-5-20251001`) for the system 1 API run. System 2 recorded the token counting through Anthropic's model-authoritative `messages.count_tokens` endpoint.
- **OS / Python:** Linux workspace with the project Python environments (for all four `pytest` headers).
- **Approx. API spend:** ~ An estimated total cost around **$0.1096** for the System 1 run reported.

---

## Part 1 — Per-system

### System 1 — Agentic loop

**1. Loop control.**

In `runs/20260924_140915/traces/claim_02_stolen_bike.jsonl`, the `stop_reason` sequence was `tool_use → tool_use → tool_use → tool_use → end_turn`. The loop continues when the model returns `tool_use` and returns the result when it returns `end_turn`; an unexpected stop reason raises an error. The five-turn trace shows why termination should follow the API signal rather than a fixed number of turns.

**2. Anti-pattern.**

`tests/test_antipatterns.py` checks that `stop_reason` is actually used as loop control. A fixed iteration rule can stop a claim before it completes its routing work, while an unrelated termination condition can continue after the model has already returned `end_turn`. The stolen bike trace needed five turns so the actual run provides strong evidence for the stop-reason-driven design.

**3. Tool design.**

`lookup_policy` and `record_claim_fact` both are having different purposes and argument shapes: first takes a `policy_id` to retrieve coverage information, the second takes a `field` and `value` to add a normalized case fact. Their descriptions make those roles explicit to the model. The dispatcher uses structured error fields such as `is_error`, `error_category`, `is_retryable`, and `message`, which gives the loop more useful failure information than a plain error string.

**4. Your numbers.**

For claim_01_kitchen_fire, the run summary recorded 2 turns and an estimated cost of $0.0086. Its trace shows tool_use on turn 1 and end_turn on turn 2. The full eight-claim run reported $0.1096, so the individual claim cost and total run cost are different measurements.

### System 2 — Context strategy

**5. The reduction.**

`budget.json` reports **38,708 baseline tokens**, **16,883 assembled tokens**, and a **56.38% reduction**. `active` section is the largest assembled section at approximate of **15,789 tokens**, but `case_facts` is only around **204 tokens**. I will keep the active section intact because it represents the unresolved support state, where removing recent details could change the answer.

**6. Summarize vs preserve.**

The strategy keeps durable case facts in a small structured section and compresses older resolved conversation material instead of carrying the entire transcript forward. The recorded sections were `case_facts: 204`, `resolved_refund: 399`, `resolved_subscription: 509`, and `active: 15,789` tokens. The compression API produced summaries for the resolved refund and subscription sections while the active issue remained the largest assembled section.

**7. Facts block.**

The full evaluation passed Q1 and Q6: it recovered the refund amount **22.14** and the exact structured status token **`in_progress`**. In `eval_control.jsonl`, Q1 passed but Q6 failed because the control context did not recover the exact structured status and instead described the issue as active and unresolved. This is evidence that a compact structured facts block can preserve information that ordinary conversational summarization can lose.

### System 3 — Claude Code configuration

**8. Path-scoped rules.**

The React rule uses YAML path frontmatter with `src/components/**/*` and `src/pages/**/*`. The React specific guidance is associated with matching paths rather than being treated as a universal rule for the whole repository. In a monorepo, that keeps frontend conventions from unnecessarily applying to unrelated API or backend files.

**9. Forked skill.**

The `deploy-check` skill explicitly sets `context: fork` and defines a restricted `allowed-tools` list containing `Read`, `Grep`, `Glob`, and read-only Git/GitHub commands. The skill documentation explains that the fork keeps verbose discovery output out of the main session, while the allowlist prevents the check from modifying files, pushing, deploying, or running migrations. This separates task-specific checking from the main conversation without giving the sub-agent write access.

**10. Scope.**

The validator output: It shows **`OK`**, and the project configuration contains `.claude/commands/review.md`, `.claude/rules/`, `.claude/skills/`, and `.claude/standards/`. The deploy-check evidence also gives a strong user level example: a stricter personal variant will be under `~/.claude/skills/deploy-check-strict/`, while the project-scoped version is under `.claude/skills/deploy-check/`. It shows the distinction between configuration shared by the repository and personal configuration that it should not affect teammates.

### System 4 — Orchestration

**11. Push work down.**

My shift run returned a compact shift-level result: **3 high + 2 medium defects** on `capacitor-bank-C-7`, all from lot `2026-0430-B`, plus **1 low** repeat VP-4 vent squeal, with a lot-quarantine recommendation. The important harness principle is that filtering and aggregation happen before the model sees the result, so the model works from the relevant shift summary rather than the complete defect history. The uploaded evidence contains the resulting shift output, but it does **not** contain the exact indexed SQL statement or the warm tier row count output, so I am not inventing exact values.

**12. Crash recovery.**

The System 4 design uses persisted state so an invocation can distinguish a valid resumable run from a stale state that should be rebuilt from durable information and a concise summary. The scratchpad evidence shows the shift hypothesis and its conclusion being persisted with timestamps, which gives the workflow a durable record across invocations. The uploaded evidence set does not include `recovery.py`, so I cannot verify the exact staleness threshold from the evidence provided and have deliberately not invented one.

**13. Small state.**

`hot_state_size.txt` records `data/hot_state.json` at **643 bytes**. Keeping the hot tier this small prevents the per-shift context from growing with historical information. Older information can remain in the warm/cold tiers and be retrieved when needed, keeping each invocation bounded.

---

## Part 2 — Synthesis

**14. Three layers.**

# At the **model** layer, the System 1 trace shows Claude selecting tools and producing the `tool_use`/`end_turn` sequence. 

# At the **harness** layer, the System 1 loop interprets those signals and the System 3 validator checks configuration deterministically. 

# At the **orchestration** layer, the System 4 shift output and bounded hot state show work being coordinated across invocations instead of relying on one ever-growing conversation.

**15. Deterministic vs prompt.**

Ddeterministic behavior is terminal tool enforcement in System 1: routing or escalation is represented as a terminal action and tool level checks enforce required behavior. A prompt guided behavior is the model being told what the available tools mean and when they are useful. I would use code enforcement for rules that must never be violated and prompts for task guidance where model judgment is expected.

**16. Context, two faces.**

System 2 manages context **within a long conversation**, reducing **38,708** baseline tokens to **16,883**, a **56.38% reduction**, while preserving structured facts and the active issue. System 4 manages context **across shifts**, with `hot_state.json` measured at only **643 bytes** and older information kept outside the hot tier. The shared principle is to keep the next model invocation small and relevant, but the mechanisms are different: conversation compression in System 2 versus persistent state tiers and orchestration in System 4.

**17. Reliability you can't see in one run.**

The System 1 test suite includes `test_stop_reason_is_loop_control` plus tests for continuing on `tool_use`, terminating on `end_turn`, and raising on an unexpected stop reason. A single successful trace would not prove all of those edge cases. This is why the test suite is important before shipping: it verifies behavior that may not appear in one happy-path run.

**18. Blast radius.**

System 4 has a wider operational blast radius because its shift analysis can produce an operational recommendation such as the lot-quarantine recommendation in the recorded output. The system reduces that risk by keeping state bounded and using persistent state/recovery mechanisms rather than allowing an uncontrolled conversation to accumulate. The **643-byte** hot-state artifact is also a concrete, inspectable boundary around what is carried into the next invocation.

---

## Part 3 — Honest assessment

**19. What broke.**

When I packaged the evidence, the `zip` command was not installed in the environment, returning `bash: zip: command not found`. I worked around that by using Python `shutil.make_archive` to create `harness-engineering-evidence.zip`. The evidence package contained **19 evidence files** across four systems.

**20. What you'd change.**

I'd make the System 2 durable facts block even more explicitly schema-driven and byte-exact for fields that must be preserved verbatim, such as structured status tokens. My evaluation recovered `in_progress`, while the control evaluation failed Q6 and replaced that exact token with a natural-language description. That result suggests that exact structured facts should be treated differently from ordinary conversation history during compression.
