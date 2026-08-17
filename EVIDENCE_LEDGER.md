# HALO paper evidence ledger

Updated 2026-08-17. A quantitative statement may enter the manuscript only when this ledger names
its source, scope, and fingerprint. `verified` means reproducible from an existing local artifact;
`development-only` means it may guide design but not support an external-performance claim;
`sealed` means the protocol is fixed but results have not been consumed; `pending` means no
provenance-complete artifact exists.

## Repository divergence (added 2026-08-17)

**The manuscript is pinned to a superseded artifact.** Every quantity below still reproduces from the
artifact it names, so nothing here is wrong — but `halo/` has moved to a different checkpoint,
corpus, and Phase-A recipe. A reader who opens the code will not find the run described here.

| | Manuscript | Repository head |
|---|---|---|
| Phase-A checkpoint | `phase_a_sensor_v1_20260813_v2/best.pt`, step 4,000 | `phase_a_fixed_1s_rotation_20260817/best.pt`, step 27,000 |
| Patch grid | two resolutions (0.4–0.8 / 1.0–1.5 s) | single fixed 1.0 s |
| Descriptor objective | coefficient 0.5, active | 0.0, disabled |
| Augmentations | full stack; rotation shared across views | rotation only at p=1, drawn independently per view |
| Loss coefficients | JEPA 3.998090 / VICReg 0.689099 | JEPA 7.053501 / VICReg 0.653232 |
| Native grids | 1,871,011 Phase-A rows, 161 labels | 1,963,606 rows (final partial contexts retained; KU-HAR reconverted), 166 labels |
| Bank | 7,653,242 sensor rows | 2,629,972 sensor rows, rebuilt 2026-08-17 09:53 |
| Gate artifact | `artifact_version: 2` | loader requires `3` — the reported gate can no longer be loaded |

The newer checkpoint has a bank and a train-only resolvability table but **no fitted gate and no
enrollment run**, so the results table cannot be repinned yet. Repinning requires, in order:
settle the veto/neutral calibration, `gate_predictor --fit --bank`, add at least one
cross-configuration development cell, then re-run development enrollment. `STALENESS.md` holds the
full audit and the per-dataset corpus deltas.

## Artifact identities

| Artifact | Identity |
|---|---|
| Phase-A run configuration | `halo/training/tokenizer/outputs/phase_a_sensor_v1_20260813_v2/run_config.json`; SHA-256 `6913828d8a25a7c62cdafbf4c45d68ea6a4800f7a8b91099826710728f3f0a0b` |
| Selected checkpoint | `best.pt`, step 4,000; fingerprint `35f676127e739ec9ba2cb1d226815f17a80e2f2da14f836bb6adac9e9c56e06e` |
| Memory-bank build | `halo/training/evidence/outputs/phase_a_checkpoint_selection_20260816/step4000/build_memory.log`; SHA-256 `a24a0ba3a6517f2068bc22eb9769f8cfc4cd74b49a533bcd555bd34e3194b7df` |
| Memory bank | fingerprint `3084dae7d8d289ac`; checkpoint-bound schema-4 artifact |
| Admissibility gate | fingerprint `63fdc553dc643788be9c8efe8f7a81ddb03c5da61b75ced5b00331a492bcf735` |
| Development evaluation | `eval_dev_coherent.json`; SHA-256 `8083ea0dea02fc759cc22710afa28b61257629b56f250af497f85531aabed127` |
| Evaluation protocol | fingerprint `d43e80a22ec24d27c06cd9f30aefc1c6aeab48484ed04720dcda7f0d7132866c`; source fingerprint `ca0b64d2e5b1472ea9a29d6239e483ed593effe2f8a08aa44d57822cfc9dd421` |

## Verified manuscript quantities

| Claim | Status | Evidence |
|---|---|---|
| Phase-A roster: 18 datasets | verified | `run_config.json::train_datasets` |
| Materialized Phase-A roster: 56 configurations and 1,871,011 stream-window rows before screens/splits | verified | Sum of `.labels|length` and count of `meta.json` files for the 18 named datasets under `halo/data/datasets/*/grids/native/*/meta.json` |
| External roster: 10 datasets and 14 configurations | verified | `PRIMARY_EVAL_DATASETS`, `PHASE_B_DEV_DATASETS`, `PHASE_B_TEST_DATASETS`, and native grid metadata |
| Bank: 250,000 windows, 546 subjects, 56 configurations, 161/161 labels | verified | `build_memory.log` |
| Bank rows: 3,942,901 patch rows and 7,653,242 sensor rows | verified | `build_memory.log` |
| Phase-A coefficients: JEPA 3.9980903229, VICReg 0.6890990585, descriptor 0.5 | verified | `run_config.json` |
| Step 4,000 selection score 0.3028457; step 30,000 score 0.2831909 | verified | `exploratory_transfer_includes_sealed_step4000.json` and `exploratory_transfer_includes_sealed_step30000.json`; selection itself used only the internal Phase-A validation metric |
| Development macro-F1 curve printed in Table 3 | development-only | Dataset-level arithmetic means from `eval_dev_coherent.json`, datasets `motionsense`, `realworld`, `shoaib`, seed 20260808 |
| Gate range 0.245--0.733 and zero vetoed rows | development-only | Min/max/veto telemetry across all entries in `eval_dev_coherent.json` |

The corpus aggregation used for the paper table is:

```sh
find halo/data/datasets -path '*/grids/native/*/meta.json' -print0 \
  | xargs -0 -n1 jq -r '[.dataset,.stream_id,.rate_hz,(.labels|length)]|@tsv'
```

Rows are then restricted to the exact dataset tuples stored in the run configuration and deployment
policy. Counts are converter outputs, not quantities attributed to the dataset publications.

## Sealed evidence

The test roster is `inclusivehar`, `usc_had`, `tnda_har`, `ut_complex`, `monipar`, `spar`, and
`upper_limb_use`. No test result is printed in the paper. Do not run or add these cells until the
development gate beats or is replaced by the predefined ungated, prototype, and ridge controls.

## Pending evidence

The following previous-draft quantities were removed because no checked-in generator and raw output
could be found:

- rate-equivalence cosines `0.997` and `0.996`;
- gravity/posture cosines `0.84--0.88`, `-0.04`, and `0.99`;
- DFT-size cosine `1.000000`;
- the 14-pair/49-comparison/38-reversal text-geometry claim;
- tokenizer and objective ablation result cells;
- baseline result cells;
- cross-configuration result cells; and
- Core ML, iOS, latency, memory, and deployment-case-study claims.

Reintroducing any item requires a command, source-data manifest, raw JSON/CSV, seed, software
revision, checkpoint fingerprint where applicable, and an entry in this ledger.
