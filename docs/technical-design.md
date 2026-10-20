# Tome: a formal specification and executable engine for Magic

Tome is an experiment in formalizing Magic: The Gathering using Lean 4. Its
central question is whether a faithful translation of the Comprehensive Rules
and card text can produce correct interactions without programming those
interactions individually. The useful outcomes include a working engine, precise
accounts of ambiguous rules, and evidence about where formalization becomes
difficult.

The proposal is specification first: describe permitted game behavior in Lean,
then prove that an executable engine follows that description. Develop connected
executable slices, starting with two players and a small card corpus. Evaluate
unseen combinations, structured rules questions, and repeated automated play.
This document specifies the architecture; it does not claim that it exists.

## Related information

These sources establish the domain and proof boundaries. Versioned inputs must
be recorded during implementation; live documentation links are background.

- [Official rules page][rules]: publication point for the Comprehensive Rules.
- [Rules snapshot, September 25, 2026][cr]: concrete reference for this design's
  rule numbers, not a promise to follow future updates automatically.
- [Gatherer][gatherer]: official card reference for reviewing Oracle text.
- [Lean elaboration and compilation][compilation]: how logical definitions,
  proofs, and executable programs relate.
- [Lean proof validation][validation]: checking theorem statements and their
  dependencies, including assumptions hidden in imported proofs.
- [Lean recursive definitions][recursion]: options for expressing terminating
  computations and reasoning about computations that may not terminate.
- [r/mtgrules][questions]: candidate questions, not an authoritative answer key.
- [Copy of a Copy of a Copy of a...][copy-question]: an example of a question
  combining several cards and requiring a precise account of copied properties.

[rules]: https://magic.wizards.com/en/rules
[cr]: https://media.wizards.com/2026/downloads/MagicCompRules%2020260925.txt
[gatherer]: https://gatherer.wizards.com/
[compilation]: https://lean-lang.org/doc/reference/latest/Elaboration-and-Compilation/
[validation]: https://lean-lang.org/doc/reference/latest/ValidatingProofs/
[recursion]: https://lean-lang.org/doc/reference/latest/Definitions/Recursive-Definitions/
[questions]: https://www.reddit.com/r/mtgrules/
[copy-question]: https://www.reddit.com/r/mtgrules/comments/1wtgmw0/copy_of_a_copy_of_a_copy_of_a/

## What would count as success?

Tome should predict interactions because the rules compose. Reproducing examples
used to write the implementation is necessary but insufficient evidence.

- A newly encoded card interacts with existing cards without matchup patches.
- Held-out scenarios produce independently reviewed answers and explanations.
- Each answer identifies its source versions, assumptions, and coverage limits.
- Automated games exercise the same transition engine used for rules questions.
- Unsupported mechanics and unfinished searches remain visible as limitations.
- Formalization gaps produce small, inspectable examples rather than guesses.

Measure coverage, correctness, and generalization separately. A system answering
nine of ten supported questions correctly has a correctness problem, even if it
supports only ten questions. A system answering all ten correctly still has a
coverage problem if the benchmark contains a thousand questions.

The initial interface accepts structured scenarios written by a human. Natural
language understanding, automated Oracle translation, strategic strength, and a
consumer game interface are outside this document's initial implementation
boundary. The long-term rules coverage goal remains comprehensive, including
multiplayer; early claims apply only to the supported two-player subset.

## What the proofs mean

