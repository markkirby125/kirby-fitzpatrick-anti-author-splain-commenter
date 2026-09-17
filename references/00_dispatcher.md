# Anti Author Splain Commenter — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [The Silent Dialogue Hack Expert Writers Use to Hook You](https://www.youtube.com/watch?v=30Ic686MRZU)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Misdirected Reply

Writing that hooks runs a **silent dialogue**. Reader attention holds because at every beat the writer answers a question the reader has not yet spoken — *what is this?*, *why does it matter?*, *what breaks next?* — one beat before the question turns into a complaint or a bounce. The lecture treats that reply chain as the load-bearing mechanism of attention: not vocabulary, not decoration, a conversation conducted entirely inside the reader's head.

**Author-splaining is that reply chain firing on the wrong side of the dialogue.** The text is still fluent, still grammatical, still short. It is simply *addressed to someone other than the reader*. Three misdirections produce every instance:

1. **The reply that answers the author's own critic** — *"I tried my best to handle this"*, *"sorry, this is hacky"*, *"probably not the right place for it"*. The addressee is an imagined judge. The payload is the author's anxiety about adequacy. The reader learns nothing.
2. **The reply that denies the reader's question exists** — *"obviously"*, *"clearly"*, *"simply"*, *"just"*. The addressee is a reader the author wants to appear competent in front of. A stuck reader now holds two problems: the defect, and a comment asserting the situation is trivial. They blame themselves and stop searching.
3. **The reply addressed to a judging audience instead of a working one** — *"note that this is intentional"*, *"as you can see"*, *"just to be clear"*, *"I know this looks weird"*. The addressee is a hypothetical reviewer whose disapproval is being pre-empted. The working reader wanted the constraint, not the defence.

In software, the misdirection is not a style flaw — it is a **channel-allocation defect**. An annotation occupies the one slot where the next reader will look for the invariant. Worse, an annotated line terminates the reader's search: once a comment exists, people stop asking whether the comment answers anything. Apology and approval-seeking in a comment are therefore *negative*-yield — they consume attention and simultaneously suppress the derivation of the fact they displaced. The failure surfaces months later, when someone edits the guarded line, believes the comment, and ships.

**Sharp boundary against the lexical skills.** Bloat is a cost-per-word problem; author-splain is an *addressee*-correctness problem. `// sorry` is a single lean token that fails this filter outright. `// This path is reachable only when the tenant's schema has not yet been migrated; it does not cover partially migrated tenants.` is 20 words of pure signal and passes every filter. Do not measure this skill in characters.

**Channel model.** Two independent conversations can run through the same comment slot, and only one of them reaches the reader:

```text
[AUTHOR CHANNEL: the mono-dialogue inside the author's head]
   "Did I do enough here?"      → "I tried my best to handle the edge case."
   "Will they think I'm sloppy?"→ "Sorry for the hack, obviously it's not perfect."
   "Am I smart enough for this?"→ "Obviously this is just a simple O(1) path."
   "Was this hard?"             → "After two days of debugging this finally works."
        │
        └── lands in the diff ──▶ reader sees reassurance ──▶ supplies their own guess
                                                ──▶ acts on the guess ──▶ incident

[READER CHANNEL: the same slot, spent correctly]
   "Which input reaches this branch?"  → "tenant.schema_version < 3 only."
   "What breaks if I change the 250?"  → "bounds the caller's 400ms deadline; see budget_test.go:88."
   "Why is the field encoded this way?"→ "upstream rejects '+' in query strings (GH-4412)."
        │
        └── lands in the diff ──▶ reader verifies ──▶ edits the line correctly, or knows why not
```

**Before / after, one annotation, same line count:**

