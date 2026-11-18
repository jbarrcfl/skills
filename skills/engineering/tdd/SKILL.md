---
name: tdd
description: Test-driven development elevated with sovereign AST mutation testing. Use when the user wants to build features or fix bugs test-first, mentions "red-green-refactor", wants integration tests, or requires verified test falsifiability.
---

# Test-Driven Development (Elevated with Mutation Testing)

TDD is the **red → green → mutate & kill** loop. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the formal falsifiability rules of the loop. Every section applies on every cycle: consult them before and during the loop, not after.

When exploring the codebase, read `GLOSSARY.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching.

## What a good test is

Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't. A good test reads like a specification: "user can checkout with valid cart" tells you exactly what capability exists, and it survives refactors because it doesn't care about internal structure.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## Seams: where tests go

A **seam** is the public boundary you test at: the interface where you observe behavior without reaching inside. Tests live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam. You can't test everything, so agreeing the seams up front is how testing effort lands on the critical paths and complex logic instead of every edge case.

Ask: "What's the public interface, and which seams should we test?" Give each proposed seam a one-line note on what it catches and what it misses.

When the shape of that interface is itself in question (how deep the module is, where the seam belongs, what the interface should expose), call the Skill tool with "codebase-design" for the vocabulary. It is the shared source of the module, interface, depth, seam, adapter, leverage and locality terms, and it is a reference to consult, not a session to run.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks when you refactor but behavior hasn't changed.
- **Tautological / Hollow**: the assertion recomputes the expected value the way the code does (`expect(add(a, b)).toBe(a + b)`, a snapshot derived by hand the same way, a constant asserted equal to itself), so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth: a known-good literal, a worked example, the spec.
- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify _imagined_ behavior: you test the _shape_ of things rather than user-facing behavior, the tests go insensitive to real changes, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## The Sovereign Falsifiability Standard: Mutation Testing

A test suite that turns green is only half-verified: it proves the code passes the test, but does NOT prove the test fails when the code is broken. Tautological or hollow tests produce false confidence while passing 100% of runs.

To mathematically prove test falsifiability, elevate the TDD loop with **AST Mutation Testing** (`cap-366-ast-mutation-kill-auditor` / `stryker-mutation`):

1. **Synthetic Mutant Injection**:
   After turning green on a slice, inject synthetic boundary relaxations (`>= 0` $\to$ `>= -1`), tautology injections, condition inversions (`==` $\to$ `!=`), and gate bypasses into the implementation/specification.
2. **Execute Mutation Engine**:
   ```bash
   # Run AST mutation testing across capability cell:
   stryker-mutation <target_path>
   # Or natively via cap CLI:
   cap mutate <cell_path>
   ```
   For critical risk, sizing, and decision gates, run the 0B-heap standalone audit:
   ```bash
   /Users/jacob/Developer/af/BarrHome/capabilities/cells/cap-366-ast-mutation-kill-auditor/bin/mutation-kill-auditor --test
   ```
3. **100% Mutation Kill Score Invariant**:
   $$\text{Mutation Kill Score} = \frac{\text{Mutants Killed}}{\text{Total Mutants Injected}} = 1.00$$
   - **Zero Surviving Mutants**: If any mutant survives, the test suite is hollow. You must strengthen the assertions at the seam before moving to the next slice.
   - Delegate mutation audits to the autonomous subagent `mutation-auditor` whenever evaluating complex test suites.

## Rules of the loop

- **Red before green.** Write the failing test first, then only enough code to pass it. Don't anticipate future tests or add speculative features.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Mutate & Kill before refactoring.** Verify that injected boundary mutations and logic inversions are deterministically killed by your tests before declaring the slice complete.
- **Refactoring is not part of the loop.** It belongs to the review stage (see the `code-review` skill), not the red → green implementation cycle.
