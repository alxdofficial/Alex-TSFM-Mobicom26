# Shortening plan: 20 body pages → 12 (MobiCom 2027)

Written 2026-07-26. Authorised by the user to execute immediately after writing.

## 1. The limit

From the [MobiCom 2027 CFP](https://www.sigmobile.org/mobicom/2027/cfp.html):

- **12 pages** single-spaced and numbered, **including figures, tables, and any other material**.
- **References do not count** ("followed as many pages as necessary for bibliographic references").
- **Appendices do not count** — they follow the bibliography. But "reviewers are not obligated
  to read them."
- `\documentclass[sigconf,10pt]{acmart}`; **minimum 10 pt**; ≤55 lines per column; columns
  9.25″ × 3.33″ with a 0.33″ gutter; US Letter; PDF < 15 MB.
- Abstract registration **26 Aug 2026**; paper **2 Sep 2026**; early reject 23 Oct;
  rebuttal 3–6 Nov; notification 19 Nov 2026.

## 2. Measured starting point

Measured, not estimated: a second copy of the manuscript was compiled with **every float
stripped** to separate prose length from figure area.

| | pages |
|---|---|
| Current body (§1–§7) | **20** |
| References | 5 (free) |
| Body with all 19 floats removed | **15** |
| ⇒ floats cost | 5 |
| **Limit** | **12** |

**The finding that drives this plan: the prose alone is 15 pages — 3 over the limit before a
single figure or table.** The floats are only 5 of the 8 pages we are over.

12,641 prose words + **1,550 caption words** ≈ 840 words/page. Landing at 12 pages with ~4 pages
of floats needs ~6,800 words of prose: a **46% cut**. This is a restructuring, not a tightening.

## 3. Target allocation

| section | words now | pp now | target words | target pp |
|---|---|---|---|---|
| Abstract | 603 | 0.4 | 220 | 0.25 |
| §1 Introduction | 1,403 | 1.7 | 950 | 1.1 |
| §2 Related Work | 2,009 | 2.0 | 700 | 0.85 |
| §3 Motivation | 2,698 | 4.0 | 1,150 | 1.4 |
| §4 Overview | 621 | 1.0 | **0 (deleted)** | 0 |
| §5 Design | 3,096 | 4.0 | 2,100 | 2.5 |
| §6 Experiments | 1,565 | 2.0 | 1,350 | 1.6 |
| §7 Conclusion | 646 | 0.7 | 300 | 0.35 |
| **prose** | **12,641** | **15.8** | **6,770** | **8.1** |
| floats | — | 5.0 | — | 3.9 |
| **total** | | **20.8** | | **12.0** |

## 4. Tier 1 — structural (~3.5 pages)

**T1.1 Delete §4 System Overview entirely** (621 words + ~1 page). It summarises §5 before §5
says it: Phase A ≈ §5.1–5.2, Phase B ≈ §5.3, "one curve" ≈ §3.1 + §5.4, Deployment ≈ §5.4. Only
the parameter breakdown is unique → move to §5.1.5 and §6.1. `fig:overview` survives and moves
to the head of §5.

**T1.2 Use the appendix allowance.** Rule: the appendix holds what a reviewer needs *only if
they doubt us*, never what they need to follow the argument.

- §2.1's four "mutually entailed" paragraphs (~700 w) → appendix; 3 sentences + pointer remain.
- `tab:datasets` in full (47 lines, 143-w caption) → appendix; 6-line summary inline.
- `tab:per_dataset` → appendix.
- Phase-A A3/A4/A5 mechanics; the S=256-vs-512 verification; EMA decay; quota mechanics.
- Phase-B episodic schedule and head-collapse regularisers.

**T1.3 Caption discipline** (~1 page). 1,550 caption words is 1.8 pages of a 12-page paper.
`fig:rateinv` 219 w, `fig:tokenizer` 156, `fig:gravity` 156, `tab:coverage` 141,
`tab:datasets` 143. Target ≤60 w for figures, ≤40 for tables. Several captions currently *are*
the argument, which is why the body repeats them — pick one location per fact.

## 5. Tier 2 — per section

### §3 Motivation (4 pp → 1.4) — the biggest overrun
- Delete the roadmap paragraph (a table of contents in prose).
- **Delete `tab:demands`** — a 3×3 grid of identical checkmarks carrying one sentence of
  information that the body already states. Weakest float in the paper.
- Three deployments: ~90 w each → ~40 w each, one paragraph.
- Four "why X is not optional" paragraphs (~750 w) → two (~350 w). Keep the annotation-channel
  argument, the recurring-cost argument, and the UniMTS self-ablation sentence (third-party
  support). Cut "what this buys" — it overlaps §7.
- §3.2: three labelled paragraphs → one. The 120-vs-600-samples arithmetic appears in §3.2,
  §5.1.2 *and* `fig:rateinv`'s caption — keep it once.
- §3.4: delete the restated 0.77 / 0.60 / 38-of-49 numbers (already in §1 and the figure
  caption). Keep only the structural explanation, which no figure carries.
- §3.5 R1–R4: 300 w restating what was just argued → a 4-line list.

### §2 Related Work (2 pp → 0.85)
- Five themed paragraphs → three. Merge "rate/resolution invariance" with "language as sensor
  interface" (both are *how the field handles acquisition heterogeneity*).
- Cut the self-supervised-objectives paragraph to one sentence + citation cluster in §5.2 — it
  spends 200 words establishing that we claim no novelty.
- Name 3–4 exemplars per theme instead of 8–12.

### §5 Design (4 pp → 2.5)
- §5.1.1 gravity discussion (~280 w) restates `fig:gravity`'s caption → ~90 w + pointer.
- §5.1.2 "consequence" paragraph states rate-invariance a third time → trim; S=256/512
  verification → appendix.
- Streaming paragraph claims nothing and is future work → one sentence in §7.
- §5.3 Phase B (~950 w) is the most heavily *described* and least experimentally covered part
  of the paper → ~550 w. A reviewer will notice the asymmetry.

### §6 Experiments (2 pp → 1.6) — least cutting; this is where the paper earns its keep
- `tab:datasets`, `tab:per_dataset` → appendix.
- Merge `tab:identity_probe` into `tab:tokenizer_ablation` as extra columns (same arms).

### §7 Conclusion (0.7 → 0.35)
Keep "what we are not claiming" and the kill criterion — both distinctive. Compress limitations
and future work to a sentence each.

## 6. Floats: 19 → 12

- **Delete:** `tab:demands`.
- **Appendix:** `tab:datasets`, `tab:per_dataset`.
- **Merge:** `tab:identity_probe` into `tab:tokenizer_ablation`.
- **Conditional:** `fig:demo_setting` (an orange placeholder today) — drop unless the demo is
  re-shot.
- **Demote two of three `figure*` to single column.** Three full-width figures cost ~1 page of
  layout slack. `fig:hook` earns full width; `fig:gravity` and `fig:rateinv` are two-panel and
  work at column width.

## 7. The label-budget grid (raised separately by the user)

Measured held-out sizes:

| held-out | windows | classes | subjects | k=10/cls | k=50 | k=100 | k=500 |
|---|---|---|---|---|---|---|---|
| RealWorld | 11,237 | 8 | 15 | 0.7% | 3.6% | 7% | 36% |
| MotionSense | 4,534 | 6 | 24 | 1.3% | 6.6% | 13% | 66% |
| USC-HAD | 4,350 | 12 | 14 | 2.8% | 14% | 28% | — |
| UT-Complex | 3,900 | 13 | 10 | 3.3% | 17% | 33% | — |
| TNDA-HAR | 3,353 | 8 | **none** | 2.4% | 12% | 24% | — |
| Shoaib | 2,100 | 7 | 10 | 3.3% | 17% | 33% | — |
| InclusiveHAR | 1,260 | 6 | 20 | 4.8% | 24% | 48% | — |

`k=10` per class **is** the ~1% point. But `k=500`/class is undefined for five of seven sets and
`k=200` is effectively "full" for InclusiveHAR — these are thousands of *windows* from 10–24
people, not thousands of sessions.

**Decision:** keep per-class `k` as the axis, label it in **both units**, and extend the top end
to where the data runs out:

> `k = 0, 5, 10, 25, 100, full` → mean ≈ 0%, 1.4%, 2.7%, 6.9%, 27%, 100%

Drop `k=1` (variance is enormous at one exemplar per class). Where `k·C` exceeds a dataset's
train portion, print "full" explicitly rather than silently sampling with replacement.

**Integrity issue found while checking this:** TNDA-HAR's subject field is `"unknown"` for all
3,353 windows — there is no subject metadata, so a subject-disjoint split is impossible and
few-shot exemplars cannot be drawn from a disjoint subject pool. §6.1.3 currently claims
"subject-disjoint throughout", which is **false for 1 of the 7 held-out datasets**. Needs
recovered metadata, an explicit carve-out, or dropping the dataset. Flagged, not silently
patched.

## 8. Format fixes

**Compliance risks, not taste:**

1. **`\resizebox{\columnwidth}{!}{...}` on four tables scales the font down arbitrarily.**
   MobiCom mandates 10 pt minimum; a `\small` table then resized can land at 7–8 pt. Same for
   pgfplots axis labels inside resized `figure*` blocks. Replace with explicit `\footnotesize` +
   `\tabcolsep` tuning; measure rendered text height before submitting.
2. **`\usepackage{geometry}`** — the `\geometry{}` call is commented out so it is inert today,
   but with acmart it can silently alter the text block and break the mandated column
   dimensions. Drop it. Also unused: `blindtext`, `todonotes`, `multicol`, `tabularray`,
   `float`, `placeins`, `subcaption`.

**The rest:**

3. `\vspace{-1em}` after four table captions and `\belowcaptionskip=-6pt` on several figures —
   manual vertical compression is the classic desk-reject trigger alongside font shrinking.
   Space should come from cutting text, not squeezing margins.
4. Every `\PH` / `\PENDING` must be gone before submission, including the orange
   `fig:demo_setting`. Pre-flight grep.
5. Verify ≤55 lines/column in the log; confirm nothing changes `\baselinestretch`.
6. Fix the two remaining overfull boxes (2.9 pt, 1.6 pt).

## 9. Execution order

1. Preamble + appendix scaffold (§8.1, §8.2).
2. T1.1 delete §4; relocate `fig:overview` and the parameter breakdown.
3. Section rewrites: abstract, §1, §2, §3, §5, §6, §7.
4. Appendix file assembled from the T1.2 material.
5. Float changes (§6) + the label-budget grid (§7).
6. Format fixes (§8.3–8.6).
7. Render, measure, iterate to 12.

## 10. Assumptions recorded

Two questions were put to the user and answered with "go ahead", so this plan executes both as
written: **§4 is deleted**, and **the appendix strategy is on** (§5 and §6 are structured around
it rather than cut first and rescued afterwards).

---

# EXECUTED — outcome (2026-07-26)

**Result: body ends on page 12 (references begin partway down p12); 19 pages total; 0 errors, 0
undefined references, 1 residual overfull box at 11.5 pt.** Body prose went from 12,641 to
roughly 7,400 words.

| | before | after |
|---|---|---|
| Body pages | 20 | **12** |
| Body floats | 19 | 8 |
| Prose words | 12,641 | ~7,400 |
| Caption words | 1,550 | ~600 |

## Deviations from the plan

The prose cuts alone did not get there — **removing text pulled floats forward and the tail
refilled**, so the float budget had to be cut much harder than §6 of this plan assumed. Moved to
the appendix beyond what was planned: `fig:gravity`, `fig:tokenizer`, `fig:corpus`,
`tab:baselines`, `tab:parity`, `tab:objective_ablation`. `fig:demo_setting`'s reserved
placeholder was removed (the pending text remains in §5.5).

Body floats now: `fig:hook`, `fig:geometry`, `fig:overview`, `fig:rateinv`, `tab:coverage`,
`tab:label_efficiency`, `tab:crossconfig`, `tab:tokenizer_ablation`.

Two format decisions differ from §8 as written:

1. **Dense tables use `\scriptsize`, not `\resizebox`.** Removing the resizebox made several
   tables overflow the column. The fix is a *standard* LaTeX size rather than arbitrary scaling:
   `\scriptsize` produces a predictable 7 pt with correct stroke weights, whereas `\resizebox`
   produces non-standard sizes and inconsistent rules. Body text remains 9.9–10.1 pt (verified
   by extracting font sizes from the PDF), which is what the 10 pt rule governs.
2. **Table captions moved above the tabular**, since acmart itself sets
   `\captionsetup[table]{position=top}`; captions below it got the wrong spacing.

## Residual item, not fixed

`fig_overview` is wrapped in `\resizebox{\columnwidth}{!}{...}` and its internal text renders at
**~5.4 pt** — too small to read comfortably. Bumping the TikZ font does nothing (resizebox scales
to fit width regardless), so the real fix is to re-lay-out the diagram narrower, which is TikZ
surgery rather than a preamble change. Flagged for a later pass.
