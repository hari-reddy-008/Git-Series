# Project Harness Engineering — Reflection Brief

**Name:** Hari Teja Reddy
**Date:** 2026-09-25

## Environment
- **Model(s):** The (claude-haiku-4-5-20251001 model) was employed in the system 1 API run and the token counting was recorded by system 2 using Anthropic's model-authoritative messages.count_tokens endpoint.
- **Operating system / Python:** A Linux environment consisting of the Python setup used for all four of the `pytest` headers.
– **API expenditure and results:** $0.1096, with System 1 having 29 passed, System 2 having 28 passed and 2 skipped, System 3 having 35 passed, and System 4 having 33 passed.

**1. Loop control.**
In the file called `runs/20260924_140915/traces/claim_02_stolen_bike.jsonl` the trace indicates `tool_use → tool_use → tool_use → tool_use → end_turn`. The way the loop is implemented involves checking the value of `response.stop_reason`: if the result is `tool_use` then the loop continues, but when it is `end_turn` the process returns the result. It is therefore the model's API that signals the termination condition rather than using a fixed number of iterations.

**2. Anti-pattern.**
The System 1 tests verify that the `stop_reason` is actually employed as loop control; otherwise a fixed iteration limit might cause processing to stop before routing is complete, and a condition that has nothing to do with the task could make the process continue after the model has returned `end_turn`. The stolen-bike trace showed that five turns were required, which illustrates the need for the loop to follow the API signal.

**3. Tool design.**
In the file `claims_intake/tools.py` the function `lookup_policy` takes as its argument a `policy_id`, whereas the function `record_claim_fact` takes a `field` and a `value`. During run `20260924_140915` the claim trace indicates that these tools were used as part of the processing workflow; the fact that they have separate schemas means that the model has different interfaces for retrieving policy information and for recording normalized facts.

**4. Your numbers.**
The run summary for `claim_01_kitchen_fire` showed 2 turns and an estimated cost of **$0.0086**; the full eight-claim run had an estimated total cost of **$0.1096**. These figures are based on the actual run and not on the approximate example cost given in the exercise documentation.

**5. The reduction.**
The baseline is 38,708 tokens and the assembled context 16,883 tokens, which represents a reduction of 56.38%. The active section stayed at 15,789 tokens while the durable case-facts sections were considerably smaller. This indicates that the resolved conversation history can be compressed without compromising the active conversation.

**6. Summarize vs preserve.**
The context strategy distinguishes between permanent facts and the part of the conversation that has already been resolved. According to my `budget.json` file, the case facts take up 204 tokens, whereas the two compressed sections relating to the resolved part of the conversation have 399 and 509 tokens respectively. By doing this, structured information needed for future questions is kept while at the same time the amount of storage required for earlier conversation content is reduced.

**7. Facts block.**
The results of the evaluation indicate that all six questions have been passed, including the precise structured 'in_progress' status in question six. The control evaluation fails question six on purpose since the compact structured status is lost when the case facts are deleted. This shows the reason why important exact facts should be kept in a durable facts block.

### System 3 — Claude Code configuration

**8. Path-scoped rules.**
Rules in the System 3 system make use of file-path globs such as `src/components/**/*`, `src/pages/**/*`, and the testing rule's `**/*.test.tsx`. This is advantageous when related conventions apply across various surfaces in a monorepo, for example, components, pages, and API code. One directory level can have a naturally occurring `CLAUDE.md` file which applies to that particular directory hierarchy, whereas rules that are based on paths can target files that match across different directories.

**9. Forked skill.**
The deploy-check skill makes use of context: fork and has an explicit read-only allowed-tools list; this ensures that the deployment validation is kept separate from the main session and at the same time limits the operations available to the skill. The project-scoped skill is thus both isolated and constrained.

