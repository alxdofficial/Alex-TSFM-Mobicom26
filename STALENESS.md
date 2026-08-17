# Paper-vs-code staleness audit — 2026-08-17

Compares the manuscript against `halo/` at working-tree state (commit `8e64e39` plus uncommitted
changes). Verified by re-deriving each quantity from the repo, not by reading prose.

**Headline: the manuscript is not internally wrong.** Every number I checked reproduces exactly from
the artifact it names (`phase_a_sensor_v1_20260813_v2/best.pt`, step 4,000). The paper is a
provenance-complete snapshot of a state the repository has since left behind. Staleness is
*divergence*, not error — which is why the fix is a pinning decision, not a correction pass.

---

## 0. The one decision that gates everything else

| | Manuscript | Repository head |
|---|---|---|
| Phase-A checkpoint | `phase_a_sensor_v1_20260813_v2/best.pt`, step 4,000 | `phase_a_fixed_1s_rotation_20260817/best.pt`, step 27,000 |
| Bank | 250k windows / 7,653,242 sensor rows / 161 labels | 2,629,972 sensor rows / 166-label vocab, rebuilt 2026-08-17 09:53 |
| Resolvability table | schema-2, bound to v2 | schema-2, bound to `fixed_1s`, rebuilt 09:54 |
| Gate artifact | `artifact_version: 2` | `load_gate` requires `3` — **the paper's gate can no longer be loaded by current code** |
| Enrollment results | Table 3, verified | **none exist for the new checkpoint** |

The new checkpoint has a bank and a resolvability table but **no gate and no enrollment run**. So the
results table cannot be repinned today. Two coherent options:

- **(A) Keep the manuscript pinned to v2.** Everything stays valid; add an explicit "this is not repo
  head" statement so a reader who opens the code is not misled. Cheap, honest, available now.
- **(B) Repin to `fixed_1s`.** Requires `gate_predictor --fit --bank` then a development
  `eval_enrollment` run, and rewrites Table 3, Appendix E, Appendix F, the abstract, and Table 2.

I have applied (A) — marked in-place with `\TODO{}` — because (B) would print numbers that do not
yet exist. Flipping to (B) is a one-command decision once those artifacts are built.

---

## 1. Corpus counts — Table 2 and §6.1 (STALE, verified)

Native grids were rebuilt on 2026-08-17 05:40. Two changes moved every Phase-A count: native grids
now retain each recording's final partial context (`lengths.npy`), and KU-HAR was reconverted.

| Dataset | paper | current | Δ |
|---|---:|---:|---:|
| CAPTURE-24 | 1,530,792 | 1,543,573 | +12,781 |
| WISDM | 70,205 | 72,227 | +2,022 |
| RealDISP | 50,886 | 63,432 | +12,546 |
| DSADS | 37,815 | 38,175 | +360 |
| XRF V2 | 34,764 | 59,442 | +24,678 |
| NFI-FARED | 26,876 | 29,184 | +2,308 |
| PhyTMo | 24,684 | 29,180 | +4,496 |
| HARMES | 18,576 | 25,583 | +7,007 |
| UniMiB SHAR | 11,771 | 11,771 | 0 |
| Opportunity | 10,765 | 28,785 | +18,020 |
| FORTH-TRACE | 10,340 | 12,330 | +1,990 |
| UCI-HAR | 10,299 | 10,299 | 0 |
| HHAR | 9,587 | 11,587 | +2,000 |
| KU-HAR | 9,324 | 11,263 | +1,939 |
| MM-Fit | 6,280 | 8,496 | +2,216 |
| SP-SW-HAR | 3,757 | 3,757 | 0 |
| PAMAP2 | 3,186 | 3,292 | +106 |
| MHEALTH | 1,104 | 1,230 | +126 |
| **total** | **1,871,011** | **1,963,606** | **+92,595** |

The delta is exactly the corpus-wide partial-window count. Canonical labels 161 → **166**.

**Not stale:** the external-evaluation column is byte-for-byte correct — those ten datasets' grids
were *not* rebuilt (they have no `lengths.npy`; 56 of 85 native grid directories carry the sidecar).
Configuration counts 56 and 14 are unchanged, so `figures/fig_corpus.tex` needs no redraw.

> Side finding for the code, not the paper: training grids keep their final partial context and
> evaluation grids silently discard theirs. `GridRef.load_lengths()` falls back to all-full for
> legacy grids, so nothing errors — but train and eval are on different grid contracts.

---

## 2. Design section describes a recipe the code no longer runs (STALE)

All of these were true of the v2 artifact and are false of the current default recipe.

| Location | Manuscript | Code now |
|---|---|---|
| §4.4 encoder, `fig_overview` | "two patch-duration resolutions" | `multiresolution=False`, fixed 1.0 s |
| §5, abstract | descriptor-mask retrieval as part of the JEPA family | `descriptor_weight=0.0`, `descriptor_event_p=0.0` — head frozen and forward path skipped |
| §4.3.2 | "two physical augmentations" | rotation-only at p=1.0, and rotation moved `CONFIG_GROUP` → `NUISANCE_GROUP`, so the two views now draw **independent** rotations |
| §5 | coefficients calibrated at step 2,000 | code default is now 500 |
| App. E | batch 256 / 30,000 steps / lr 3e-4 / warmup 1,000 / wd 0.05 / EMA 0.996 | defaults now 1,024 / 7,500 / 6e-4 / 250 / 0.1 / 0.984 — **never used to produce a checkpoint** |
| App. E | short 0.4–0.8 s, long 1.0–1.5 s, ratio 1.75 | now reachable only via the `--multiresolution` ablation |

