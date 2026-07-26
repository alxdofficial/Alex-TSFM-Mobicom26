# HALO manuscript revision plan — v3 (2026-07-26)

> ## MARKING POLICY (decided 2026-07-26, supersedes everything below on this point)
>
> **Nothing is marked "outdated".** Anything inconsistent with the current design is *deleted*
> outright — the manuscript states one version of the project and nothing else. Git history is
> the archive. The orange markers are now purely forward-looking:
>
> | macro | meaning |
> |---|---|
> | `\PENDING{...}` | an orange note: this prose will be written once the run lands |
> | `\PH` | one orange bar standing in for a number in a results table |
> | `\PHFIG{text}{height}` | an orange box reserving vertical space for a figure |
>
> `\OUTDATED` and `\STALE` are gone. There are no references to prior versions anywhere in the
> source, and no v1 image assets remain (`fig_hook.png` and `demo_setting.png` were the last two
> and are deleted — the hook is now `figures/fig_hook.tex` in TikZ; the case-study figure is a
> reserved orange box).
>
> ## STATUS: Waves 1–3 EXECUTED, 2026-07-26.
>
> The manuscript now reflects this plan. What actually shipped, and where it deviated:
>
> | plan item | outcome |
> |---|---|
> | abstract, §1, §2, §3, §4, §5, §6, §7 | **all rewritten** |
> | streaming | **demoted** as recommended — no claim in abstract/§1/§3/§4; one paragraph in §5.1.5 as a design affordance; listed in §7 future work |
> | five v1 result tables + severe-OOD table + v1 UMAP | **deleted from source** — recoverable from git history |
> | stale figure assets | **all deleted** — `fig_alignment`, `fig_encoder`, `fig_overview.png`, `fig_embedding_umap`, `fig_dataset_scatter`, `fig_dataset_discrepancy`, `fig_native_ablation`, `fig_patch_sensitivity`, `demo_performance`, `fig_hook.png`, `demo_setting.png`. **No raster figures remain**; every figure is TikZ or a reserved box. |
> | `fig_label_geometry` | **built** as `figures/fig_geometry.tex` (TikZ, measured data) |
> | new figures beyond plan | `fig_corpus.tex`, `fig_tokenizer.tex`, `fig_overview.tex`, `fig_hook.tex` — all TikZ. The v1 hook depicted a shared IMU–text embedding space with cosine-similarity recognition, i.e. exactly the mechanism §3.3 argues against, so it was rebuilt to carry L1/L2/L3. |
> | `fig_label_efficiency` | **not drawn** — a figure with no data would be a placeholder; Table `tab:label_efficiency` carries the skeleton instead |
> | T1–T8 | T1/T2 written with real numbers; T3 (label-efficiency curve), T4 (per-dataset), T5 (objective ablation), T6 (tokenizer bias), T7 (dataset-identity probe), T8 (parity) are marked skeletons |
> | citations | 4 verification agents run; **6 entries had fabricated or wrong author lists** (`crosshar`, `lanhar`, `mitra`, `recgym`, `kuhar`, `vttconiot`), `wisdm` cited the wrong dataset entirely, `harth` pointed at an unrelated DOI — all fixed; 40 → 97 entries |
> | **title** | changed: "Language-**Aligned** … Open-Set" → "Language-**Conditioned** … Label-Efficient". The old title asserted the exact thing §3.3 now argues against. Acronym unchanged. |
>
> **New in §2.1:** a coverage table over the closest systems plus the *mutual-entailment* argument —
> rate anchoring is a precondition for the memory, conditioning granularity is forced by the
> retrieval, gravity preservation is forced by the geometry finding, and label-free pretraining is
> forced by the same shortcut argument — so the contribution is a dependency chain, not a list.
>
> **Open decisions still outstanding** (see §6 below): the iOS demo re-shoot, and whether to rename
> the "evidence engine" given ZARA (ACL 2026) publishes "evidence-grounded" LLM agents for motion
> time series in an adjacent venue.
>
> **Wave 4 (fill the numbers) has not started** and is gated on the Phase-A training run.

Supersedes `REVISION_PLAN.md` (2026-07-05, now deleted). That plan was written for the *July-6* reframe
("single language-alignment stage + streaming"). The July 20--25 redesign invalidates parts of the
reframe itself, so this document restates the target and the per-file disposition.

**Purpose of this pass:** produce a manuscript the advisor can read end-to-end for a first scan.
Everything that can be written from decided design + implemented code IS written. Everything that
depends on an unrun experiment is a marked skeleton. Nothing states an unmeasured number.

---

## 1. North star — the three-legged argument

