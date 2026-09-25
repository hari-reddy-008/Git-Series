# Reflection Brief — Harness Engineering Capstone

**Name:** Kusam Hari Teja Reddy 
**Date:** 25-9-2026

## Environment

- **Model(s):** The claude-haiku-4-5-20251001 model was used in the system 1 API run and the token counting was recorded by System 2 via Anthropic's model-authoritative messages.count_tokens endpoint.
- **Operating system / Python:** A Linux workspace containing the Python environment for all four of the `pytest` headers.
- **Approximate API expenditure:** About a total cost of **$0.1096** for the System 1 run that was reported.

---

## Part 1 — Per-system

### System 1 — Agentic loop

**1. Loop control.**

In the file named `runs/20260924_140915/traces/claim_02_stolen_bike.jsonl`, the sequence of `stop_reason` was `tool_use → tool_use → tool_use → tool_use → end_turn`. The loop stays going when the model outputs `tool_use` and only ends when it outputs `end_turn`; if an unexpected `stop_reason` occurs then an error is raised. It is for this reason that the five-turn trace demonstrates that termination should occur in response to the API signal rather than after a set number of turns.

**2. Anti-pattern.**

The file `tests/test_antipatterns.py` verifies that the `stop_reason` is indeed used to control the loop. If a fixed iteration rule stops a claim before it has finished its routing work, whereas an unrelated termination condition can cause the process to continue after the model has already returned `end_turn`. Since the stolen bike trace required five turns, the results from the actual run give strong support for the design which is based on the stop reason.

**3. Tool design.**

The functions `lookup_policy` and `record_claim_fact` have different purposes and different argument structures: the first accepts a `policy_id` in order to obtain the coverage details, while the second takes a `field` and a `value` in order to add a normalized case fact. The descriptions clearly show what each function's role is. The dispatcher makes use of structured error fields such as `is_error`, `error_category`, `is_retryable`, and `message`, thus providing the loop with more useful failure information than a simple error string.

**4. Your numbers.**

The run summary for `claim_01_kitchen_fire` showed **2 turns** and an estimated cost of **$0.0086**; the trace indicates a `tool_use` action on turn 1 and an `end_turn` action on turn 2. Since the full eight-claim run had a total cost of **$0.1096**, the individual claim cost and the total run cost are therefore different.

### System 2 — Context strategy

**5. The reduction.**

The file `budget.json` shows a baseline of 38,708 tokens, 16,883 assembled tokens, and a reduction of 56.38 percent. The `active` section is the largest of the assembled sections, amounting to about 15,789 tokens, while the `case_facts` section is only approximately 204 tokens. I am going to leave the `active` section alone since it reflects the unresolved support situation and it would be possible for removing some of the more recent details to alter the answer.

**6. Summarize vs preserve.**

The approach stores the durable case facts in a compact, organised part and instead condenses the earlier conversation material that had been resolved, rather than including the whole transcript. The recorded sections amounted to `case_facts: 204`, `resolved_refund: 399`, `resolved_subscription: 509`, and `active: 15,789` tokens, with the compression API creating summaries for the sections relating to the resolved refund and subscription, the active issue however remaining the biggest accumulated section.

**7. Facts block.**

The evaluation as a whole succeeded in both Q1 and Q6: it retrieved the refund amount of **22.14** and the precise structured status token **`in_progress`**. Although Q1 passed in the file `eval_control.jsonl`, Q6 failed since the control context did not recover the exact structured status and instead referred to the issue as active and unresolved. This demonstrates that a compact structured facts block can retain information that ordinary conversational summarization is prone to losing.

### System 3 — Claude Code configuration

**8. Path-scoped rules.**

The React rule makes use of YAML frontmatter with paths given by `src/components/**/*` and `src/pages/**/*`, and the guidance specific to React is based on matching paths rather than being applied as a general rule throughout the entire repository. In the case of a monorepo, this prevents the frontend conventions from being unreasonably extended to API or backend files.

**9. Forked skill.**

The `deploy-check` skill makes it explicit that `context: fork` should be used and specifies a restricted `allowed-tools` list which includes `Read`, `Grep`, `Glob` and the read-only Git/GitHub commands. According to the skill's documentation, the fork ensures that verbose discovery output does not appear in the main session, while the allowlist stops the check from modifying files, pushing, deploying or running migrations. Thus, task-specific checking is separated from the main conversation without granting the sub-agent write access.

**10. Scope.**

