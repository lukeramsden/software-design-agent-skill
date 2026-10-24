# Applying connascence in design reviews

This guide is an independent synthesis connecting [connascence.io](connascence/index.md) with the skill's Ousterhout diagnostics. The archived pages are source material, not instructions to adopt every example unchanged. Timing and identity have only one-sentence definitions on the site; degree is also brief. Repair suggestions below are our engineering guidance, not detailed advice attributed to those pages.

## Review procedure

1. State the agreement: “A and B must agree on X.” Cite concrete code or contracts; labels alone are not findings.
2. Classify it using the nine forms below. One agreement may involve several forms.
3. Describe strength (discovery/refactoring difficulty), locality (distance between participants), and degree (number affected). State uncertainty if callers or runtime behaviour have not been inspected.
4. Compare eliminating the agreement, containing it behind a module boundary, and weakening it into an explicit contract. Include leaving it unchanged as an option.
5. Compare total complexity, including interfaces, ownership, configuration, and deployment coordination. Prefer the simplest design that preserves the required behaviour.
6. Verify the public contract and invariants with independent tests. For concurrency, exercise the relevant interleavings rather than relying on sleeps.

A useful finding states the participants, agreement, change/failure scenario, contextual impact, and proposed repair with its cost. Do not require a fixed report template or a finding for every form.

## Nine forms, with distinctions

| Form | Agreement | Diagnostic question |
|---|---|---|
| [Name](connascence/name.md) | Labels referring to an entity | Which declarations and references must be renamed together? |
| [Type](connascence/type.md) | Types/representations | Do producer and consumer agree on the accepted type? |
| [Meaning](connascence/meaning.md) | Interpretation of values | Is `10` seconds or milliseconds? Does `None` mean absent or failed? |
| [Position](connascence/position.md) | Order of fields or arguments | Could a reordered field silently change behaviour? |
| [Algorithm](connascence/algorithm.md) | Calculation or transformation | Must both ends use the same encoding, checksum, or validation rule? |
| [Execution](connascence/execution.md) | Order of operations | Must initialisation precede use, or locks be acquired in a shared order? |
| [Timing](connascence/timing.md) | When operations execute | Does correctness assume another operation finishes within a time window? |
| [Value](connascence/value.md) | Values changing together | Who preserves an invariant across related values? |
| [Identity](connascence/identity.md) | Reference to the same entity | Would an equal but distinct instance break correctness? |

Position concerns data order; execution concerns operation order. Execution is not automatically timing: “await completion before reading” specifies order, whereas “wait 100 ms and assume completion” relies on timing. Meaning concerns interpretation of values; value concerns coordinated values. Type agreement alone does not express units or currencies. Identity is stronger than equality: two equal objects need not share mutations or a lock.

The site groups name, type, meaning, position, and algorithm as static forms; execution, timing, value, and identity as dynamic forms. Static dependencies are generally easier to inspect than runtime ones. This is not a universal severity ladder: a public name used by hundreds of clients may cost more to change than an execution dependency confined to a function.

## Repair examples and costs

### Meaning: magic role numbers

Before: producers return `2` for administrator and consumers check `role == 2`.

After: both refer to a named `Role.Admin`, or the owning module exposes the relevant permission operation.

The first repair replaces implicit meaning with name/type agreement, but adds a shared declaration. Across independently deployed services, an enum does not remove wire-format compatibility or rollout coordination. Do not create a universal roles framework for one comparison.

For units or currencies, a domain type can reject invalid combinations. A renamed primitive documents intent but cannot enforce compatible arithmetic.

### Position: records and arguments

Before: `user[3]` means administrator; adding a field can silently change the check.

After: `user.is_admin` makes the agreement explicit by name. Named arguments can similarly distinguish same-typed parameters without introducing a new class.

Use a structured type when it represents a coherent concept or owns useful constraints. Do not replace every short positional call: `point(x, y)` may be clear and stable.

### Algorithm: serialisation knowledge