```text
[BEFORE: fluent, lean, and addressed to nobody in particular]       7 lines
  // I tried my best to handle the weird format here. It's a bit
  // hacky but honestly it should probably be fine for now, hopefully
  // no corner cases left. Obviously we just take the last separator.
  // Be careful if you change this.
  -> All five lines carry: effort, apology, hope, unearned authority, unevidenced
     warning. Zero verifiable claims. Reader must re-derive the format rules from
     the regex, then guess whether their case was the "corner case".

[AFTER: same slot, five forensic claims]                           7 lines
  // Three live formats: "1,234.56" (EU), "1.234,56" (DE), "1 234,56" (FR, NBSP).
  // The final separator is the decimal point; every earlier one is grouping.
  // Assumption, unverified: no other locale has been observed. Widen this only
  // after checking dist(amount) in the ledger (query in ISSUE-1043).
  // Raises InvalidOperation on unmatchable input rather than returning 0, because
  // a silent zero already produced one reconciliation incident (ISSUE-1043).
  -> Every claim is checkable by a reader with repo access; the unverified part is
     labelled as unverified and carries its verification route.
```

**Load-bearing vocabulary.**

- **Silent question** — the specific question the reader holds at a specific line (`what breaks if I change this number?`). Every annotation either answers one or wastes the slot.
- **Author-splain** — an annotation, commit body, or review reply addressed to the author's own anxiety or to an imagined judge, carrying no verifiable claim for the working reader.
- **Apologia** — self-justification of effort, difficulty, or intent (*I tried my best*, *after much debugging*, *sorry*, *hacky but*). Zero information content by construction.
- **Unearned authority marker** — a word asserting ease or inevitability without mechanism (*obviously*, *clearly*, *simply*, *just*, *trivially*, *of course*).
- **Hope-hedge** — unlocalized uncertainty transferred to the reader (*hopefully*, *should work*, *probably fine*, *seems to*). A hedge is a debt with no named creditor.
- **Manner-adverb laundering** — the `-ly` class (*carefully*, *properly*, *safely*, *cleanly*, *robustly*, *efficiently*) smuggling an untested *property claim* into the durable record, in the one place where nobody runs a test. `// safely retries` asserts a safety property that no artifact enforces.
- **Forensic comment** — subject is the artifact, not the author; states observable fact + binding constraint + consequence of violation + provenance (measurement, issue, ADR, commit, date) or an explicitly labelled unknown.
- **Diff-adjacent prose** — commit bodies, PR descriptions, and review replies. Same register, same filter, larger blast radius: this prose becomes the historical record and gets quoted as justification years later.

Why the filter binds hardest in software: annotations are read as ground truth by people with production credentials who did not write the line and cannot ask the author. Hedges get promoted into beliefs. Apologies get diffused into the design record, so the archaeology of a decision returns mood instead of cause. Manner adverbs get quoted back as guarantees — *the comment said `safely`* — during a post-mortem. The forensic register is the only one whose claims survive the author.

---

## 2. Core Transformation Protocols

Run every incoming annotation, commit body, PR paragraph, and review reply through **five passes in order**. Order matters: audience triage first, because deletion is the correct outcome for most apologia and no amount of rewriting repairs a comment addressed to the wrong party.

```text
[THE ANTI-AUTHOR-SPLAIN FILTER]
  in ──▶ 1 AUDIENCE TRIAGE ──▶ whose question does this answer?
             │ author's                │ reader's
             ▼                         ▼
        2 APOLOGIA BAN            3 AUTHORITY BAN
        (effort / intent /        (obviously / clearly /
         hope / judgement)         simply / just / -ly)
             │                         │
             ▼                         ▼
        4 EVIDENCE SUBSTITUTION: replace every assertion with mechanism,
          bound, provenance, or an explicit labelled unknown
             │
             ▼
        5 RESIDUE CHECK: does each surviving line carry a verifiable claim?
          No ──▶ delete. (A line with no payload still bills the reader.)
          out ──▶ forensic annotation
```

### Rules