The rotation-group move is the substantive one: independent per-view SO(3) at p=1.0 asks VICReg for
full orientation invariance, which erases the gravity direction that §3.2 and §4.2.1 rely on for
posture. It also contradicts the reasoning still in `augmentations.py:247`. Measured effect on the
new checkpoint: internal val kNN-BA 0.352 → 0.257, with `grad_cosine/jepa_vs_vicreg` finishing at
−0.90. Not attributable — three things changed at once — but it is the change to interrogate.

---

## 3. Verified current — no action

- Filterbank: $S{=}256$, $K{=}32$, 0.3–15 Hz, $Q{=}4$, observability and resolution masks. ✓
- Encoder: width 256, 6 layers, 8 heads, MiniLM, physical-time RoPE. ✓
- `sensor_bias`: 14-d = 7 statistics + 7 support bits; field list matches `SENSOR_BIAS_FIELDS`
  exactly (gravity magnitude, gravity presence, noise floor, quantization step, clip fraction, rate
  fidelity, rest bias). ✓
- Gate: rank 2, veto 0.15, geometric mean of query and row sides, applied before top-$k$, budget 64,
  bias blend weight 0. ✓
- Prediction order, vote rule, fusion, and all five controls match `admissible_retrieval.py`. ✓
- Test roster (7 datasets) matches `PHASE_B_TEST_DATASETS`. ✓
- **Table 3 is exact.** Every cell reproduces from the `*_unweighted` columns of
  `phase_a_checkpoint_selection_20260816/comparison.json`.
- **Gate telemetry is exact.** Re-derived across all 15 dev cells: min 0.2450, max 0.7329,
  `vetoed_fraction` 0.0 everywhere. Paper says 0.245–0.733, no vetoed rows. ✓

---

## 4. Findings the paper should absorb (new since it was written)

§6.3 currently says the veto never fires and the gate "only reweights already selected evidence."
That is right but understates it. Three sharper facts are now measured:

1. **On same-configuration cells the gate is structurally a per-candidate prior.** The evidence rows
   share the query's descriptor, so the geometric mean `sqrt(g(query,c)·g(row,c))` collapses to a
   function of the candidate alone. Multiplying an evidence-based vote by a query-independent prior
   can only add bias. Every reported cell is same-configuration, so the protocol as run can measure
   the mechanism's cost and never its benefit.
2. **Why the veto is inert is now attributable.** On the freshly rebuilt table the fitted neutral is
   0.190 against a veto of 0.15, so abstention *rescues* unfamiliar rows from the veto rather than
   triggering it. The constant was calibrated when the table median was 0.33; it is now 0.22, and
   the fitted-cell fraction below veto moved 14.4% → 29.9%. Nothing asserts `neutral > veto`; if a
   future fit inverts them, an unfamiliar configuration is vetoed on every candidate.
3. **Alias cells carry no gate information.** `semantic_labels=False` sets admissibility to all-ones,
   so gated and ungated arms are bit-identical there (confirmed: 51.3247 vs 51.3247).

(1) is the strongest available reading of the negative result and belongs in §6.3 — it turns "the
gate did not help" into "the protocol could not have shown it helping," which is a more useful
statement and directly motivates the cross-configuration cell listed in §6.4.

---

## 5. Figures

| File | Status |
|---|---|
| `fig_corpus.tex` | **current** — 56/14 configurations and the per-rate histogram re-verified today |
| `fig_evidence.tex` | **current** — matches the implemented prediction order |
| `fig_geometry.tex` | **current** — qualitative, no numbers |
| `fig_overview.tex` | **WITHDRAWN** — replaced by a full red TODO placeholder box. Original TikZ preserved verbatim in `fig_overview_v2artifact.tex`; redraw from it after the repin |
| `fig_gravity.tex` | retired, comment-only stub, no longer `\input` anywhere |
| `fig_hook.tex` | retired, ditto |
| `fig_rateinvariance.tex` | retired, ditto |
| `fig_tokenizer.tex` | retired, ditto |

The four retirements are already correct and traceable; each names why it was pulled and what would
license its return. No redraws attempted, per instruction.

---

## 6. Suggested order of work

1. Decide pinning (§0). Everything below assumes (B).
2. `gate_predictor --fit --bank` against the rebuilt artifacts — but settle the veto/neutral
   calibration first, or the run bakes in an uncalibrated threshold.
3. Add at least one cross-configuration development cell before scoring, or §6.3 repeats.
4. Regenerate Table 2 and the abstract's corpus sentence from the current grids (§1).
5. Rewrite Appendix E and the §4.4/§5 recipe claims to the new run (§2).
6. Only then reconsider the retired figures.
