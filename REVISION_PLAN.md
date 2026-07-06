# HALO resubmission — paper revision plan (framing now, numbers after retrain)

Two treatments, kept strictly separate:
- **REWRITE (framing/posturing only, safe now — NO unmeasured numbers):** align the thesis with the
  new direction (single-stage, physical-Hz filterbank tokenizer, principled conditioning, streaming,
  honest eval). Touches: abstract, §1 intro, §2 related, §3 motivation, §4 overview, §7 conclusion.
- **MARK OUTDATED (do NOT rewrite — flag, keep as reference until the M4 retrain):** every measured
  number, every results table, and the *method text that is being replaced but not yet trained/measured*
  (old tokenizer, two-stage SSL, sinusoidal PE). Touches: all of §6 experiments + the numeric claims
  embedded in §5 design + the abstract/intro results sentences.

## Marking mechanism (add to 0_main.tex preamble)
```latex
\usepackage{soul}
\newcommand{\OUTDATED}[1]{\todo[inline,color=orange!40]{OUTDATED — pre-resubmission (v1) result / method; pending M4 retrain. #1}}
\newcommand{\STALE}[1]{\textcolor{orange}{#1}}   % inline wrap for a single stale number/phrase
```
Rule: never delete a stale number (co-authors need the reference) — wrap it in `\STALE{}` and drop an
`\OUTDATED{}` note. Put one red banner at the top of §6.

## What changed in the model/data/eval (the source of every edit)
1. **Single-stage, not two-stage.** Stage-1 SSL (MAE + patch-contrastive) was deleted; the encoder is
   trained from scratch during language alignment. → kill all "Stage 1 / Stage 2" scaffolding.
2. **Physical-Hz filterbank tokenizer**, not adaptive-pooling + spectral-temporal CNN. Rate-invariant &
   anti-aliased *by construction* (fixed constant-Q Gaussian filterbank on a native-rate rDFT). Stronger
   story than adaptive pooling. (docs/v2/design_tokenizer.md)
3. **RoPE over physical time**, not sinusoidal timestep-index PE. This is literally the fix for the flaw
   §3 already criticizes ("timestep-index PE conflates physical time with sequence length").
4. **Streaming — entirely new.** One weight set serves offline session recognition AND low-latency
   on-device streaming (RoPE physical-time + causal/windowed masks + train-both + offline→online
   distillation; streaming-equivalence verified). Not in the paper at all. (research_streaming_design.md)
5. **Principled conditioning split.** Placement/gravity stay in language (still the largest ablation
   contributor); sampling rate + window duration go NUMERICALLY into the tokenizer + Nyquist/resolution
   masks and are REMOVED from the text (frozen-SBERT numeracy is broken). (research_conditioning.md)
6. **Unified augmentation** (signal + physics + label-text + channel-text phrase + channel-text dropout,
   all in one config; measured bug-free).
7. **Honest eval:** subject-disjoint ZS-XD vs each dataset's own labels, macro-F1 primary, ConSE bridge,
   parity rows. The old open-set-over-87-labels + severe-OOD numbers were leakage-inflated.
8. **Corpus/counts:** 11 train (added CAPTURE-24) / 6 test (opportunity → appendix; VTT-ConIoT dropped);
   ~94 labels not 87. HARTH is now a regular test set, not "severe-OOD."

## Per-section plan