1. **Audience triage before rewriting.** For each annotation, name the silent question it answers. If the question is the author's (*Am I adequate? Will this be judged? Was this hard?*), the annotation is author-splain: rewrite it to the reader's question or delete it. Never leave it standing on the grounds that it is harmless.
2. **Ban the apologia class outright.** *I tried my best*, *sorry*, *my bad*, *apologies for the noise*, *this is probably wrong*, *hacky but it works*, *not sure if this is the right place*, *bear with me*, *I know this is ugly*, *for now*. No rewriting salvages them: delete and state the mechanism or the known limit instead.
3. **Ban the unearned authority class.** *obviously*, *clearly*, *of course*, *simply*, *just*, *trivially*, *everyone knows*, *as expected*, *needless to say*. If something is genuinely obvious, its checkability is the payload — state the mechanism or the test that proves it, and the word becomes unnecessary.
4. **Convert every hope-hedge into a bound or a ticket.** *hopefully*, *should work*, *probably fine*, *seems to*, *fingers crossed*, *this ought to*. A hedge is uncertainty that has not been localized. Either state the condition under which it holds with evidence, or emit `TODO(owner, trigger)`. `Hope` is not an engineering state.
5. **Purge subjective `-ly` manner adverbs.** *carefully*, *properly*, *correctly*, *safely*, *cleanly*, *nicely*, *manually*, *gracefully*, *robustly*, *efficiently*, *quickly*. Each one launders a property claim into an untested location. If the property matters, name the guard or test that enforces it; if it does not, delete the word.
6. **Ban intention narration.** *I wanted to*, *the goal here is*, *intending to*, *this was meant to*, *the idea was*. Intent is a promise; readers need a constraint. State what the code does and which invariant forces it.
7. **Ban effort accounting.** *I spent two days on this*, *after much debugging*, *hard-won*, *painful*, *finally works*. Difficulty is not information. Keep only what the difficulty taught: the trap, the external bug ID, the removed workaround condition.
8. **Ban process residue in commits.** *forgot to add*, *fixing my earlier mistake*, *WIP*, *misc*, *cleanup*, *oops*, *noise*. Rewrite as the behavioural change; if the commit corrects an earlier one, name the invariant that was violated.
9. **Ban reader-directed persuasion and previews.** *note that*, *as you can see*, *remember to*, *don't forget*, *just in case*, *this is intentional*, *trust me*. State the fact so the reviewer can verify it unaided; the working reader needs no guidance to read a diff.
10. **Delete restated code.** A comment paraphrasing the adjacent line is the author reading the diff aloud. The slot belongs to *why*, the invariant, the external constraint, and the cost of removal.
11. **Make the artifact the subject.** *The lock is held across the write*, not *I'm holding the lock here*. First person survives only as genuine accountability (a live incident note with an owner, a dated assumption that someone is on the hook for) — never as justification.
12. **Replace certainty adverbs with provenance.** *definitely*, *certainly*, *verified*, *always* carry no weight. Name the measurement, the version, the issue, the ADR, the commit, or the date: *verified 2026-02-11 against gateway/timeouts.go:14*, *p95 480 → 210 ms, workload bench-recon-100k*. Confidence is earned by the referenced artifact, never by the adverb.
13. **Label the unknown as unknown, with its route out.** *Assumption, unverified: the provider caps pages at 100. Verify with `curl -i …`; tracked in ISSUE-1043.* An unlocalized unknown is a trap; a localized one is engineering.
14. **In review threads, convert defence into disposition.** The moment you notice yourself justifying a decision in prose, stop: acknowledge the point, add the missing evidence to the artifact, resolve the disagreement in code. Reviewer-side symmetry applies too — *I'd prefer*, *nit*, *maybe we could* are author-facing noise; phrase as claim, consequence, and falsifier.
15. **One claim per annotation.** A blended sentence of apology + observation + adverb has no consumer. Each line carries one verifiable statement plus its condition.
16. **Prefer deletion to decoration.** If a line survives all five passes but carries nothing, delete it. Residual politeness still bills attention and still convinces the next reader that the topic was covered.

### 2.1 Banned lexicons by class (with the substitution each one owes)

| Class | Banned lexicon | Forensic substitution |
|---|---|---|
| Apologia | *I tried my best*, *sorry*, *my bad*, *apologies* | Delete; state the mechanism or the known limit |
| Judgement-seeking | *hopefully fine*, *probably ok?*, *thoughts?*, *is this right?* | The condition under which it holds, plus its evidence |
| Unearned authority | *obviously*, *clearly*, *simply*, *just*, *trivially* | The mechanism, or the test that proves it |
| Hope-hedge | *should work*, *seems to*, *ought to*, *fingers crossed* | Bound + evidence, or `TODO(owner, trigger)` |
| Manner adverb | *carefully*, *properly*, *safely*, *cleanly*, *robustly* | The guard, test, or invariant that enforces the property |
| Effort accounting | *spent hours*, *after much debugging*, *painful* | The trap extracted: upstream bug ID, removed workaround |
| Intention narration | *I wanted to*, *the goal is*, *meant to* | The behaviour and the constraint it satisfies |
| Process residue | *forgot*, *fixing my mistake*, *misc*, *WIP* | The behavioural change, and the invariant violated earlier |
| Persuasion | *note that*, *as you can see*, *this is intentional* | The fact, so it can be verified without being argued for |
| Self-reference | *I*, *me*, *my* in durable prose | The artifact or role as subject |