A **formal specification** is a mathematical description of allowed behavior.
For Tome it describes game states, legal decisions, and state transitions. A
**proof** is evidence checked by Lean that a stated mathematical claim follows
from the definitions and assumptions it uses. It does not validate the English
meaning intended by the author; see [Lean's validation guidance][validation].

There are three distinct obligations:

- Translation fidelity: the specification expresses the intended Magic rules.
  Establish evidence through source review and independent scenarios.
- Engine correctness: the executable transition procedure implements the formal
  specification. Establish this relationship with Lean theorems.
- Execution trust: the compiled program actually executes those definitions. The
  compiler, runtime, and machine remain part of the practical trust boundary.

**Soundness** means every successful engine transition is permitted by the
specification. **Completeness** means every permitted transition in a stated
supported domain can be represented and obtained by the engine. Soundness alone
would allow an engine that rejects every action.

These schematic statements illustrate the intended relationship, not runnable
Lean declarations. Here `denote` maps executable to abstract states, and `D`
requires a well-formed state within a supported domain. `AllowedRun` describes a
finite sequence between public decision or game-end boundaries; `inputs` records
all player decisions and random outcomes supplied along that sequence.

```text
soundness:
  D(s) and run(s, inputs, fuel) = Success(u)
    -> AllowedRun(denote(s), inputs, denote(u))
completeness within D:
  D(s) and AllowedRun(denote(s), choices, t)
    -> exists inputs, fuel, u,
       run(s, inputs, fuel) = Success(u) and denote(u) = t
```

Completeness also requires the encoded inputs to represent the specified
choices, not merely reach the same state by another route. Its existential fuel
means a sufficient finite budget exists; it does not promise success under every
limit. Inputs may be supplied incrementally as the engine pauses for decisions.

A **refinement** is an implementation whose observable behavior respects a more
abstract specification. The theorem relates public calls to internal sequences,
including pauses. Exhaustion may return a valid partial execution, but must not
masquerade as completion of the requested sequence.

An **axiom** is an assumption used without proof. Use definitions and relations
to express game rules, not axioms that assert desired outcomes. A relation
describes which transitions are allowed; it can admit several successors without
making Lean's logic inconsistent. Prove useful invariants, such as valid
references and well-formed pending work, and show that the admitted starting
states are nonempty.

Inspect assumptions used by accepted theorems, including indirectly through
imported proofs. Reject unfinished proofs or project-specific axioms asserting
engine behavior; record any standard logical axioms used. Interpretive
assumptions belong in explicit theorem hypotheses or versioned definitions,
never hidden proof gaps.

**Native evaluation** uses compiled computation in proof checking; its trust
implications depend on the Lean version and mechanism. Do not silently broaden
the advertised proof boundary to trust that execution. These checks follow the
distinctions in [Lean proof validation][validation].

## Sources, coverage, and interpretation

Line-by-line translation means clause-level accountability, not one function per
paragraph. Shared definitions may formalize several clauses, while one clause
may impose multiple obligations. The mapping must work in both directions.

A **ruleset** is an immutable combination of rules text, card-text snapshots,
formal definitions, and recorded interpretation decisions. Identify it by source
versions and content hashes. Pin Lean and dependency versions with each build.
Do not label an answer merely as using the latest rules.

For every source clause, record its formal targets, dependencies, and status.
Useful statuses distinguish untranslated, specified, executable, and proved
obligations; a single supported flag would conceal too much information. A
clause may contain both covered and uncovered conditions or exceptions.

For example, the traceability record for object identity might look like this:

```text
source: rules snapshot / clause 400.7
formal targets: zone transition relation; object identity allocation
exceptions: separately mapped to their source subclauses
execution: supported only when all applicable exceptions are modeled
evidence: transition theorem; leave-and-return scenario
```

A **coverage profile** declares the admitted rules, card definitions, and
scenario conditions for a particular executable subset. It must include their
semantic dependencies. If a supported card can create an unsupported effect,
that card is not unconditionally supported. Check profiles before execution and
guard dynamic boundaries; unknown behavior must never become a no-op or an
illegal action.

An **interpretation ledger** records a passage, plausible readings, the selected
reading, evidence, and a distinguishing scenario. An unresolved choice makes
answers conditional on that reading. Official rulings can reveal a missing
premise, but importing that premise must be explicit and separately identified.

A failed proof is not a discovered contradiction. First classify the failure:

- Translation error: the formal statement omitted or changed a condition.
- Missing coverage: a required clause or card behavior has no formal meaning
  yet.
- Legitimate choice: multiple outcomes follow from different legal decisions.
- Interpretation gap: the source leaves a relevant distinction unresolved.
- Candidate specification defect: obligations conflict under stated premises.

For the last case, preserve the smallest witness, source clauses, and
assumptions. Check that the state is reachable, or label reachability as
unestablished. Failure to find a successor is only evidence after checking
allowed decisions, coverage, and legitimate stopping conditions. No generic
consistency detector is promised: the research contribution is precise claims
and inspectable evidence.

## State and semantic boundaries

Separate the logical game model, executable operations, and external tools. Keep
parsing, reporting, random sampling, and policy code outside the rules core.
Start with closely related logical and executable representations; introduce
more elaborate refinement mappings only when measurement justifies them.

The state model needs explicit representations for:

- Players, ownership, control, turn position, priority, and consecutive passes.
- Objects, zones, ordered collections where relevant, and object identities.
- Printed or copied characteristics, counters, damage, and derived properties.
- Stack entries, announced choices, targets, costs, and resolution progress.
- Pending events, triggered abilities, continuous effects, and durations.
- Information retained for rules that inspect earlier object states.
- A suspended computation when a player must make a decision.

Use typed identifiers so an object reference cannot accidentally name a player
or a card definition. Keep card identity distinct from game-object identity. A
card leaving and returning generally represents a new object; rule 400.7 and its
exceptions motivate this distinction. [Source: Comprehensive Rules][cr].

Use immutable logical states and explicit well-formedness predicates. Prefer
ordinary data structures plus boundary proofs over placing a large proof inside
every runtime record. Validate imported scenarios before admitting them to the
engine. Record whether they are merely well formed or accompanied by evidence of
reachability from game setup; these are different guarantees.

An illustrative state distinction is:

```text
card identity: C17
old battlefield object: O81
returned battlefield object: O94
old target reference: O81
```

Do not update old references automatically to O94. Model any authorized
cross-zone tracking as a specific rule, including the information it preserves.
Historical information should be retained because a semantic dependency needs
it, rather than by copying the entire game history into every state.

The engine may know hidden information to simulate a concrete game. Player
policies receive only their permitted observation. Scenario authors must supply
missing facts or quantify over them explicitly; the engine must not invent a
library order or assume that an unknown hand is empty.

## Transitions, choices, and events

Use **small-step semantics**, meaning a relation describing individual advances
in a computation, so complicated actions can pause and resume without collapsing
all rules into one enormous function. A state includes the current operation.
External callers see meaningful decision boundaries rather than every microstep.

Keep the authoritative relations explicit: legal decisions, event generation,
effect interpretation, and scheduler transitions. Computational predicates may
be shared with the engine, but defining legality as whatever the engine accepts
would make the main correctness claim circular.

An engine operation has the following conceptual result family:

```text
advance(state, suppliedInput?, budget) ->
  Ready(state, inputRequest) | Finished(state, gameResult)
  | InvalidDecision(reason) | Unsupported(reason)
  | Exhausted(resumableState) | InternalFailure(diagnostic)
```

A decision request states who chooses, what is being chosen, and the
constraints. Validate submitted decisions before applying them. For large or
unbounded choice domains, describe constraints rather than enumerating every
option. Rejection must preserve the last accepted state; internal failures must
stop the run.

Distinguish choices of modes, targets, payments, orderings, and optional
effects. Deterministic replay records the actual accepted decisions. A
simulation policy selects decisions; it cannot change legality. Random outcomes
are explicit inputs with admissibility checks; a seeded sampler is outside the
semantic core. Do not infer uniformity or fairness from a legal-outcome check
alone. The input channel distinguishes player decisions from sampled outcomes; a
request names which kind is required and who or what supplies it.

Events are structured descriptions of changes, not just log messages. Preserve
simultaneous batches and the before/after information needed by rule predicates.
Replacement and prevention processing must transform proposed events before
commitment. Trigger detection observes the appropriate event context and stores
pending abilities separately from their later placement on the stack.

The scheduler owns priority boundaries and automatic processing. For example,
rule 704.3 requires repeated state-based-action checks and pending-trigger
processing before priority is granted. This cannot be implemented as one cleanup
pass after each card callback. [Source: Comprehensive Rules][cr].

Represent casting, paying costs, and resolving as distinct operations with their
own decision points. A generic pass action must not skip automatic work or
invent a priority window during a suspended resolution. Test scheduler behavior
through the public engine boundary, not only isolated effect functions.

## Cards and persistent effects

A card definition combines versioned characteristics with a typed description of
its abilities. A shared **effect language** describes card instructions as data,
with formal meaning supplied by the rules specification and its interpreter. It
is an architectural interface, not a claim that a tiny language fits all cards.

Provide constructs for sequencing, conditions, object selection, player choices,
events, and creation of persistent effects. Separate target restrictions and
announcement choices from instructions executed during resolution. Unusual cards
may require new reusable constructs and associated proofs, never matchup checks.

This illustrative encoding deliberately leaves targeting to the spell framework:

```text
spell declaration:
  target: one creature, checked by the shared targeting rules
effect body:
  request Damage(source = thisSpell, recipient = target, amount = 2)
```

Damage goes through shared event semantics; it is not implemented by directly
subtracting toughness. Card identifiers select definitions and provenance only.
A branch of the form "if card A and card B are present" is unacceptable as an
interaction fix. A characteristic explicitly referenced by card text is
different.

Persistent effects require separate semantic machinery. Store their source,
applicability, duration, ordering information, and any captured choices. Derive
current characteristics through a dedicated evaluator rather than repeatedly
overwriting printed characteristics. Treat copy information as distinct data.

A **continuous effect** modifies characteristics or rules over its duration. A
**replacement effect** changes an event before it happens. A **triggered
ability** reacts to a matching occurrence and creates work to process
subsequently. These need distinct machinery, not one ordered callback list.
Detailed ordering and applicability rules need appendices; these boundaries are
requirements now.

Before declaring a subset supported, include the mechanisms it depends on. An
initial corpus can exclude complicated effects explicitly; the implementation
cannot pretend such effects are absent from an admitted scenario. Include an
early stress case combining a persistent effect with ordinary resolution to test
whether the abstractions compose beyond simple spell sequences.

## Rules questions and explanations

A structured question supplies a ruleset, coverage profile, initial state,
explicit assumptions, a predicate to ask about, and a stopping condition. State
whether it asks about one decision sequence, some legal sequence, or all legal
sequences within a specified domain. These are different mathematical questions
and must remain different result types.

For example, a targeted spell can be tested against an object leaving and
returning before resolution. The spell still targets O81; the returned object
O94 does not become that target. If all targets are illegal, the spell does not
resolve under rule 608.2b. [Source: Comprehensive Rules][cr].

A question could encode that situation as follows:

```text
given: S on the stack, with exactly one target, creature O81
response: an instant B that exiles its target, then immediately returns it
choices: legally cast B targeting O81; both players pass to resolve B
continue: both players pass to process S; stop before the next decision
ask: did S resolve?
assumptions: costs payable; no other effects; returned object is O94
```

For a fixed sequence, return the outcome and semantic trace, or an explicit
unsupported or unresolved result. For an existential question, one valid witness
can establish possibility. A negative existential answer or positive universal
answer requires complete exploration of the stated domain, or a separate
theorem. A successful sampled simulation proves neither.

Explanations are generated from structured derivations with rule and card-text
references. They state choices and assumptions before conclusions, identify
relevant objects, and show the causal steps. A readable trace is not itself a
Lean proof; label checked theorems separately from ordinary execution output.

Preserve the source question and a reviewed expected answer independently from
the formal scenario. The linked copy question illustrates how a short post can
combine several semantic mechanisms. It is a development example, not held-out
evidence, and its comments do not establish the correct answer automatically.

## Termination and bounded exploration

Lean's treatment of recursive definitions requires an explicit approach to
potentially unbounded computation; see [recursive definitions][recursion]. Tome
must not assume that every game, interaction, or search terminates.

Use finite individual operations and a driver with an explicit step budget. Here
**fuel** means a counter consumed by the driver to guarantee it returns. If an
internal computation can also grow without bound, budget that work too.
Exhaustion preserves resumable state and remains an operational limitation.

Separate three cases in reports:

- The modeled rules establish a game outcome, including a draw when justified.
- A chosen policy keeps playing or produces a repeated state.
- The computation ran out of time, memory, or fuel before establishing an
  answer.

Repeated state hashes are diagnostic hints until equivalence is justified. A
cycle in a simulation is not automatically a mandatory rules loop. Loop
shortcuts require separate semantics and proof obligations. Do not turn a search
limit or a policy's repeated choice into a game-result rule.

Bounded exploration records depth, choices considered, omitted domains, and
unresolved frontiers. State merging must preserve the question's relevant future
behavior; matching battlefield layouts alone is insufficient. Initially prefer
transparent exploration over aggressive pruning with unclear correctness.

## Performance and repeated simulation

The runtime must support many games or action sequences under simple policies,
as well as individual questions. Compilation and proof checking occur separately
from ordinary runs; reuse the transition correctness theorem across executions.
The compiled execution trust boundary remains explicit, as described in [Lean's
compilation documentation][compilation].

Use compact runtime identifiers, indexed object lookup, and incremental derived
state only when profiling supports them. Every cache needs a specified invariant
and invalidation rule. A faster representation requires a refinement argument;
changing storage must not redefine the abstract game state or legal behavior.

Run simulations without full explanatory traces by default. Retain ruleset,
seed, accepted decisions, stopping reason, and failure context for replay.
Enable detailed explanations when reproducing a failure. Trace mode must not
change semantic outcomes, including ordering of simultaneous work.

Measure at least these workloads independently:

- Fixed-scenario latency after compilation, targeting seconds for ordinary
  cases.
- Actions per second for replay and policy-driven execution, reported
  separately.
- Whole games per second, with deck corpus, policy, and stopping limits
  recorded.
- Scaling with object count, active effects, triggers, and choice-domain size.
- Proof build time, peak memory, and explanation overhead.

Record hardware, toolchain, corpus, and distributions rather than a single best
number. Establish a numeric throughput baseline in the first executable slice;
there is insufficient evidence now for a defensible games-per-second promise.
Use replay to distinguish engine cost from policy cost and legal-choice search.

## Validation and research milestones

Maintain separate evidence for translation, proofs, and runtime behavior. Tests
can expose a wrong formalization even when its implementation proof is correct.
Accepted theorems can rule out engine bugs that examples would never exercise.

The first assembled check should cross scenario loading, validation, card
interpretation, scheduling, execution, and explanation. Use the targeted-spell
leave-and-return scenario above; check its source mapping and final result. This
checks a real integration boundary before growing the card catalog.

The executable-slice strategy uses these evidence gates:

- Establish a narrow two-player slice with casting, priority, resolution,
  supported automatic actions, and a runnable structured question.
- Add interaction stress cases involving triggers, persistent effects, and event
  replacement; extend shared semantics when those cases expose a gap.
- Run complete games for the admitted corpus, measure throughput, and publish
  held-out interaction results with coverage and proof obligations attached.
- Expand coverage only while maintaining reproducibility and explicit limits.

Hold out interactions after defining their constituent rules and cards. Do not
call a case unseen after using its answer to tune the implementation. Once a
failure is diagnosed, move it into regression tests and select new held-out
cases. Independent reviewers should derive expected answers from the pinned
sources.

Automated validation includes invariants, illegal-decision rejection, scenario
import failures, deterministic replay, and identical results with tracing off
and on. Compare optimized and simple execution paths against common semantic
claims. Use generated legal histories, plus separately labeled arbitrary states,
to avoid mistaking impossible game states for rules defects.

Always report unsupported cases and unresolved searches alongside successes. A
disagreement should retain a minimal scenario, decision log, source versions,
proof coverage, and expected-answer provenance. Investigate the translation,
engine, harness, and expected answer independently before assigning blame.

## Source evolution and further design topics

A rules update creates a new ruleset. Diff changed clauses and card texts,
follow traceability dependencies, and revalidate affected definitions and
examples. Keep old scenario results attached to their old ruleset; never rewrite
history to imply that an old answer was derived under a newer interpretation.

Scenario imports declare a schema version and ruleset identity. Reject unknown
versions or mismatched definitions. Any future migration must be explicit and
preserve the original scenario; it cannot silently reinterpret object
references, card meanings, or previous interpretation assumptions.

Appendices should settle the following detailed topics without expanding this
high-level document into a complete formal specification:

- State identity, zone changes, historical information, and reachability.
- Card-language expressiveness, translation review, and targeting and costs.
- Priority, simultaneous events, triggers, and scheduler correctness.
- Continuous effects, layers, dependencies, copying, and derived-state caches.
- Replacement and prevention effects, applicability, and player decisions.
- Loops, shortcuts, finite search, and proofs of termination where available.
- Lean proof organization, trust audits, and executable refinement techniques.
- Corpus construction, held-out evaluation, and reproducible performance tests.
- Multiplayer, Commander, hidden information, and future natural-language input.

## Manual QA

These checks apply to the eventual engine. They are not evidence that this
architecture document has implemented or proved any Magic behavior.

- Load the targeted-spell scenario through the public scenario interface.
  Inspect its assumptions, identities, accepted decisions, outcome, and rule
  references.
- Remove the leave-and-return action and confirm that the explanation changes
  for the semantic reason, without changing either card definition.
- Supply an illegal decision and verify that the accepted state is unchanged.
  Load an unsupported mechanic and verify that it produces a coverage report.
- Stop a search with a tiny budget. Confirm that the result is unresolved, then
  resume it and compare with an uninterrupted run using the same decisions.
- Replay a seeded simulation with tracing enabled. Confirm identical semantic
  results and a readable explanation of the first interesting interaction.
- Inspect one accepted theorem's assumptions and trace it back to source
  clauses. Compare its actual guarantee with the wording shown in the answer
  report.
- Change the requested ruleset or scenario version. Confirm that incompatible
  inputs are rejected explicitly and previous results retain their provenance.
