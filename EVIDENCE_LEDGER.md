# HALO paper evidence ledger

Updated 2026-08-17. A quantitative statement may enter the manuscript only when this ledger names
its source, scope, and fingerprint. `verified` means reproducible from an existing local artifact;
`sealed` means the protocol is fixed but results have not been consumed; `pending` means no
provenance-complete artifact exists.

The manuscript is pinned to the **completed sensor-granularity encoder**,
`phase_a_fixed_1s_rotation_20260817/best.pt`, step 27,000. Everything printed derives from that
checkpoint or from the corpus and code that produced it.

## Artifact identities

| Artifact | Identity |
|---|---|
| Phase-A checkpoint | `halo/training/tokenizer/outputs/phase_a_fixed_1s_rotation_20260817/best.pt`, step 27,000; fingerprint `cc71fbee9f97191a5301a2c1a75afdf9db1ad12f241779f688f6c4896815823f` |
| Phase-A run configuration | `run_config.json` in the same directory |
| Memory bank | `halo/training/evidence/outputs/memory_bank.pt`, schema 4, fingerprint `02e5425cf46ca6a0`, bound to the checkpoint above |
| Resolvability table | `halo/training/evidence/outputs/resolvability.json`, schema 2, scope `phase_a_training_subjects_only_per_sensor`, same checkpoint hash |
| Admissibility gate | **does not exist for this checkpoint** |
| Enrollment results | **do not exist for this checkpoint** |

## Verified manuscript quantities

| Claim | Status | Evidence |
|---|---|---|
| Phase-A roster: 18 datasets, 56 configurations | verified | `run_config.json::train_datasets`; count of `meta.json` under `grids/native/` |
| 1,963,606 Phase-A stream-window rows before screens/splits | verified | sum of `.labels\|length` over the 18 rostered datasets' native grids |
| External roster: 10 datasets, 14 configurations | verified | `PRIMARY_EVAL_DATASETS`, `PHASE_B_DEV_DATASETS`, `PHASE_B_TEST_DATASETS`, native grid metadata |
| Per-dataset rows and rates in Table 2 | verified | same aggregation, restricted to the rostered dataset tuples |
| Bank: 250,000 windows, 546 subjects, 56 configurations, 166/166 labels | verified | `memory_bank.pt` |
| Bank: 1,350,834 patch rows, 2,629,972 sensor rows | verified | `memory_bank.pt` |
| Phase-A coefficients: JEPA 7.053501481, VICReg 0.653232013, descriptor 0.0 | verified | `run_config.json` |
| Phase-A hyperparameters in Appendix E | verified | `run_config.json` |
| Encoder 7.70\,M parameters | verified | parameter count of the checkpoint's `encoder` state dict |
| 30,000 steps in 33 min, 4.36 GiB peak, one RTX 4090 | verified | `log.jsonl` final record |
| Codebase line counts and 578 tests in Table 1 | verified | `wc -l` per subsystem; `pytest` collection |
| 6.9 GB bank on disk | verified | `ls -l` on the schema-4 bank |

The corpus aggregation used for Table 2 is:

```sh
find halo/data/datasets -path '*/grids/native/*/meta.json' -print0 \
  | xargs -0 -n1 jq -r '[.dataset,.stream_id,.rate_hz,(.labels|length)]|@tsv'
```

Rows are then restricted to the exact dataset tuples stored in the run configuration and deployment
policy. Counts are converter outputs, not quantities attributed to the dataset publications.

## Sealed evidence

The test roster is `inclusivehar`, `usc_had`, `tnda_har`, `ut_complex`, `monipar`, `spar`, and
`upper_limb_use`. No test result appears in the paper. Do not run or add these cells until the
development criteria in the paper's pending-measurements section are met.

## Pending evidence

No recognition accuracy of any kind is currently claimed. The gate has not been fitted against the
completed encoder's memory, so no enrollment curve, control comparison, or gate telemetry is
attributable to the system the paper specifies.

Quantities removed from earlier drafts because no checked-in generator and raw output could be found,
or because their producing artifact was superseded:

- the development enrollment table and its control columns, and the gate range and vetoed-row
  telemetry that accompanied it — produced by a gate artifact the current loader refuses;
- the step-4,000 checkpoint selection score;
- rate-equivalence cosines `0.997` and `0.996`;
- gravity/posture cosines `0.84--0.88`, `-0.04`, and `0.99`;
- the DFT-size cosine `1.000000`;
- the 14-pair/49-comparison/38-reversal text-geometry claim;
- tokenizer and objective ablation result cells;
- baseline result cells;
- cross-configuration result cells; and
- Core ML, iOS, latency, memory, and deployment-case-study claims.

Reintroducing any item requires a command, source-data manifest, raw JSON/CSV, seed, software
revision, checkpoint fingerprint where applicable, and an entry in this ledger.

## Available but not yet in the manuscript

The resolvability artifact contains a paired-contrast measurement over five datasets whose streams
record the same sessions simultaneously. It is provenance-complete against the current checkpoint and
would support the design's central claim that placement quality inverts across concepts. It is not
printed anywhere yet; adding it requires an entry here and a figure.
