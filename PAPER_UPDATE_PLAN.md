# Paper update plan — code state 2026-08-10 vs paper HEAD 06d28c9 (2026-08-06)

Analysis of what the manuscript must change to match the code, what should be cut to the repo, and
what the figures need. Written 2026-08-10.

## STATUS (2026-08-10)

**Applied, uncommitted** — everything that does not need results:

- §A1–A6 all applied. Stale-claim sweep is clean (`cross-placement`, `identity-at-initialisation`,
  `non-negative evidence`, `calibrated rejection`, per-window caps, collapse regularisers, the
  R0/R1/R2 rows, `18,400 paired events`, the batch reservation — all gone, including one straggler
  in `2_related_work.tex`).
- §B1 sealed roster and §B4 scope limits added to the protocol; prototype/ridge rows and the two
  enrollment canaries added to `tab:label_efficiency`; kill criterion now names the closed-form
  rivals; `tab:crossconfig` caption says its rows are training-corpus streams.
- New design material: coreference slots, and the relative attention bias as the sole retriever
  gradient path.
- `app:phaseb` rewritten against `policy.py`.
- Figures: `fig_overview` (objectives, rejection, trainable Phase-B parts) and `fig_evidence`
  (full redraw — the old one was drawn for the deleted decoder).
- Verified: builds clean, 0 undefined refs/citations, **body still 12 pages** (refs start p.13,
  same as baseline HEAD); the one >5pt overfull hbox is pre-existing, confirmed against a pristine
  HEAD build. The extra total page is appendix, which does not count against the CFP limit.

**Deferred — needs results or a measurement:**

- §D `fig_geometry` measured rewrite (needs the all-pairs text-vs-sensor-centroid scatter run).
- §D new enrollment-curve figure (needs the curves).
- §E the framing decision.

Sources checked: `5_design.tex`, `6_experiments.tex`, `8_appendix.tex`, `1_introduction.tex`,
`3_motivation.tex`, `figures/fig_overview.tex`, `figures/fig_evidence.tex` against
`docs/design/PHASE_B_TRAINING_INTENT.md`, `docs/design/PHASE_A_B_AGREED_IMPLEMENTATION_PLAN.md`,
`docs/results/PHASE_B_STEP0_CONTROL.md`, `training/evidence/policy.py`,
`model/evidence/relational_decoder.py`, `training/tokenizer/losses_repr.py`.

---

## A. Claims the code no longer supports

Ordered by how badly a reviewer would react.

### A1. "The decoder is identity-at-initialisation by construction" — false

`5_design.tex:286-290`:
> Every sublayer is gated by a LayerScale initialised near zero and the residuals are
> zero-initialised, so at initialisation the decoder is *exactly* the untrained
> retrieval-plus-text-ensemble mechanism: every trained arm has an exact untrained control.

`6_experiments.tex:318-321` repeats it: "the decoder is identity-at-initialisation by construction".

**Code:** `relational_decoder.py:104` sets `layerscale_init = 0.1`, with the comment:

> LayerScale was 1e-4 so the stack was identity at init and the logits equalled the closed-form
> base exactly. With no base term that rationale is gone... It is still a stabiliser, just not an
> identity switch.

The closed-form base (`base_c`) was **removed** — measured reachable for only 15.5% of decisions,
and the two branches sat 6.4 logits apart, so the base decided by *which branch a candidate landed
on*. A candidate logit is now the readout on its token and nothing else.

