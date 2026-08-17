# MobiCom submission requirements — cached reference

Fetched 2026-08-17 from the official CFPs:
- MobiCom 2027 (our target): <https://www.sigmobile.org/mobicom/2027/cfp.html>
- MobiCom 2026 (cross-checked; formatting rules identical): <https://www.sigmobile.org/mobicom/2026/cfp.html>

Program co-chairs (2027): Joerg Widmer, Nirupam Roy — `mobicom27.pc.chairs@gmail.com`

---

## 1. Hard formatting rules (violations are desk rejections)

| Rule | Requirement | Our status |
|---|---|---|
| Body length | **≤ 12 single-spaced numbered pages**, *including figures, tables, and any other material* | ✅ ~7.5 pages — **4.5 pages of headroom** |
| References | As many pages as needed, **do not count** toward the 12 | ✅ pages 8–11 |
| Appendices | Allowed **after** the bibliography, do not count. "Reviewers are not required to read appendices." | ✅ pages 12–14 |
| Font size | **No smaller than 10 pt** | ✅ `10pt` class option |
| Columns | Double column, each **9.25 in × 3.33 in**, **0.33 in** gutter, **≤ 55 lines of text per column** | ✅ enforced by `acmart`; do not load `geometry` or change `\baselinestretch` |
| Paper size | US Letter, 8.5 in × 11 in | ✅ |
| Template | LaTeX: `\documentclass[sigconf,10pt]{acmart}` | ✅ exact match |
| Format | PDF only, Adobe Acrobat (English) compatible. Postscript and MS-Word rejected. | ✅ |
| File size | **< 15 MB** | ✅ ~0.4 MB |

## 2. Double-blind rules

- "Authors' names must **not** appear anywhere in the paper or in the PDF file." No affiliations, no funding acknowledgments.
- **"The PDF file must also not contain any embedded hyperlinks, as these may compromise author anonymity."**
  → `acmart` loads `hyperref`, which emits link annotations by default. **Suppress with `\hypersetup{draft}`** and verify the built PDF has zero `/Link` and `/URI` objects.
- Self-citation in third person: write "In prior work, Smith [3] presented…", never "In prior work [3], we presented…".
- Never write "[3] Reference deleted for double-blind review".
- Supplemental/code links only if anonymized, and "must not be used to provide additional text that does not fit within the 12-page limit".
- Non-anonymized preprints are allowed, but the submission must not reference them, and do not advertise the preprint on social media or in the press while under review.

## 3. Desk-rejection triggers

- Placeholder title or abstract ("TBD" or similar).
- Non-bibliographic content beyond 12 pages.
- Any formatting non-compliance (checked by manual inspection).
- Double-blind violations.
- Non-PDF submission.
- ACM Publications Policy violations.

## 4. Review criteria

Papers are judged on **originality, technical soundness, significance, rigor of evaluation, and impact**.

The CFP explicitly asks for work that "address[es] important research challenges and **build[s] practical working systems**", via "rigorous analysis, system design, prototyping, experimental evaluations and real-world measurement study". It also rewards papers that "highlight both the significance **and limitations**" of the work, and that describe how authors will release "codebases, well-documented datasets, modeling and/or simulation tools".

## 5. Deadlines (MobiCom 2027)

| Milestone | Date |
|---|---|
| Abstract registration | 2026-08-26, 23:59 AoE |
| Paper submission | 2026-09-02, 23:59 AoE |
| Early reject notification | 2026-10-23 |
| Rebuttal period | 2026-11-03 – 11-06 |
| Acceptance notification | 2026-11-19 |

No deadline extensions.

## 6. Process rules worth knowing in advance

- **Rebuttal: max 500 words.** Only (a) correcting factual errors in reviews or (b) answering reviewer questions. "Responses must not include new experiments, new data, or new figures, describe additional work completed since submission or promise additional work to follow."
- **One-shot revision:** at most three major changes/clarifications specified by reviewers; the paper stays under review at MobiCom and cannot go elsewhere without withdrawing.
- **Shepherding:** 4–6 weeks, anonymous; may require new experimental results.
- **Prior submission disclosure:** *required* for MobiCom one-shot revisions and recent MobiCom rejections — report the previous reviewers' major concerns and how they were addressed. *Strongly encouraged* for rejections from SIGCOMM, NSDI, MobiSys, SenSys, UbiComp, etc.
  → **Applies to us**: this work was rejected from MobiCom '26, so a disclosure summary is required at submission.