**10. Scope.**
The deploy-check skill is scoped to the project and can be found in the directory `.claude/skills/deploy-check/`. Alternatively, a more strict version for individual use could be placed in `~/.claude/skills/`. The project location ensures that the team's shared practices remain linked to the repository, while the location at the user level is suitable for personal conventions.

**11. Push work down.**
During the System 4 run, the warm store had 40 fixture rows and the historical query produced 17 rows with N equal to 40 and `since=2026-04-01T00:00:00`. The query used was `SELECT * FROM defects WHERE ts > ? ORDER BY ts DESC LIMIT ?`, and SQLite indicated that it had carried out a search using the index idx_defects_ts with the condition (ts>?). The run log showed `shift=C` together with `new=17`, which means that the pipeline had retrieved the historical warm-tier data prior to processing the shift.

**12. Crash recovery.**
The 30-minute staleness threshold is used in the crash-recovery implementation, the value of this threshold being set as `STALE_RESUME_THRESHOLD_MINUTES = 30` in the file `shift_monitor/recovery.py`, and the test suite checks that this threshold is correct; if the state is within this time frame it can be resumed but partial state that is older than the threshold is considered stale and is therefore restarted. The script `fork.py` copies the hot state into a separate hypothesis directory that is isolated from the base state without altering the base state, and the fork tests confirm that the various hypotheses have their own independent scratchpads.

**13. Small state.**
The System 4 run resulted in a small hot-state artifact, and the recorded hot-state size was stored separately in a file called `hot_state_size.txt`. Rather than storing the complete defect history, the design retains the current shift information and any active alerts in the hot tier. As a result, the warm SQLite database contains the historical details while the hot state stays compact.

---

## Part 2 — Synthesis

**14. Three layers.**
The System 4 design ensures that the hot state, the warm historical data, and the forked investigation state are separated. For the run, the warm SQLite database was used to store the historical defects, while the shift output and the scratchpad were used to capture the current workflow; the fork tests then show how hypothesis-specific state can remain isolated before the findings are merged.

**15. Deterministic vs prompt.**
The guidance for the System 1 tool definitions is aimed at the model, but terminal routing and escalation are carried out as tool operations rather than depending solely on natural-language instructions. As shown in the `claim_02_stolen_bike.jsonl` trace, the model proceeds through a number of `tool_use` steps before reaching `end_turn`. This arrangement separates the deterministic behaviour of the application from the guidance provided at the prompt level.

**16. Context, two faces.**
The findings from System 2 indicated that context engineering has both a aspect relating to the token budget and one relating to correctness; the budget decreased from 38,708 to 16,883 tokens, whereas the evaluation kept the same structured status that was required by Q6. It therefore follows that optimising only the number of tokens could result in information necessary for giving correct answers being removed.

**17. A reliability which cannot be determined from a single test run.**
The evaluation using System 2 control was useful since it deliberately got rid of the compact representation of the facts and therefore failed question six. While the standard evaluation succeeded in all six questions, the control demonstration showed what would go wrong in the absence of durable facts. This meant that the reliability boundary could be seen rather than depending merely on a single successful run.

**18. Blast radius.**
The deployment validation in the System 3 forked deploy-check design is restricted in the operations it can carry out. Similarly, the System 4 fork carries out a test that isolates the hypothesis scratchpads and safeguards the base state. In both cases, the amount of unrelated state that a single operation can affect is reduced.

---

## Part 3 — Honest assessment

**19. What broke.**
The key evidence issue was the initial eight-hour window set as the default for System 4, during which no new defects were recorded. When the seeded 40-row warm database was used with an explicit April 1 timestamp, 17 historical rows were obtained together with a meaningful Shift C run. It thus became clear that evidence collection should make use of the intended historical fixture window and not depend on the CLI default.

**20. What I should change**
While running each system I would gather the evidence artifacts rather than reconstructing them at the end; specifically, I would save the relevant traces, the query-plan output, the validator output, and the state-size measurements right after each successful run. This approach would mean that the final submission could be put together more quickly and would also lower the risk of missing an artifact.