| Section | Current | Change | Treatment |
|---|---|---|---|
| **Abstract** | "two-stage framework"; adaptive-pooling; "beats 5 baselines on 8 metrics, +13.7% (42.0 vs 28.3), 35M vs MOMENT 341M" | single-stage language-aligned model; contributions = filterbank tokenizer + principled conditioning + **one model for offline + streaming** + honest benchmark | REWRITE prose; `\STALE{}`+`\OUTDATED{}` every number; keep video link |
| **§1 Introduction** | two-stage (L27-28); adaptive-pooling; R1-R3; contributions bullets; results para (L31) | single-stage; filterbank; add streaming/real-time as a capability (+ maybe R4); rewrite bullets 2-3 (drop two-stage, add filterbank + streaming + honest benchmark) | REWRITE framing; `\OUTDATED` the L31 results para |
| **§2 Related Work** | positions vs MOMENT/LanHAR/LLaSA (being dropped as baselines) | reposition vs **UniMTS + ssl-wearables** (new baselines) + streaming-ASR unified-model lineage + anisotropy/whitening lit | REWRITE (light) |
| **§3 Motivation** | heterogeneity + generalization; 17/14 datasets; VTT-ConIoT severe-OOD example; criticizes timestep-index PE | keep the two challenges; update counts; drop VTT-ConIoT; **add a real-time/streaming motivation** (real phone/watch use is online, not just whole-session); the PE critique now sets up RoPE-physical-time | REWRITE framing + counts |
| **§4 Overview** | two-stage pipeline; 35M split; inference-only | single-stage; filterbank; add streaming (offline mask vs causal-window+KV-cache); param split marked pending | REWRITE; `\STALE` the 35M split |
| **§5 Design** | §Adaptive-pooling tok; §Spectral-temporal CNN; sinusoidal PE; §Self-supervised Pretraining (Stage 1); §Conditioning (rate in text); §Synonym label-aug; §Soft contrastive (z-score = the "mean-subtraction") | old tokenizer/PE/Stage-1 **superseded but not yet retrained** → mark outdated + one-line forward-pointer to the new design docs; conditioning + augmentation framing UPDATED; add a **streaming design subsection stub** (design-level, results pending) | MARK OUTDATED (old methods + embedded numbers −19.6/−7.0/9.5pp/76.7%/1.3pp); UPDATE conditioning + augmentation framing; optional new-design stubs |
| **§6 Experiments** | 10 tables of measured results on the v1 model | red banner "all results reflect v1 model+eval, being re-measured after M4 — do not cite"; UPDATE the *protocol* prose (honest ZS-XD, subject-disjoint, ConSE, parity) since it's decided; UPDATE dataset table (11/6) + baseline table set; REMOVE the severe-OOD table/section | MARK OUTDATED all numbers; UPDATE protocol + dataset/baseline framing |
| **§7 Conclusion** | "beats 5 baselines on 8 metrics"; future work = per-sample cond / multi-prototype / UDA | drop the unmeasured claim; future work → streaming/dense supervision/free-living data; per-sample conditioning partly done | REWRITE; `\OUTDATED` the 9.9pp RealWorld number |
| **reference.bib** | — | add: anti-aliasing/filterbank, RoPE/RoFormer, Dual-mode ASR + U2/U2++, anisotropy/BERT-whitening/SimCSE, UniMTS, ssl-wearables | ADD (safe housekeeping) |

## Phasing (what to do when)
- **Phase 1 (now, mechanical + honest):** add the `\OUTDATED`/`\STALE` macros + the §6 banner; wrap every
  stale number + the two-stage/old-tokenizer/severe-OOD content. Makes the draft internally honest
  immediately, deletes nothing.
- **Phase 2 (now, framing):** rewrite abstract/intro/motivation/overview/related/conclusion posturing to
  the new thesis. No numbers.
- **Phase 3 (now-ish, design framing):** mark §5 old methods outdated + forward-point; update the
  conditioning + augmentation subsections; add short design-level stubs for the filterbank + RoPE +
  streaming (design finalized, results pending).
- **Phase 4 (AFTER the M4 retrain):** fill the new numbers, unwrap `\STALE`, finalize §6 tables + the
  ablations, and the abstract/intro/conclusion headline results.

## Guardrail
Nothing in Phases 1-3 states a performance number that hasn't been measured on the new model. Every
claim is either (a) a design/positioning statement, or (b) explicitly marked OUTDATED/pending. The
new headline numbers only appear in Phase 4, after M4.