Before: a writer and reader separately know the encoding, framing, and version rules.

After: a codec owns both transformations behind a small interface. Verify with known fixtures and round-trip tests; round trips alone can miss matching bugs on both sides.

If participants are independently deployed, one shared library does not guarantee compatible versions. Keep explicit versioning and compatibility tests. Client and server validation may both be required for different trust boundaries; centralise the contract only where it reduces actual drift without removing necessary validation.

### Execution: initialisation sequences

Before: callers must invoke `setRecipient`, `setSubject`, then `send` correctly.

After: accept complete input in `send(message)`, or construct a valid message before sending. For resources, a scoped operation can own acquisition and release.

Do not introduce elaborate builders or state types for a simple operation. Some protocols genuinely require stages: enforce legal transitions and document the remaining preconditions instead of pretending the sequence vanished.

### Timing: completion assumptions

Before: a consumer sleeps for 100 ms and then reads a producer's output.

After: await an explicit completion signal or synchronise through an operation with a clear completion contract. Timeouts can remain as failure bounds, not proof of completion.

This replaces an implicit timing assumption with explicit coordination; it does not eliminate the dependency. Check cancellation, timeout handling, and failure propagation. Do not introduce a distributed coordinator for a local task.

### Value: coordinated state

Before: callers separately change `items` and a cached `item_count`.

After: derive the count when practical, or let one operation update both atomically.

Derivation may have a runtime cost; atomicity may require a transaction or lock. Establish the actual invariant and consistency requirement before choosing. Values that merely happen to be equal do not establish a coordinated-change requirement.

### Identity: shared resources

Before: participants require the same lock instance, but each can accidentally construct a separate one.

After: one owner creates and supplies the shared lock, or owns the entire protected operation.

Equivalent lock instances do not protect the same critical section. Explicit ownership helps; a global singleton may instead broaden coupling and complicate isolation. Preserve identity where the contract requires it rather than replacing it with value equality.

### Name and type: ordinary interface dependencies

A function and its callers agree on its name and accepted types. Usually retain this coupling: a useful interface requires agreement. A rename may be mechanically safe in one repository but require compatibility handling across external clients. Add adapters or migrations only for a demonstrated compatibility need.

## When not to refactor

- A lock acquisition/release pair is contained within one function with clear ownership. Extracting each stage would spread the agreement.
- A cohesive parser uses positional offsets internally while exposing a simple named result. Do not export internal mechanics merely to make them reusable.
- A small stable positional API is unambiguous. An options object can add more interface than it removes.
- A public contract deliberately specifies a default value. A test should fail if that default changes unexpectedly.

Strong internal agreements can support deep modules. Move or weaken them when their reach or change cost creates a concrete problem, not to reduce a coupling count. Do not invent a numerical connascence score.

## Tests: independent requirements, not duplicated implementation

The site's algorithm example rightly warns about computing an expected fingerprint with the same algorithm as production: both may share the same mistake, and the test may lock in an incidental implementation choice. First establish what the contract promises. A specified hash algorithm deserves independent known-answer tests; an unspecified implementation detail usually does not.

The site's value example replaces an expected initial state with the same `InitialState` constant production uses. This is safe only if the test intends to check internal consistency. If the public contract says new articles start in `Draft`, keep an independent assertion of `Draft`; sharing the implementation's constant could hide a regression.

For a publish test, establish a valid precondition and assert the promised published outcome. Remove unrelated setup assertions only when they are not requirements under test. Never suppress a legitimate failure just to reduce coordinated test edits.

## Source and attribution

Definitions and the three contextual axes come from [connascence.io](https://connascence.io/); see the [archive index and licence](connascence/index.md) and [origins/resources](connascence/pages/about.md). The source credits Meilir Page-Jones's use of the term in 1992 and later work expanding it. External talks and articles are pointers, not additional material analysed by this guide.

Original synthesis and examples here are under the repository's MIT licence. The separately archived Markdown adaptation remains CC BY-SA 3.0; that content is not relicensed as MIT.