- Cannot resubmit a rejected paper until 11 months have passed.
- No simultaneous submissions; double submissions are immediately rejected from all venues involved. arXiv/tech reports are *not* concurrent submissions.
- Artifact Evaluation is voluntary, post-acceptance, and does not influence acceptance.
- Authors must comply with the ACM Publications Policy on Research Involving Human Participants and Subjects.
- **AI/LLM:** MobiCom follows the ACM Policy on Authorship, including its guidance on AI use. (Note the mirror-image rule for reviewers: submissions are confidential and may not be uploaded to language models.)
- All authors declared at submission; ORCID required before publication; ≥1 author must register and present; 90-second teaser video with camera-ready.

## 7. Solicited topics (where we fit)

Our work sits under **"Physical and embodied AI for mobile systems"** (world models, foundation models), **"Applications of machine learning to mobile/wireless systems"** (on-device ML, transfer learning), and **"Human-centric interactive systems"** (mobile health, wearables). Also relevant: "Measurements and deployment (operational networks, datasets, benchmarks)".

---

## 8. Structural conventions observed in accepted papers

Sampled to calibrate our own structure (2026-08-17):

**M4, "Mobile Foundation Model as Firmware" (MobiCom '24)** — 15 figures, 5 tables
`Intro → Background and Motivation → Design and Prototyping → Experiments and Analysis → Related Work → Limitations and Future Work → Conclusions`
Related work sits **late**, after the evaluation. Implementation detail is folded into the experimental setup subsection. Evaluation covers accuracy, latency, memory, energy, storage, and operator complexity.

**SKELAR (SenSys '25)** — `Intro → Related Work → Method → Datasets → Experiments → Discussion and Followup Challenges → Summary and Conclusions`
Related work **early**; an explicit discussion-of-limitations section before the conclusion.

**Takeaways for us:**
1. Either Related Work position is defensible; placing it late gets the design in front of a skimming reviewer sooner.
2. A **Background/Motivation** section is expected, and at MobiCom it usually carries a measurement or preliminary study.
3. An **Implementation** section (or a substantial implementation subsection) is expected — SIGCOMM's author guide notes that systems papers are commonly rejected "because reviewers feel key details of the system are missing".
4. A **Limitations/Discussion** section is standard and, per the CFP's review criteria, actively rewarded.
5. Accepted papers are **figure-dense** (M4: 15 figures). Three figures reads as thin for a 12-page slot.
6. Contributions are stated as a **bulleted list** of "We design… / We build… / We evaluate…".

## 9. Tone conventions observed

- **Active voice with "we"** dominates the contribution and design text: "We delineate a vision…", "We design and prototype…", "We have constructed…".
- **Present tense** for system description ("HALO computes…", not "HALO computed…"); past tense reserved for what was measured.
- Contributions are asserted, not hedged — no "we believe", "we hope to show", "may potentially".
- Background paragraphs tolerate more passive/impersonal construction; design and evaluation sections should not.
- Short declarative sentences. The problem statement lands in the first two paragraphs, not the fourth.

---

## 10. Pre-submission checklist

- [ ] Body ≤ 12 pages; verify from the build log, not by eyeballing the PDF.
- [ ] `\hypersetup{draft}` present; built PDF has **zero** `/Link` and `/URI` objects.
- [ ] No author names, affiliations, or acknowledgments anywhere in the source or PDF metadata.
- [ ] All self-citations phrased in third person.
- [ ] Title and abstract are real (not placeholders) — including no leftover `\TODO{}` markers.
- [ ] File < 15 MB, PDF opens in Acrobat.
- [ ] Prior-submission disclosure prepared (required: MobiCom '26 rejection).
- [ ] No `geometry` package, no `\baselinestretch` change.

Verify the hyperlink rule with:

```sh
python3 - <<'EOF'
import zlib, re
d = open('0_main.pdf','rb').read()
link = uri = 0
for m in re.finditer(rb'stream\r?\n', d):
    s = m.end(); e = d.find(b'endstream', s)
    try: t = zlib.decompress(d[s:e])
    except Exception: continue
    link += t.count(b'/Link'); uri += t.count(b'/URI')
print('/Link', link, '/URI', uri)   # both must be 0
EOF
```