### 2.2 Transformation table: anti-patterns and clean replacements

| Anti-pattern | Failure class | Clean replacement |
|---|---|---|
| `// I tried my best to handle the weird formats here.` | Apologia + unspecified scope | `// Three live formats: "1,234.56", "1.234,56", "1 234,56" (NBSP). The final separator is decimal; earlier ones are grouping.` |
| `// This is hacky but it works.` | Apologia + missing constraint | `// Works around upstream GH-4412 (v2.3–2.5): the loader drops the final record when a batch closes by timeout. Delete once the pinned version is >= 2.6.` |
| `// Obviously we just take the last separator.` | Unearned authority | `// The last separator is decimal by provider contract (docs v4 §Amounts); earlier ones are grouping.` |
| `// Hopefully no corner cases left.` | Hope-hedge | `// Assumption, unverified: no other locale in production. Check dist(amount) in the ledger before widening (query in ISSUE-1043).` |
| `// Should be fine for now.` | Hope-hedge | `// Holds only while `tenant_limit <= 512`. Above that the buffer grows unbounded; see ADR-021 preconditions.` |
| `// We carefully check the cache first.` | Manner-adverb laundering | `// The mutex is held across the miss path only when a fetch is in flight, so concurrent misses collapse to one upstream call.` |
| `// This is simply faster.` | Unearned authority + no provenance | `// p99 480 → 210 ms, workload bench-recon-100k, baseline 4f9c1ab, 5 runs, ±6 ms.` |
| `// I spent two days tracking this down.` | Effort accounting | `// Node time must be read before the socket is drained: after `recv` the timestamp is the drain time, not the arrival time (sampled against tcpdump, 2026-01-08).` |
| `// Just a safety check, probably redundant.` | Authority + hedge, both untestable | `// Reached only when the journal is replayed after a crash mid-append. Removing it makes a torn append silently accepted; covered by recovery_test.go:118.` |
| `// Please don't remove the mutex, it might break things.` | Persuasion + threat without mechanism | `// Removing the mutex reintroduces duplicate upstream calls, which the provider meters and bills per request.` |
| `// Note that this is intentional.` | Persuasion | `// The early return is required: the caller's retry loop treats a partial result as a failure and would re-send the full batch.` |
| `// TODO: clean this up` | Process residue with no owner | `TODO(schema-v3): delete the legacy path once all tenants are migrated; tracked in ISSUE-1043.` |
| `fix my earlier mistake in the migration` | Process residue (commit body) | `fix(db): re-run the backfill for tenants skipped by the 0007 no-op` |
| `sorry for the noise` (commit body) | Apologia | Delete; the commit body states the behavioural change, nothing else |
| `docs: misc cleanup` | Process residue | `docs(api): correct the retry budget in the client README (3 attempts, 1.6s worst case)` |
| `## Summary: I tried my best, this probably isn't the right approach but it seems to work` | Apologia in PR body | `## What changed` / `## Why` (mechanism) / `## Evidence` (command + numbers) — no self-assessment |
| `Let me know if anything looks weird!` | Judgement-seeking | `## Unverified assumptions` (listed, each with how to check it) |
| `I think this should scale, performance looks fine.` | Hedge in an RFC | `## Evidence: 4k rps held p99 < 800 ms single-writer; not valid under multi-writer partition assignment (see Preconditions).` |
| `In my opinion this is clearly the better design.` | Authority in an ADR | `## Decision: flush on min(interval, 512 records). ## Rejected: ticker-only — bounds latency but leaves records unbounded (burst-4k: 12,480 rows buffered).` |