**Consequence:** the untrained control still exists, but as a *separately built artifact*
(`make_step0_predictor.py` packages init weights in the evaluator's schema), not as an
architectural guarantee. This is a weaker but still perfectly good claim, and it is arguably more
honest — but it must be rewritten, because "by construction" is checkable and currently wrong.

### A2. Phase-A cross-placement term is deleted — 6 locations still describe it

Live recipe (`PHASE_A_B_AGREED_IMPLEMENTATION_PLAN.md` §"Final Phase-A Objective") is exactly:

```
JEPA masked contextual prediction + augmentation VICReg
```

Stale references:

| location | text |
|---|---|
| `5_design.tex:232-238` | "a low-weight subterm ties together *verified simultaneous* recordings... (18,400 paired events, three datasets)" |
| `5_design.tex:243` | "Earlier drafts also carried..." — the list of removed things is now missing this one |
| `6_experiments.tex:314` | "strip the masked-latent objective, then the cross-placement subterm" |
| `8_appendix.tex:233` | "the cross-placement subterm is weighted 0.1 within the relation term" |
| `8_appendix.tex:242` | "A fifth of each batch is reserved for verified simultaneous pairs" |
| `8_appendix.tex:271-273` | ablation rows R0/R1/R2 are built around the three-term recipe |
| `figures/fig_overview.tex:44` | "A2 relation: VICReg over views **+ cross-placement**" |

**Do not just delete it.** The verified-simultaneous-event asset did not disappear from the system
— it moved from Phase A's loss to **Phase B's episode construction**, where event identity groups
synchronous placements and enforces query/support leakage exclusion
(`PHASE_A_B_AGREED_IMPLEMENTATION_PLAN.md`: "Phase A does not sample by event identity. Phase B
uses event identity to group synchronous placements and prevent query/support leakage"). Relocate
the paragraph rather than dropping it; it is a stronger story in Phase B, where it underwrites
`tab:crossconfig`.

### A3. The Phase-B decoding paragraph describes a deleted mechanism

`5_design.tex:281-290` ("Candidate-aware decoding") makes three claims that are now false:

| paper | code |
|---|---|
| "refines each evidence item's label text as a residual in frozen sentence space — which is what lets a neighbour labelled *walking* retrieved in a stair-climbing context contribute as *walking upstairs*" | text-refinement residual removed; binding is via **randomised episode-local coreference slots** — a discrete shared embedding, so enrollment identity never has to be inferred from a cosine |
| "pools evidence as a residual on the retrieval prior" | no base term; retrieval scores enter as a **shift-invariant additive attention bias**, `log_softmax(s/τ) + log N` |
| `5_design.tex:278` "contributions are capped per source window and per label so one long recording cannot dominate a vote" | `PHASE_B_TRAINING_INTENT.md` §5: "no per-label or per-window cap alters that ranking" |

### A4. "No gradient step occurs anywhere to the right of freeze" — overclaim

`fig:overview` caption (`5_design.tex:29-30`):
> Phase~B adds no parameters to that path... no gradient step occurs anywhere to the right of
> \textsf{freeze}.

Phase B trains a 3-layer relational transformer (d=256, 4 heads) **and** four learned retrieval
projections, for 3,000 steps at lr 2e-4. The true claim is narrower and still strong: **no
parameter is added per label or per deployment, and no gradient step occurs at enrollment time.**
As written a reviewer reads "Phase B is parameter-free" and then finds the decoder in §5.4 two
paragraphs later.

### A5. Confidence and rejection is parked

`5_design.tex:292-297` presents calibrated rejection as part of the system; `fig_overview.tex:56`
puts "+ calibrated rejection" in the output node. `PHASE_B_TRAINING_INTENT.md` §6: "Confidence
calibration code remains available as a parked follow-up experiment, but it is not part of the
current Phase-B launch or claim." §11 repeats it. Either cut, or move to a one-line future-work
sentence.

### A6. `app:phaseb` episode spec is stale in every parameter

| paper | code (`policy.py`) |
|---|---|
| candidate budget "10--100% of the vocabulary" | `CANDIDATE_COUNT_RANGE = (2, 16)` — an absolute integer, deliberately decoupled from vocab size |
| support "0, 1, 2, 4, 8, or all" | `SUPPORT_COUNT_RANGE = (1, 8)` for training; `(1,2,4,8)` only for fixed validation canaries |
| "Head collapse... resisted by output decorrelation, projection diversity, and contribution-usage regularisation" | all removed; `PHASE_B_TRAINING_INTENT.md` §11: "There is no auxiliary retrieval term, corpus-classification head, subject-adversarial loss, EDL predictor loss, repeatability penalty" |
| — | missing: `ALIAS_PROBABILITY = 1/3`, `EPISODES_PER_STEP=8 × QUERIES_PER_EPISODE=8`, the paired support/zero counterfactual |

---

## B. What the code now measures, and how far the experiment section is from it

This matters more than §A. The experiments are drafted around **zero-shot cross-dataset macro-F1 +
a label-efficiency curve to k=full**. The code measures something narrower and sharper.

### B1. The roster is now sealed, and the paper doesn't say so

`policy.py` splits the held-out sets:

```python
PHASE_B_DEV_DATASETS  = ("motionsense", "realworld", "shoaib")
PHASE_B_TEST_DATASETS = ("inclusivehar", "usc_had", "tnda_har", "ut_complex")
```

`tab:datasets` lists all seven as one pool. The seal is the direct answer to "you selected
hyper-parameters on your evaluation set" and belongs in §6.1.3 (Protocol) beside the existing
label-leakage paragraph. It is free credibility that the paper currently discards.

### B2. The real controls are four, and none of them is in a table

`eval_enrollment.py` builds one fixed subject/candidate/query cohort with nested,
execution-independent support prefixes and compares against **support removal, shuffled support
labels, prototype, and ridge on exactly the same split** (`PHASE_B_TRAINING_INTENT.md` §13.7).

`tab:label_efficiency`'s only control is "harnet frozen + memory" — the mechanism-vs-representation
control. That is still worth having, but it is no longer the *primary* control, and the primary
ones are the ones currently losing:

**Measured, dev roster, cross-subject, 17,871 queries** (`PHASE_B_STEP0_CONTROL.md`):

| k | step 0 | step 1000 | prototype | ridge | chance |
|---:|---:|---:|---:|---:|---:|
| 0 | **16.18** | **9.44** | — | — | 13.77 |
| 1 | 45.34 | 46.53 | 52.85 | 52.28 | 13.77 |
| 2 | 49.41 | 49.39 | 54.95 | 54.42 | 13.77 |

Same-subject k=2: trained 63.88 vs prototype **83.29**.

Prototype and ridge are closed-form and use **zero Phase-B parameters**. They beat the trained
engine in every supported cell. §6.3's pre-registered kill criterion names the *probe* as the
rival — the actual rival that is winning needs no fitting at all, which is a harder comparison.
**Add a prototype/ridge row to `tab:label_efficiency`.** The kill criterion should name it.

### B3. Two measured facts that change the k=0 story

- **Training moves k=0 below chance** (16.18 → 9.44 cross-subject; 24.92 → 15.53 same-subject;
  worst cell shoaib 25.44 → 5.32 against a 14.29 floor). The untrained mechanism is *above* chance
  in both regimes. ConSE runs for free inside the mechanism and closed-vocabulary episodic CE
  trains it away.
- **Training makes real activity names actively harmful.** Same-subject k=2: coherent 63.88,
  neutral aliases **74.02**. At init the two are equivalent, as they should be. After training the
  model is 10 points better when the names are *meaningless*.

Both have diagnoses in hand (SBERT common mode 0.467, margin 0.151; the optimiser's cheapest move
is to suppress the semantic channel). That is a publishable negative result with a mechanism, and
it is the reason coreference slots exist. See §E.

### B4. Scope limits that must be stated, not discovered by a reviewer

`PHASE_B_TRAINING_INTENT.md` §7.1:

- Cross-configuration enrollment is only testable where the same activity **and** subject exist on
  more than one stream. **No sealed dataset supplies it.** Only Shoaib, in the dev protocol.
- TNDA-HAR has one subject and window-level execution ids → contributes the coherent zero-support
  condition only. It cannot support any same- or cross-subject enrollment claim. (The paper already
  excludes it from the subject-disjoint mean for a *different* reason — no subject ids — so the
  footnote needs a second clause.)
- Sealed adaptation claims therefore come from InclusiveHAR, USC-HAD, UT-Complex only.
- 35 of 93 labels exist on exactly one stream; 0 of 93 have only one subject.

`tab:crossconfig` as drafted (SP-SW-HAR / NFI / WISDM / XRF V2) is measurable — but those are
**training-corpus** streams. It is a legitimate mechanism claim; it must not read as held-out.

---

## C. Keep vs. cut

### Integral — this is the contribution, keep it in the body

- L1/L2 arguments and the entailment chain (physical-Hz anchoring ⇒ one flat memory is well-posed).
- The tokenizer's three physical commitments + observability masks + signed gravity DC.
- Per-**sensor** (not per-channel) conditioning, and the dataset-identity probe that tests it.
- Phase A = two label-free terms. One sentence each.
- Phase B = retrieval in sensor space, append-only memory, candidates declared at run time.
- **New, and currently absent:** the relative attention bias as the *only* differentiable route
  from candidate CE back to the retriever, given that top-k is hard. This is a real design decision
  with a real consequence (a missed support row can no longer be promoted), and the removed MIL
  boundary loss has a principled obituary: its positives were the *true* candidate's support, so it
  was label-conditioned at train time and absent at eval.
- **New, and currently absent:** coreference slots. They are the fix for the measured failure in
  §B3 — enrollment identity as a discrete shared embedding instead of a cosine.
- The untrained-control discipline (rewritten per §A1).

### Cut to the repo — currently in the paper, no reader depends on it

| what | where | keep instead |
|---|---|---|
| S=256 vs S=512 band-energy verification | `app:tokenizer` | one clause: "zero-padding is frequency-domain interpolation; verified not to alter band energies" |
| σ_k = c_k/2Q, the 10% Nyquist guard arithmetic | `5_design.tex:164-166` | the *concept* of two masks; constants to appendix or repo |
| sampling exponents n^0.25 / 25% cap / n^0.5 | `app:phasea` | the reason — "without tempering one free-living wrist corpus supplies 88% of every gradient" — is the interesting part and is already there |
| candidate/support/distractor staging percentages, 48-canary Cartesian product, hardness definition | `app:phaseb` | four regimes exist; alias episodes are load-bearing; partial enrollment is the only regime where the mechanism is the sole method that can answer |
| archive budget 250k, active view 16/label, refresh every 5 steps | not yet in paper — **keep it out** | at most one clause on *why* the active view exists (the archive is 28,648 `sitting` windows against a 30-window minimum; balance is restored by the view) |
| `ema_finetune` tokenizer mode | not yet in paper | one sentence that it exists and changes the privacy/versioning analysis, or omit for this submission |

The saving is roughly one appendix page, which is worth having because §B needs new tables.

---

## D. Figures

| figure | verdict |
|---|---|
| `fig_overview` | **3 edits.** (a) objective node → "A1 JEPA masked latent (EMA teacher) · A2 VICReg over two augmented views"; (b) drop "+ calibrated rejection" from the output node; (c) split the single Phase-B box so the reader can see *what is trainable* — retriever projections and relational decoder are parameters, and the caption currently implies otherwise (§A4). |
| `fig_evidence` | **Redraw.** It is drawn for the deleted design: band 3 is one opaque "candidate-aware evidence decoder", band 4 says "non-negative evidence per candidate" (plain logits now). Show the actual sequence `[candidates; background labels; query patches; evidence]`, the coreference slot binding an evidence row to a candidate, and the attention-bias edge running *back* to the retriever as the only gradient path. Right now the two mechanisms that are genuinely new are both invisible. |
| `fig_geometry` | **Highest-value change, and it closes the outstanding advisor note** (`HANDOFF_COWORK.md`: "the current fig_geometry uses 14 hand-picked pairs and never measures sensor similarity at all"). You now have a 248,351-window patch bank with per-label centroids on disk, so the measured version — *all* label pairs, text cosine on x, sensor-centroid cosine on y, inverted quadrant shaded — is cheap. Converts L2 from argument to measurement, which is exactly what was asked for. |
| **new: enrollment curve** | This is the paper's actual result. x = k ∈ {0,1,2,4,8}, y = macro-F1; lines for HALO memory, support-removed, shuffled-label, prototype, ridge, untrained step-0; paired subject-bootstrap CIs. One panel carries §6.3 and replaces most of `tab:label_efficiency`. |
| `fig_hook`, `fig_corpus`, `fig_rateinvariance`, `fig_gravity`, `fig_tokenizer` | **Unchanged.** All still true against current code. |

Also available and unused (`HANDOFF_COWORK.md`): `fig_zeroshot` (2×2 seen/unseen labels × configs)
and `fig_heterogeneity`, built in Cowork, vector PDF + source. `fig_zeroshot` would answer the
advisor's "which design addresses which challenge" note directly.

---

## E. The framing decision (needs Alex, not mechanical)

The manuscript is written as "HALO works". The measured state is:

- the **untrained** mechanism works — 47.5 macro-F1, edging harnet's 47.3;
- Phase-B training is net-negative so far — destroys k=0, is a wash at k=2 cross-subject
  (retriever −1.27 exactly cancelled by decoder +1.25), and loses to a closed-form prototype by 20
  points same-subject;
- the redesigned relational decoder is built and unit-tested but **has never been trained at
  scale** (`PHASE_B_TRAINING_INTENT.md` header; `PHASE_B_STEP0_CONTROL.md` §5).

Two viable framings:

1. **Wait.** Train the relational decoder, hope the numbers turn, keep the current framing.
2. **Reframe around the representation + mechanism.** The contribution becomes: physical-Hz
   anchoring makes one flat cross-device memory well-posed, and untrained retrieval over it already
   transfers competitively; learned candidate-conditioned decoding is reported as a negative result
   *with its diagnosis*. This is unusually well-instrumented — the retriever/decoder decomposition,
   the alias inversion, and the SBERT common-mode measurement are all in hand.

**Recommendation:** these are separable. Do §A now — the design section is wrong against the code
regardless of which way the results land, and every hour spent writing experiments against a
deleted decoder is wasted. Do §B1/B4 now too (seal + scope limits are protocol, not results). Defer
the §E choice until the relational decoder has trained once — but pre-write framing 2, because the
kill criterion in §6.3 is already pre-registered and the paper is committed to reporting it.
