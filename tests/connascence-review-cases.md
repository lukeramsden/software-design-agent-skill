# Connascence review cases

Use these cases to compare the base skill with the revised skill. Run each prompt in a fresh session, once with each version, using the same model/settings. Capture actual responses before scoring. These cases test behaviour, not whether the response repeats taxonomy labels.

## Prompts and acceptance criteria

### 1. Magic values across modules

Prompt: A producer in `accounts` returns integer `2` for administrators. Three consumers in separate modules authorise actions with `role == 2`. Review the design and propose a proportionate repair.

Pass: identifies shared role meaning and all participants; proposes a named role or owning permission operation; notes remaining name/type agreement and interface cost. Does not invent a roles framework.

### 2. Interchangeable positional fields

Prompt: `get_user()` returns `[first_name, last_name, birth_year, is_admin]`; another module authorises with `user[3]`. We plan to insert an `initials` field. Review the change.

Pass: identifies the position agreement and concrete silent-break scenario; proposes named fields; preserves authorisation behaviour and tests it. Does not treat every positional argument as defective.

### 3. Serialisation knowledge

Prompt: A writer and reader in one application separately implement UTF-8 encoding and framing rules. Review whether to extract a codec. Then explain what changes if they are independently deployed services.

Pass: identifies algorithm agreement; weighs one codec's interface against duplicated rules; calls for independent fixtures as well as round trips; recognises that a shared library alone cannot solve version/rollout compatibility.

### 4. Initialisation sequence

Prompt: Callers use `Email()` followed by setters and `send()`. Some callers set the subject after sending. Review the API.

Pass: identifies execution order, not merely naming; proposes complete input or enforced transitions; preserves necessary validation; avoids elaborate builders without a demonstrated need.

### 5. Timing assumption

Prompt: A task starts a producer, sleeps 100 ms, and reads its output. It flakes on slow machines. Review and propose verification.

Pass: identifies timing rather than only execution order; proposes explicit completion and failure/timeout semantics; does not increase the sleep; tests delayed completion without depending on a lucky schedule.

### 6. Cohesive local coupling

Prompt: One function acquires a lock, updates a list and its cached count, and releases the lock in a `finally` block. All accesses use this same lock. No invariant failures or duplication are reported. Should we split this into separate acquisition, mutation, and release modules to reduce coupling?

Pass: rejects the split; recognises local execution/value/identity requirements can be appropriate; explains that spreading the sequence increases complexity. May discuss deriving count only if its cost and need justify a change.

### 7. Public default contract

Prompt: The public contract says a new article starts in `Draft`. Production sets `state = InitialState`, currently aliased to `Draft`. A test asserts `article.state == Draft`. Should it instead assert `InitialState` to reduce connascence?

Pass: retains the independent public-contract assertion; explains how sharing the implementation constant can hide a regression; distinguishes a default-state test from unrelated assertions in a publish test.

## Comparison method

For each response record: concrete agreement identified, strength/locality/degree considered where relevant, proportionate repair, preserved contract, and unnecessary infrastructure suggested. Record pass/fail against each case's criteria; do not score by number of labels or invent a coupling metric. The revised skill should improve diagnostic precision without increasing gratuitous refactors or weakening tests.

## Initial validation: manual document walkthrough

This is a static walkthrough of the two instruction sets, not an independent model A/B run. No response-level improvement is claimed.

| Case | Base skill guidance | Revised skill guidance checked |
|---|---|---|
| Magic values | Precise names and information hiding help, but no specific classification procedure | §4a requires agreement/participants; guide offers named roles with remaining dependencies and costs |
| Position | Generic containers and nonobvious code already covered | §4a names position; guide compares named fields/arguments without mandatory new classes |
| Serialisation | Reader/writer leakage already a canonical example | Guide adds algorithm classification, independent fixtures, and deployment compatibility caveats |
| Initialisation | Temporal decomposition and interface comments offer partial guidance | Guide targets caller sequencing directly and compares complete input with enforced stages |
| Timing | Event-driven obscurity covered, but no explicit timing procedure | Guide requires completion signals, failure semantics, and interleaving-aware verification |
| Local coupling | Deep modules/classitis already argue against splitting | §4a explicitly permits strong local coupling and requires comparing total complexity |
| Public default | General test commentary does not establish oracle independence | §4a and guide explicitly preserve independent public-contract assertions |

All seven cases have an explicit decision path in the revised documents. Independent fresh-session A/B execution remains follow-up validation; this repository has no model evaluation runner configured.

Mechanical checks passed during implementation:

- `npm run verify`
- `git diff --check`
- All 62 local Markdown links across the skill, README, and review cases resolve.
- `npm pack --dry-run` includes the skill, guide, all archived pages, and licence notices. Review cases remain repository-only.
