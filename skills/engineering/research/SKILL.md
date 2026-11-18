---
name: research
description: Investigate a question against high-trust primary sources, sovereign capability cells, and indexed codebases. Elevated with holo find, zoekt trigram search across 2,548 capability cells, and zero-token evidence retrieval. Use when the user wants a topic researched, docs or API facts gathered, or reading legwork delegated to a background agent.
---

# Research (Elevated with Holographic Discovery & Zoekt Trigram Search)

Spin up a **background agent** to do the research, so you keep working while it reads. Start it with a fresh context window.

Its job:

1. Investigate the question against **primary sources** (official docs, source code, specs, first-party APIs, sovereign capability cells), not a secondary write-up of them. Follow every claim back to the source that owns it.
2. Write the findings to a single Markdown file, citing each claim's source with exact file links and line ranges.
3. Save it where the repo already keeps such notes; match the existing convention, and if there is none, put it in `docs/research/` and say where.

---

## Sovereign Discovery Engines

### 1. Minkowski Hyperbolic Capability Discovery (`holo find`)
Before investigating external sources or inventing new abstractions, query the 2,548 sovereign capability cells via hardware-accelerated $\mathbb{H}^{16}$ Minkowski SIMD search:
```bash
# Instant discovery across capability cells (<30ns):
holo find "<concept or acronym>" --cap

# Exact symbol discovery across capability implementations:
holo find "<symbol_name>" --code
```
Inspect the discovered capability cell's `manifest.cue`, `behavior.allium`, and `behavior.un` contracts. Reusing established sovereign capabilities guarantees architectural coherence and avoids redundant code generation.

### 2. Distributed Trigram Code Search (`zoekt-search`)
Query across all repositories hosted on Gitea using the trigram-indexed REST search engine:
```bash
# Search all indexed repositories on Gitea:
curl -s "https://zoekt.m.barrhome.org/search?q=<query>&format=json" \
  | jq '.result.FileMatches[]? | {repo: .Repo, file: .FileName, url: .URL}'
```
Useful search qualifiers:
- `repo:<regex>`: Scope search to specific repos (e.g. `repo:BarrHome/capabilities`).
- `file:<regex>`: Filter by file extension or path (e.g. `file:\.zig$`, `file:SKILL\.md`).
- `sym:<name>`: Direct symbol definition matching.

### 3. Zero-Token Evidence Retrieval
Avoid polluting agent context windows with massive source files. Retrieve only the exact bounded evidence:
```bash
# Slice exact AST symbol subtree without loading full file:
holo slice --symbol <function_or_struct> <file_path>

# Extract token-bounded line range:
holo slice --range 120:185 <file_path>

# Generate hardware-accelerated cryptographic Merkle digest for provenance:
holo digest <file_path>
```

---

## Epistemic Grounding Standard

Classify every factual claim and research assertion into one of the four formal categories:
- **Grounded**: Verified directly against source AST, test suite execution, or formal proof bridge.
- **Attested**: Quoted verbatim from official first-party documentation, RFC, or signed specification.
- **Inferred**: Derived logically from grounded premises. State the deductive premise and inference step explicitly.
- **Falsified**: Directly refuted by source code, execution failure, or counterexample.

Never present an inference or guess as an attested fact. If an API behavior is unverified, flag it as `[UNVERIFIED]` and provide the exact probe command to test it.

