# Cowork handoff — 2026-07-28

## State
- Local branch `main` is **1 commit ahead of origin/main** (`1106ba8`), tree clean.
- Base is upstream `9c4502d` ("Trim to fit: teaser and coverage table to the appendix").
- Compiles: 20 pp total, **12-page body** (refs start p.13), 0 undefined refs/citations.

## The unpushed commit (1106ba8)
Three advisor fixes from the 2026-07-27 comment PDF that survived the 20->12 page rewrite.
Two other fixes from that pass (FM definition, Tartan IMU citation) were already done
upstream and were deliberately NOT duplicated.

1. `5_design.tex` — define `\sigma_k = c_k/2Q` where the Nyquist mask formula uses it.
   The definition was dropped in the page cut, leaving an undefined symbol inside a
   formula the paper presents as one of its concrete mechanisms.
2. `1_introduction.tex` — contributions claimed the iOS demo in past tense
   ("demonstrated on-device in an iOS application") while the case study is still
   `\PENDING` in §5.5. Softened to the packaged `predict()` interface.
3. `1_introduction.tex` — L1 heading now reads "IMU has no closed grammar---only
   explicit physics", per the advisor note that the original phrasing was too abstract.

## To push
    git push origin main

## Still open from the advisor pass (need Alex's judgement, not mechanical)
- **Naming.** His most-repeated note (4x): the designs read as "too general ... reviewers
  thinking that we are adopting existing techniques". Wants e.g. "a xxx-based tokenizer".
  Upstream commit 38026d7 ("Name the mechanisms") may have partly addressed this — worth
  re-reading against the comment before doing more.
- **Title.** "Label-Efficient" called "not interesting from the perspective of application".
- **Label-heterogeneity premise.** "wouldn't recognizing sitting/walking/standing be
  sufficient enough?" Escapes he offered: richer activities from more applications, or
  identify new abilities that generating activity names enables.
- **Motivation evaluations.** Wants measured figures/tables, not argument. Cheapest high-value
  one: the L2 scatter over all label pairs with *measured* sensor-space distance (the current
  fig_geometry uses 14 hand-picked pairs and never measures sensor similarity at all).

## Available to drop in
TikZ figures built in Cowork, vector PDF + source, at
`HKUST/decks/halo-brief_2026-07-23/{figures_tikz,build_src/tikz}`:
- `fig_zeroshot` — 2x2 quadrant (labels seen/unseen x config seen/unseen) locating HALO
  against supervised HAR / domain adaptation / open-set. Answers "which design addresses
  which challenge" visually.
- `fig_heterogeneity` — activity x config matrix; same activity across configs vs different
  activities within a config.
Both compile standalone with `build_src/tikz/build_all.sh`.