The paper is ONE argument in three steps, not a feature list. Every section must serve it.

| leg | claim | consequence |
|---|---|---|
| **L1** | IMU has **no closed grammar**: re-orientation, mounting, or unknown on-device filtering can put a recording outside any corpus. Scale reduces surprise, it does not close the space. | Compatibility must be **inductive bias**, not data. → the tokenizer with *calibrated* assumptions (sensor-grouped axes, physical-Hz filterbank, language-supplied sensor identity). |
| **L2** | **Sensor similarity ≠ linguistic similarity**, bidirectionally (*drinking coffee*/*drinking water* = same motion, distant strings; *sitting*/*sitting down* = opposite motions, near-identical strings). | A single projection into label-text space is mis-specified and pushes toward dataset memorisation. → the **evidence engine**: retrieve in sensor space, vote in label space. |
| **L3** | The real quantity is **label efficiency**, not zero-shot vs fine-tuning. | ONE curve: k=0 (no target labels) → k≤10 (demonstrated exemplars, no gradient updates, nothing leaves the device) → full supervision. |

**Positioning rule.** L1 and L2 are the contributions; L3 is how we measure them. We do not claim
"best zero-shot HAR" as the headline; we claim a foundation that is more label-efficient at every
point on the curve, and we give the mechanistic reason why.

---

## 2. What changed since the manuscript was last written

Source of every edit below.

1. **Phase A is now entirely activity-label-free SSL.** Objective set: fixed-feature masked
   prediction + EMA-latent masked prediction + augmentation VICReg + time--frequency VICReg +
   simultaneous-placement VICReg + a small physical-grounding rail. The manuscript's central
   sentence, "trained in a single language-alignment stage", describes a model we no longer train.
2. **Phase B is the evidence engine** (memory bank, retrieval, candidate-aware decoder, patch
   evidence, calibrated rejection). Entirely absent from the manuscript.
3. **Streaming is unfunded.** `temporal_mode='full'|'causal'` exists in the encoder, but nothing in
   the current plan trains causal, distils offline→online, or measures streaming equivalence.
   **DECISION REQUIRED (§6).**
4. **Corpus/counts:** 12 train / 7 held-out; 93 labels; 300,231 train / 42,909 val windows;
   ~7.17M encoder + ~2.81M decoder (not 35M).
5. **Baselines:** harnet, UniMTS, CrossHAR, LiMU-BERT, NormWear (MOMENT/LanHAR/LLaSA dropped).
6. **Data corrections that invalidate prior cells:** USC-HAD gyroscope was in deg/s (57.3× too
   large) — every pre-fix usc_had number is void; 95 physically impossible windows dropped;
   validation now covers all 93 labels.
7. **Retracted results must stay retracted:** the 49.5 evidence-decoder headline and the
   "beats harnet" reading. No number from them may reappear.

---

## 3. Per-file disposition

Legend: **W** = writable now · **S** = skeleton (marked, awaiting experiment) · **D** = delete ·
**R** = regenerate asset.

| file | disposition | notes |
|---|---|---|
| `0_main.tex` (abstract) | **W — DRAFTED** | Rewritten to L1/L2/L3. No numbers. Keeps video link + release footnote. |
| `1_introduction.tex` | **W** | Rewrite around L1/L2/L3; contribution bullets become the three legs. `\OUTDATED` the v1 results paragraph (keep text for reference). |
| `2_related_work.tex` | **W (light)** | Reposition vs harnet / UniMTS / NormWear / CrossHAR / LiMU-BERT. Add: retrieval-augmented & query-by-example recognition; VICReg/BYOL/I-JEPA lineage for the SSL objectives. ⚠ gated on the open few-shot/QbE prior-art sweep (§6). |
| `3_motivation.tex` | **W + NEW** | Keep heterogeneity subsection (update counts). **Replace** "limited generalization" with two new subsections: *No closed grammar* (L1) and *Sensor vs linguistic geometry* (L2, with the measured figure). Requirements list R1--R4 → restated as L1/L2/L3 consequences. |
| `4_overview.tex` | **W** | Rewrite to the two-phase system: Phase A label-free representation pretraining; Phase B evidence engine. Remove "single language-alignment stage". Param split updated. |
| `5_design.tex` | **W (mostly)** | Old subsections (adaptive-pooling tokenizer, spectral-temporal CNN, sinusoidal PE, Stage-1 SSL, soft contrastive alignment) → **D**, replaced. New, all implemented so writable: (a) filterbank tokenizer + sensor grouping; (b) factored conditioning (axis role + per-sensor identity); (c) Phase-A objective set; (d) evidence engine: memory, patch retrieval, candidate-aware decoder, confidence/rejection. |
| `6_experiments.tex` | **S** | Red banner. Protocol prose kept + updated (leakage-free ZS-XD, subject-disjoint, ConSE bridge, parity). Dataset + baseline tables **W**. All result tables → skeletons. Severe-OOD table → **D**. |
| `7_conclusion.tex` | **W** | Rewrite to L1/L2/L3; drop unmeasured claims; future work → data scaling, family-holdout evaluation, streaming (if demoted). |
| `reference.bib` | **W** | Add: VICReg, BYOL, I-JEPA, RoPE, constant-Q/filterbank, retrieval-augmented classification, query-by-example HAR, harnet/ssl-wearables, UniMTS, NormWear. |

