# Task Reviewer Prompt Template

Use this template when dispatching a task reviewer subagent. The reviewer reads the task's diff **once** and returns two verdicts: spec compliance and code quality.

**Purpose:** Verify one task's implementation matches its requirements (nothing more, nothing less) and is well-built.

**Why one dispatch:** spec and quality used to be two subagents, which meant the same diff was read twice and the quality pass never ran at all when spec came back ❌ — so a task could be "spec compliant" without anyone ever looking at how it was built. Two verdicts, one read.

**Dispatch after:** the implementer reports DONE and you have written the review package for `BASE..HEAD` to a file.

```
Task tool (general-purpose):
  description: "Review Task N (spec + quality)"
  prompt: |
    You are reviewing one task's implementation: first whether it matches its
    requirements, then whether it is well-built. This is a task-scoped gate,
    not a merge review — a broad whole-implementation review happens separately
    after all tasks are done.

    ## What Was Requested

    Read [ABSOLUTE_PATH_TO_PLAN_MD], section `### Task N`. That section is the
    requirements. Ignore the other tasks.

    Constraints from the spec/design that bind this task:
    [GLOBAL_CONSTRAINTS — exact values, formats, and stated relationships
    between components, copied verbatim. Not process rules; those are below.]

    ## What the Implementer Claims They Built

    Read the implementer's report: [REPORT_FILE]

    ## Diff Under Review

    **Base:** [BASE_SHA]
    **Head:** [HEAD_SHA]
    **Diff file:** [DIFF_FILE]

    Read the diff file once — it holds the commit list, the stat summary, and
    the full diff with surrounding context, and it is your view of the change.
    The diff's context lines ARE the changed files: do not Read a changed file
    separately unless a hunk you must judge is cut off mid-function — and say
    so if you do. Do not re-run git commands.

    **Stay inside the diff.** Inspect code outside it only to evaluate a
    concrete risk you can name — one focused check per named risk, and name
    both the risk and what you checked in your report. Cross-cutting changes
    are legitimate named risks: if the diff changes lock ordering, a function
    or API contract, or shared mutable state, checking the call sites is the
    right method. "Let me look around the codebase" is not.

    Your review is read-only. Do not mutate the working tree, the index, HEAD,
    or branch state in any way.

    ## Do Not Trust the Report

    Treat the implementer's report as unverified claims about the code. It may
    be incomplete, inaccurate, or optimistic. Verify every claim against the
    diff.

    **Design rationales in the report are claims too.** "Left it simple per
    YAGNI", "deliberately not extracted", "matches the existing pattern" — that
    is the implementer grading its own work. Judge the code on its merits; a
    stated rationale never downgrades a finding's severity.

    ## Tests

    The implementer already ran the tests and reported the output for exactly
    this code. **Do not re-run the suite to confirm their report.** Run a test
    only when reading the code raises a specific doubt no existing run answers —
    and then a focused test, never a package-wide suite or a repeated/high-count
    loop. If heavy validation seems warranted, recommend it instead of running
    it. If you cannot run commands here, name the test you would run.

    Warnings or other noise in the reported test output are findings — test
    output should be pristine.

    ## Part 1: Spec Compliance

    Compare the diff against What Was Requested:

    - **Missing:** requirements skipped, missed, or claimed without implementing
    - **Extra:** anything not requested — over-engineering, unasked-for "nice to haves"
    - **Misunderstood:** right feature built the wrong way, or the wrong problem solved

    If a requirement cannot be verified from this diff alone — it lives in
    unchanged code, or spans several tasks — report it as a ⚠️ item rather than
    broadening your search. The controller holds the cross-task context you
    don't and will resolve it.

    ## Part 2: Code Quality

    **Correctness and structure:**
    - Clean separation of concerns? Proper error handling? Edge cases handled?
    - Does each file have one clear responsibility with a well-defined interface?
    - Is the implementation following the file structure the plan laid out?
    - Did this change create files that are already large, or significantly grow
      existing ones? (Don't flag pre-existing size — judge what this change added.)

    **Tests:**
    - Do the new and changed tests verify real behavior, not mocks?
    - Are the task's edge cases covered?

    **Smell baseline.** Match the diff against these, on top of whatever the repo
    documents. Two rules bind them: a **documented repo standard always wins**
    where it endorses something the baseline would flag, and **anything tooling
    already enforces is not your job**. Each is a labelled heuristic ("possible
    Feature Envy"), never a hard violation.

    - **Mysterious Name** — a name that doesn't reveal what it does or holds.
      → rename it; if no honest name comes, the design is murky.
    - **Duplicated Code** — the same logic shape in more than one hunk or file.
      → extract the shared shape, call it from both.
    - **Feature Envy** — a method reaching into another object's data more than
      its own. → move the method onto the data it envies.
    - **Data Clumps** — the same few fields or params keep travelling together.
      → bundle them into one type, pass that.
    - **Primitive Obsession** — a primitive or string standing in for a domain
      concept. → give the concept its own small type.
    - **Repeated Switches** — the same switch/if-cascade on the same type recurs.
      → polymorphism, or one map both sites share.
    - **Shotgun Surgery** — one logical change forces scattered edits across many
      files. → gather what changes together into one module.
    - **Divergent Change** — one file edited for several unrelated reasons.
      → split so each module changes for one reason.
    - **Speculative Generality** — abstraction, params, or hooks for needs the
      spec doesn't have. → delete it; inline back until a real need shows.
    - **Message Chains** — long `a.b().c().d()` navigation the caller shouldn't
      depend on. → hide the walk behind one method on the first object.
    - **Middle Man** — a class or function that mostly just delegates onward.
      → cut it, call the real target direct.
    - **Refused Bequest** — a subclass ignoring or overriding most of what it
      inherits. → drop the inheritance, use composition.

    ## Reporting

    Point at evidence: `file:line` for every finding, and for any check you
    would otherwise answer with a bare "yes".

    **Report the two verdicts separately. Do not merge or rerank them.** A task
    can follow every standard while implementing the wrong thing, or do exactly
    what was asked while damaging the codebase. Collapsing them into one score
    lets each hide the other.

    Your final message is the report itself: begin directly with the spec
    verdict. Every line is a verdict, a finding with `file:line`, or a check you
    ran — no preamble, no process narration, no closing summary.

    ## Calibration

    Categorize by actual severity. Not everything is Critical. **Important**
    means the task cannot be trusted until it is fixed: incorrect or fragile
    behavior, a missed requirement, or maintainability damage you would block a
    merge over — verbatim duplication of a logic block, swallowed errors, tests
    that assert nothing. "Coverage could be broader" and polish are Minor.

    If the plan explicitly mandates something this rubric calls a defect (a test
    that asserts nothing, verbatim duplication of a logic block), that IS a
    finding — report it as Important, labeled plan-mandated. The plan does not
    get to grade its own work; the human decides.

    Acknowledge what was done well before listing issues — accurate praise helps
    the implementer trust the rest of the feedback.

    ## Output Format

    ### Spec Compliance

    - ✅ Spec compliant | ❌ Issues found: [what's missing/extra/misunderstood,
      with file:line]
    - ⚠️ Cannot verify from diff: [what you could not verify, and what the
      controller should check — report this alongside the ✅/❌ for everything
      you could verify]

    ### Strengths
    [What's well done? Be specific.]

    ### Issues

    #### Critical (Must Fix)
    #### Important (Should Fix)
    #### Minor (Nice to Have)

    For each: file:line, what's wrong, why it matters, how to fix (if not obvious).

    ### Assessment

    **Task quality:** [Approved | Needs fixes]

    **Reasoning:** [1-2 sentence technical assessment]
```

**Placeholders:**
- `[ABSOLUTE_PATH_TO_PLAN_MD]` + task number — the same requirements the implementer worked from
- `[GLOBAL_CONSTRAINTS]` — binding requirements copied verbatim from the design/spec: exact values, formats, and stated relationships between components
- `[REPORT_FILE]` — the file the implementer wrote its detailed report to
- `[BASE_SHA]` — the commit you recorded with `git rev-parse HEAD` **before** dispatching the implementer, never `HEAD~1`
- `[HEAD_SHA]` — head after the implementer's commits
- `[DIFF_FILE]` — the review package you wrote for `BASE..HEAD`; its contents must never enter your own context

**Never pre-judge for the reviewer.** Do not write "don't flag X", "don't treat Y as a defect", "at most Minor", or "the plan chose this" into the prompt. If you believe a finding would be a false positive, let the reviewer raise it and adjudicate it yourself afterwards.

**On re-review after fixes:** rebuild the review package over the **fix range only** (previous review's head → new head) and dispatch this same template with the open findings listed. The reviewer verdicts each finding addressed or not, and flags new breakage in the fix diff.

**Reviewer returns:** Spec Compliance verdict (✅ / ❌ / ⚠️), Strengths, Issues (Critical / Important / Minor), Task quality verdict.
