---
name: domain-modeling
description: Build and sharpen a project's domain model elevated with sovereign .allium and .un specifications, categorical entity extraction, and non-trivial domain invariants. Use when discussing codebase terminology, formalizing entities and lifecycles, writing or editing GLOSSARY.md / behavior specs, or recording ADRs.
---

# Domain Modeling (Elevated with Sovereign Invariants)

Actively build and sharpen the project's domain model as you design. This is the *active* discipline: challenging terms, inventing edge-case scenarios, extracting categorical entities, authoring formal `.allium` / `.un` specifications, and writing the glossary and decisions down the moment they crystallise. (Merely *reading* `GLOSSARY.md` for vocabulary is not this skill: that's a one-line habit any skill can do. This skill is for when you're changing the model, not just consuming it.)

## File structure

Most repos have a single context:

```
/
├── GLOSSARY.md
├── behavior.allium                   ← sovereign entity & lifecycle spec
├── behavior.un                       ← unified notation declarative contract
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

If a `GLOSSARY-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives:

```
/
├── GLOSSARY-MAP.md
├── docs/
│   └── adr/                          ← system-wide decisions
├── src/
│   ├── ordering/
│   │   ├── GLOSSARY.md
│   │   ├── behavior.allium           ← context-specific behavioral spec
│   │   ├── behavior.un               ← context contract & invariants
│   │   └── docs/adr/                 ← context-specific decisions
│   └── billing/
│       ├── GLOSSARY.md
│       ├── behavior.allium
│       ├── behavior.un
│       └── docs/adr/
```

Create files lazily: only when you have something to write. If no `GLOSSARY.md` exists, create one when the first term is resolved. If formal invariants or state transitions emerge, author `behavior.allium` and `behavior.un`. If no `docs/adr/` exists, create it when the first ADR is needed.

## During the session

### Challenge against the glossary

When the user uses a term that conflicts with the existing language in `GLOSSARY.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account': do you mean the Customer or the User? Those are different things."

### Categorical Entity & Morphism Extraction

Map unstructured domain concepts into formal categorical structures:
1. **Objects (Entities & Value Types)**: Entities with immutable identities vs value types defined by structural equivalence.
2. **Morphisms (State Transitions & Actions)**: Operations mapping entity state $S_i \xrightarrow{f} S_{i+1}$. Ensure compositionality: $g \circ f$ preserves structural invariants.
3. **Algebraic Invariants**: Define conservation laws, capacity bounds, and balance equations that must hold over all morphisms.

### Formal Sovereign Behavioral Specification (`.allium`)

Translate crystallised domain entities and state lifecycles into a native `.allium` specification:

```allium
module barrhome.domain.<context>;

entity <EntityName> {
    id: string;
    capacity_units: int;
    utilized_units: int;
    status: string;
}

behavior <LifecycleName> {
    initial state Proposed;
    state PreconditionsVerified;
    state Active;
    state Settled;

    transition verify: Proposed -> PreconditionsVerified;
    transition activate: PreconditionsVerified -> Active;
    transition settle: Active -> Settled;
    transition reset: Settled -> Proposed;

    action propose_entity();
    action verify_system_invariants();
    action execute_transition();
    action settle_audit_log();
}

invariant CapacityNotExceeded {
    entity.utilized_units <= entity.capacity_units && entity.utilized_units >= 0;
}
```

### Declarative Contracts & Topology (`.un`)

Express domain boundaries, service interfaces, transports, and schema contracts in Unified Notation (`.un`). Validate envelopes and relations:

```bash
un validate behavior.un
un links behavior.un
```

Enforce contract-level preconditions and postconditions:
```un
capability <ContextName>Capability {
    domain: applied
    version: "1.0.0"

    structure <EntityName> {
        id: String
        utilized_units: Int
        capacity_units: Int
    }

    behavior <LifecycleName> {
        initial state Proposed
        state Active
        state Settled

        transition activate: Proposed -> Active
        transition settle: Active -> Settled

        invariant CapacityBounded {
            forbids: incoming.utilized_units > incoming.capacity_units || incoming.utilized_units < 0
            on_violation: reject(incoming)
        }
    }
}
```

### Discrete State Transition Verification (`model-checker`)

Before implementing code from the domain model, verify the state transition graph using the sovereign model checker:
```bash
~/Developer/af/BarrHome/capabilities/cells/component-state-machine/bin/model-checker
```
Verifies:
- **Complete Reachability**: $\text{Reach}(S_0) = S$ (no orphaned, unreachable states).
- **Deadlock Freedom**: No absorbing sink states prior to designated terminal settlement states.

### Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### Cross-reference with code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible. Which is right?"

### Update GLOSSARY.md inline

When a term is resolved, update `GLOSSARY.md` right there. Don't batch these up: capture them as they happen. Use the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).

`GLOSSARY.md` should be totally devoid of implementation details. Do not treat `GLOSSARY.md` as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

### Offer ADRs sparingly

Only offer to create an ADR when all three are true:

1. **Hard to reverse**: the cost of changing your mind later is meaningful
2. **Surprising without context**: a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off**: there were genuine alternatives and you picked one for specific reasons

If any of the three is missing, skip the ADR. Use the format in [ADR-FORMAT.md](./ADR-FORMAT.md).