---

## 4. Tables — inventory and disposition

| # | table | disposition |
|---|---|---|
| T1 | Dataset overview | **W** — rebuild for 12 train / 7 held-out, 93 labels |
| T2 | Baseline overview | **W** — harnet / UniMTS / CrossHAR / LiMU-BERT / NormWear + bridge tier |
| T3 | **Label-efficiency curve** (NEW, headline) | **S** — k ∈ {0,1,5,10,50,full} × {HALO retrieval, HALO probe, harnet probe, **harnet+retrieval**} |
| T4 | Per-dataset ZS-XD | **S** |
| T5 | Phase-A objective ablation | **S** — one row per objective arm |
| T6 | **Tokenizer inductive-bias ablation** (NEW, L1 evidence) | **S** — sensor-grouped+factored vs per-channel-text vs layout-locked |
| T7 | **Dataset-identity probe** (NEW, L1 evidence) | **S** — can the embedding predict its source dataset? cheap, checkpoint-only |
| T8 | Parity control | **S** — retained from v1 design |
| — | v1 overall / per-dataset / ablation / native-rate / scaling tables | **D** (five tables) |
| — | Severe-OOD table | **D** |

## 5. Figures — inventory and disposition

| figure | disposition |
|---|---|
| `fig_hook.png` | keep (conceptually still right) |
| `fig_overview.png` | **R** — must show two phases, not single-stage alignment |
| `fig_encoder.png` | **R** — filterbank + factored conditioning |
| `fig_alignment.png` | **D** — soft-contrastive alignment is gone |
| `fig_dataset_scatter.pdf` | **R** — 12/7 datasets |
| `fig_dataset_discrepancy.pdf` | audit usage; likely **R** |
| `fig_embedding_umap.png` | **D** or **R** — v1 "epoch 100" artifact |
| `fig_patch_sensitivity.pdf` | **R** or **D** |
| `fig_native_ablation.pdf` | **R** or **D** |
| `demo_setting.png`, `demo_performance.png` | keep (iOS demo) |
| **NEW** `fig_label_geometry` | **W NOW** — sensor-near vs sensor-far label pairs against text cosine; prototype measured, needs plotting |
| **NEW** `fig_label_efficiency` | **S** — the k-curve, headline figure |

---

## 6. Open decisions (need the advisor / Alex)

1. **Streaming.** Currently sold in abstract/§1/§3/§4 as "a capability no prior HAR model offers",
   but unfunded by the current plan. *Recommendation:* demote to a design affordance (one paragraph
   in §5, no claim in the abstract) for this submission. Alternative: fund the causal-training and
   equivalence experiments.
2. **iOS demo.** Keep the v1 demo as-is, or re-shoot against the evidence engine (enrollment is a
   much better demo: user demonstrates an activity 10×, app recognises it).
3. **Prior-art sweep on few-shot / query-by-example HAR is still open** (POSITIONING §10 records it
   as never completed). L3 lives closest to that literature; §2 cannot be finalised until it closes.

---

## 7. Order of work

- **Wave 1 (now, zero experiments):** abstract ✔drafted, §1, §3 (+ the geometry figure), §4, §7,
  §2 light. → the advisor can read a coherent story end-to-end.
- **Wave 2 (now):** §5 design rewrite — everything here is implemented, so it is writable even
  though details may shift.
- **Wave 3 (now):** §6 hygiene — banner, delete the five v1 tables + severe-OOD, rebuild T1/T2,
  insert skeletons T3--T8.
- **Wave 4 (after experiments):** fill T3--T8, regenerate figures, unwrap `\STALE`, write the
  results narrative.

**Guardrail (unchanged from v2):** nothing in Waves 1--3 states a performance number that has not
been measured on the current model. Every claim is a design/positioning statement or is explicitly
marked. Never delete a stale number without marking it — co-authors need the reference.

**Compile check after every wave** (`make` in `paper/`), so the advisor always gets a building PDF.