### 2.3 The silent-question ledger

Use this to triage an annotation whose addressee is unclear — match the reader's silent question at that line, then check which answer the text actually supplies.

| Reader's silent question at the line | Required forensic answer | Author-splain that displaces it |
|---|---|---|
| *What breaks if I change this number?* | The dependency the number is bound to, and the test that fails | *"obviously tuned"*, *"picked this carefully"* |
| *When does this branch actually run?* | The trigger condition and its observed frequency | *"just a safety check"*, *"probably never"* |
| *Why is this here at all?* | The upstream defect or constraint, plus its removal trigger | *"hacky but it works"*, *"legacy, sorry"* |
| *Is this tested?* | The test name, or the explicit gap with an owner | *"I tried my best to cover it"* |
| *Who owns this decision?* | Role plus artifact ID (issue, ADR, rotation) | *"per my judgement"*, *"I decided"* |
| *Is this fast enough?* | Baseline, workload, measurement, spread | *"much faster now"*, *"definitely fine"* |
| *What happens on failure?* | Error semantics and the caller's obligations | *"hopefully it won't fail"* |
| *Why such an odd encoding?* | The external constraint that forces it | *"weird, I know, sorry"* |
| *Can I trust the comment?* | Provenance: who measured what, when, on which version | *"verified"*, *"definitely"*, *"obviously correct"* |