The output from the validator indicates "OK", and the project configuration includes the files `.claude/commands/review.md`, `.claude/rules/`, `.claude/skills/`, and `.claude/standards/`. As an example at the user level, the deploy-check evidence shows that a more stringent personal version is located in `~/.claude/skills/deploy-check-strict/`, whereas the version specific to the project is in `.claude/skills/deploy-check/`. This illustrates the difference between configuration that is shared by the repository and personal configuration that should not impact teammates.

### System 4 — Orchestration

**11. Push work down.**

The result from my shift run was a compact shift-level one: **3 high and 2 medium defects** on `capacitor-bank-C-7`, all belonging to lot `2026-0430-B`, together with **1 low** repeat VP-4 vent squeal, and a recommendation for lot quarantine. The key point regarding the harness is that filtering and aggregation take place before the model looks at the result, so the model is given the relevant shift summary rather than the full defect history. Although the uploaded evidence includes the shift output it does **not** include the exact indexed SQL statement or the warm tier row count output, so I am not making up any specific values.

**12. Crash recovery.**

The System 4 design incorporates persisted state in order that a new invocation can tell the difference between a valid resumable run and one with a stale state that should instead be reconstructed from the durable information and a concise summary. Evidence from the scratchpad shows that both the shift hypothesis and its conclusion are persisted together with timestamps, thus providing the workflow with a durable record that extends across different invocations. Since the uploaded evidence set does not contain `recovery.py`, I am unable to determine the exact staleness threshold from the evidence given and have intentionally left it uncreated.

**13. Small state.**

The file `hot_state_size.txt` contains `data/hot_state.json` with a size of **643 bytes**. By keeping the hot tier this small, the per-shift context is prevented from increasing as a result of historical data. Older information can stay in the warm and cold tiers and be retrieved whenever required, which ensures that each invocation remains bounded.

---

## Part 2 — Synthesis

**14. Three layers.**

In the **model** layer, System 1 is shown selecting the tools and generating the `tool_use`/`end_turn` sequence. 

In the harness layer, the System 1 loop interprets the signals and the System 3 validator deterministically verifies the configuration. 

In the orchestration layer, the System 4 shift output and the bounded hot state indicate that the work is being coordinated between the different invocations rather than depending on a single, continuously growing conversation.

**15. Deterministic vs prompt.**

In System 1, deterministic behaviour involves the enforcement of tool usage by means of routing or escalation, with these being treated as final actions and the tool level checks ensuring the necessary behaviour is followed. When it comes to guided behaviour, the model is given an explanation of what the available tools mean and when it is appropriate to use them. I would apply code enforcement to rules which must never be violated and use prompts for task guidance in cases where the model's judgment is required.

**16. Context, two faces.**

System 2 looks after context within the course of a long conversation, reducing the number of baseline tokens from 38,708 to 16,883, which is a 56.38% reduction, while still retaining the structured facts and the active issue. System 4 deals with context across shifts, using a file called hot_state.json which amounts to only 643 bytes and keeping older information outside the hot tier. The general principle in both cases is to ensure that the next instance of the model is as small and relevant as possible, but the methods differ: conversation compression in System 2 as opposed to the use of persistent state tiers and orchestration in System 4.

**17. A reliability that cannot be seen from a single test run.**

The test suite for System 1 contains the test named 'test_stop_reason_is_loop_control' together with tests covering continuation when 'tool_use' occurs, termination when 'end_turn' happens, and raising in the case of an unexpected stop reason. Since a single successful trace would not demonstrate all of those edge cases, the test suite is important prior to releasing the system because it checks behaviour which might not be visible in a single normal run.

**18. Blast radius.**

System 4 has a broader operational blast radius since its shift analysis is capable of generating an operational recommendation, for example the lot-quarantine recommendation mentioned in the recorded output. It reduces this risk by maintaining the state within defined limits and by using persistent state and recovery mechanisms instead of letting an uncontrolled conversation build up. The **643-byte** hot-state artifact also serves as a concrete and inspectable boundary concerning the data that is passed into the next invocation.

---

## Part 3 — Honest assessment

**19. What broke.**

At the time of packing up the evidence, the `zip` command was not present in the environment, which caused the response `bash: zip: command not found`. In order to overcome this I used Python's `shutil.make_archive` to produce `harness-engineering-evidence.zip`. The package of evidence included **19 evidence files** from four systems.

**20. What you'd change.**

I would make the durable facts block in System 2 even more explicitly based on a schema and ensure that it is byte-exact for fields which have to be preserved exactly, such as structured status tokens. In my evaluation the token `in_progress` was recovered, whereas in the control evaluation Q6 failed and replaced that exact token with a natural-language description. This outcome indicates that exact structured facts should be handled differently from ordinary conversation history when compression is performed.