**Related dispatchers.** Send any comment whose only job is to answer a reader's factual question into the reader's own artifact with the [Silent Author Test](../../kirby-fitzpatrick-silent-author-test/SKILL.md); strip the remaining filler, Latinate padding, and dead adverbs with the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md) and the [Empty Verb Extractor](../../kirby-fitzpatrick-empty-verb-extractor/SKILL.md); convert defensive review prose into durable dispositions with the [Letter of Response Reviewer](../../kirby-fitzpatrick-letter-of-response-reviewer/SKILL.md); replace your own unearned confidence with checkable observations in reviews with the [Cold Reader PR Auditor](../../kirby-fitzpatrick-cold-reader-pr-auditor/SKILL.md); lead with the decision and follow with the mechanism using [Here's Why Inversion](../../kirby-fitzpatrick-heres-why-inversion/SKILL.md); and refuse to annotate a change whose blast radius is still unmapped with the [Rhetorical Preflight Gate](../../kirby-fitzpatrick-rhetorical-preflight-gate/SKILL.md).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — the annotation slot under audit

Review is where apology and approval-seeking are most expensive, because every reader of the diff is holding a silent question and the slot is finite. Audit inline comments as annotations with an addressee, not as prose to be polished.

Three passes over the diff:

1. **Triage.** For each comment, write the silent question it answers in the margin. No question found, or the question is the author's own → rewrite or delete.
2. **Sweep the three banned classes.** Apologia (*tried my best*, *hacky*, *sorry*), authority markers (*obviously*, *just*, *simply*), and manner adverbs (*carefully*, *properly*, *safely*).
3. **Substitute evidence.** Every surviving assertion must land as mechanism, bound, provenance, or a labelled unknown with its verification route.

```python
# BEFORE — apologetic, hopeful, adverb-laundered; zero verifiable claims
def normalize_amount(raw: str) -> Decimal:
    # I tried my best to handle the weird formats here. It's hacky but
    # hopefully there are no corner cases left. Obviously we just take the
    # last separator. Careful with the comma — this properly converts it.
    cleaned = re.sub(r"[^\d.,-]", "", raw)
    cleaned = cleaned.replace(",", ".")
    return Decimal(cleaned)
```

```python
# AFTER — the same slot spent on the reader's questions
def normalize_amount(raw: str) -> Decimal:
    # Three live provider formats: "1,234.56" (EU), "1.234,56" (DE),
    # "1 234,56" (FR, NBSP). The final separator is the decimal point; every
    # earlier one is digit grouping, which is why they are stripped, not parsed.
    # Assumption, unverified: no other locale has been observed. Verify with
    # dist(amount) over the ledger before widening (query in ISSUE-1043).
    # Raises InvalidOperation on unmatchable input instead of returning 0: a
    # silent zero already caused one reconciliation incident (ISSUE-1043).
    cleaned = re.sub(r"[^\d.,-]", "", raw)
    cleaned = cleaned.replace(",", ".")
    return Decimal(cleaned)
```

The reviewer's version of the same discipline:

```go
// BEFORE — reviewer asserting taste, author defending judgement; two-way author-splain
// R: "Not sure about this, feels wrong to me honestly."
// A: "I tried it your way but it didn't work, so I kept it as is."

// AFTER — reviewer names the silent question; author answers with an artifact
// R: "Is the mutex load-bearing on the miss path? If it only serialises the
//     miss, singleflight would remove the mapping and the contention."
// A: "Load-bearing: without it, concurrent misses fan out to N upstream calls,
//     which the provider bills per request (measured 40 → 40 duplicates on the
//     recon workload). Singleflight would work; tracked as ISSUE-1188 rather
//     than folded in here, because it changes the call semantics."
```

### 3.2 PR Descriptions — removing the apology layer, keeping the evidence

A PR body is read by the person reverting at 03:00 and by the engineer in six months. Self-assessment in it is pure residue: it cannot be verified, it does not describe behaviour, and it trains readers to skim past prose that matters. Strip the author layer and let the claims carry provenance. (For whether the body is *complete*, apply the [Silent Author Test](../../kirby-fitzpatrick-silent-author-test/SKILL.md); this scenario governs only whose questions the body answers.)

```markdown
<!-- BEFORE: author-facing monologue — effort, apology, hedge, appeal, authority -->
## Summary
Sorry this took a while — I tried my best to tidy the batcher. The old code was
obviously pretty bad. I messed around for two days and it seems to work now,
hopefully it's fine. Not perfect but should be a lot faster. Let me know if
anything looks weird!
```

```markdown
<!-- AFTER: each claim carries mechanism, number, or an explicit unknown -->
## What changed
Flush now triggers on `min(interval, 512 records)` in `batcher.go:88` instead of
on the ticker alone.

## Why
A flush trigger *is* the durability contract. A time-only trigger bounds latency
in seconds and leaves record count unbounded, so a burst converts a latency
guarantee into an unbounded buffer. The new bound is a function of the buffer —
the only quantity the writer controls.

## Evidence
`make bench-batcher WORKLOAD=burst-4k RUNS=5` — main @ 9f21c0d vs this head:

| Metric | main @ 9f21c0d | head | Δ |
|---|---|---|---|
| p99 write lag | 3,140 ms | 610 ms | −81% |
| peak rows buffered | 12,480 | 512 | −96% |
| write txn / 10k rows | 42 | 59 | +40% (expected price) |

## Unverified assumptions
- Single-writer partition assignment assumed; not measured under multi-writer
  assignment. Check with the `partition_owner` gauge before merging a second writer.
- Storage cost per transaction was not re-measured; the +40% figure is txn-count
  only.

## Rollback
One commit (`git revert <sha>`); no schema, config, or flag change. The startup
banner `batch_flush_watermark=512` disappears when the revert is live.
```

The review-reply half of the same scenario: an author who notices themselves justifying in the thread stops and converts it into a disposition.

| Defensive reply (author-splain) | Forensic disposition |
|---|---|
| *"I tried that and it didn't work, so I left it."* | *"Tried singleflight at 4a12c0e; it deadlocked the miss path under the write lock. Evidence in the issue. Kept the mutex, tracked as ISSUE-1188."* |
| *"That's intentional, obviously."* | *"The early return is required because the caller's retry loop treats a partial result as failure and would resend the full batch."* |
| *"It's just a small change, I don't think it matters."* | *"Blast radius is the write path of `reconciler` only; readers see smaller batches, never partial ones. Rollback is one commit."* |
| *"Sorry, you're right, I'll fix it."* | *"Confirmed — the cap ignores the caller budget. Pushed 9f21c0d with the deadline check before each sleep; `budget_test.go:88` fails without it."* |

### 3.3 Architecture RFCs / ADRs — author-splain in the durable record

An ADR is the one artifact whose prose gets quoted back as justification for years. Author-splaining here converts into **institutionalized unearned confidence**: *"obviously the right choice"* survives as consensus nobody re-measured, and a hedge (*"we think this should scale"*) becomes a load-bearing assumption that no reader knows to challenge. Every clause must be falsifiable, and each assurance must name the artifact that produced it.

| RFC / ADR section | Author-splain failure | Forensic obligation |
|---|---|---|
| Context | *"the old system was obviously bad"* | The system property at stake, in the system's own terms |
| Decision | *"we'll simply use X"* | One falsifiable commitment a reader can compare against the code |
| Rationale | *"clearly the better design"* | Why this mechanism works, and why the rejected one does not, with numbers |
| Rejected alternatives | Effort accounting about the abandoned path | The measured result that disqualified it, reproducible by the reader |
| Preconditions | Implicit, unstated | Scale band, topology, versions, and the state the decision requires |
| Confidences | *"probably", "in my opinion", "I'm confident"* | Provenance per claim: benchmark, trace, query count, or *unverified assumption* with an owner |
| Reversal trigger | Absent | The signal that proves the decision wrong, its threshold, and the role watching it |
| Does not cover | Absent, letting readers extend the claim | Explicit outer bounds |

```markdown
<!-- BEFORE: authority + hedge + self-reference; unfalsifiable by a silent maintainer -->
# ADR-021 — Write batching
We obviously can't keep flushing on the ticker; in my opinion this is clearly the
right fix and I think it should scale fine. I spent a week on it. Don't touch the
watermark unless you know what you're doing.

<!-- AFTER: same decision, every clause checkable -->
# ADR-021 — Watermark-bounded write batching

## Context
A batcher's flush trigger IS its durability contract: a time-only trigger bounds
latency in seconds and leaves record count unbounded.

## Decision
Flush on `min(interval, 512 records)` in `batcher.go:88`.

## Rejected
Ticker-only: burst-4k buffered 12,480 rows (p99 lag 3,140 ms) versus 512 rows
(610 ms). The 512 cap is not a throughput dial.

## Preconditions
Valid for single-writer partition assignment and datasets <= 10M rows per tenant.
Under multi-writer assignment the watermark bounds one writer's buffer, not the
dataset's in-flight records.

## Reversal trigger
`write_txn_per_row` > 1.5x baseline for one week. Watched by the storage on-call
(rotation `storage-primary`), reviewed quarterly by the platform role.
```

One additional obligation applies to review commentary on these documents: adjudicating a design by adjective (*"cleaner"*, *"simpler"*, *"more robust"*) is author-splaining at architecture scale. Name the property, the workload that exposes it, and the threshold that decides — otherwise the discussion is about whose taste wins, and the decision record will not say which.

---

## 4. Verification Checklist

- [ ] **Addressee triage recorded.** Every annotation in the diff answers a named silent question the reader holds at that line, and no annotation survives whose only addressee is the author's anxiety or an imagined judge — zero apologies (*tried my best*, *sorry*, *hacky but*, *my bad*), zero judgement-seeking (*thoughts?*, *is this right?*).
- [ ] **Zero unearned authority, zero hope-hedges.** No *obviously / clearly / simply / just / trivially / of course*; every *should / probably / seems / hopefully* either states the condition under which it holds with evidence, or is an explicit `TODO(owner, trigger)`. *(Note: adversarial pass — a re-run of the grep over the final diff, not the draft.)*
- [ ] **Zero manner-adverb laundering.** No *carefully / properly / safely / correctly / cleanly / robustly / efficiently / nicely*; each property those words claimed is now asserted by a named guard, test, or invariant — or deleted with the word.
- [ ] **Every surviving claim has provenance or a labelled unknown.** Each line resolves to a measurement, issue, ADR, commit, or date; anything unmeasured appears literally as *Assumption, unverified: … + how to verify*, and no claim rests on an adverb for its confidence.
- [ ] **Diff-adjacent prose is forensic.** Commit bodies, PR text, review replies, and any RFC/ADR touched by the change contain no effort accounting, no process residue (*forgot*, *misc*, *cleanup*), and no reader-directed persuasion (*note that*, *as you can see*, *this is intentional*); each paragraph states artifact, constraint, evidence, and consequence.