---
name: tabular-result-verification
description: Decision support for validating parameter-tuning tables when a grid search
  over assignment radii and detection thresholds has produced F1 scores per MCF setup
  and reported optima must be cross-checked.
when_to_use_negative:
- Input is a single fitted result (one r, one theta) rather than a scanned grid —
  there is no landscape to verify optima against.
- The metric is compound concentration reconstruction accuracy rather than F1 — the
  reported optima and degradation patterns are specific to the F1 classification task
  on 2D 1H,13C HSQC artificial mixtures.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3337
- http://edamontology.org/topic_3570
tools:
- name: pandas
  role: Assembles per-setup optima and stability summaries into a CSV table
- name: mcfNMR
  role: Source of the scanned F1 landscape; its configuration exposes assignment_radius
    and detection_threshold as the swept parameters
  repo: https://github.com/GeoMetabolomics-ICBM/mcfNMR
- name: SciPy
  role: Underlying linear-optimization backend (via HiGHS interface) that produced
    the F1 scores being verified
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c01652
    title: Network Flow Methods for NMR-Based Compound Identification
  source_content_hash: sha256:63c1fa064a2edb5592467455ed76f79d5a93092e90dbfb2be395278dfcc2dcc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GeoMetabolomics-ICBM/mcfNMR
  tools:
  - SciPy
  - mcfNMR
  - pandas
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3337
  - http://edamontology.org/topic_3570
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tabular-result-verification

## Summary

Assembles a per-setup table of optimal (assignment radius r, detection threshold theta) pairs and stability summaries from a scanned F1 grid, verifying that each row reproduces the reported optimum (e.g. setup C: r=0.08, theta=0.003, F1=0.83).

## When to use

Decision support for validating parameter-tuning tables when a grid search over assignment radii and detection thresholds has produced F1 scores per MCF setup and reported optima must be cross-checked.

## When NOT to use

- Input is a single fitted result (one r, one theta) rather than a scanned grid — there is no landscape to verify optima against.
- The metric is compound concentration reconstruction accuracy rather than F1 — the reported optima and degradation patterns are specific to the F1 classification task on 2D 1H,13C HSQC artificial mixtures.

## Inputs

- F1-score grid (CSV) with columns for assignment radius (r), detection threshold (theta), and F1 score per MCF setup (A, B, C, D)
- Reported optimal (r, theta) pairs to check against (e.g. C: 0.08/0.003; A: 0.06/0.003; B: 0.05/0.003; D: 0.06/0.005)

## Outputs

- CSV table of per-setup optimal (r, theta) pairs and maximum F1 scores
- Stability summaries describing each setup's F1 landscape (e.g. flat high-F1 plateau for setup C vs. sharp degradation for setup B)

## How to apply

Load the F1 grid produced by the earlier (r, theta) scan — the CSV written by pandas that maps each assignment radius and detection threshold to an F1 score for each MCF setup (A: single pass, B and D: single pass variants, C: incremental assignment). Isolate each setup's rows and locate the argmax to recover its optimal (r, theta) pair. Verify the reported optima: C at r=0.08/theta=0.003 (F1=0.83), A at 0.06/0.003, B at 0.05/0.003, and D at 0.06/0.005. Quantify sensitivity by computing how each setup's F1 degrades as its radius deviates from the optimum — expected behaviour is flat, high F1 for the incremental setup C across a wide range of increasing assignment radii, whereas setup B's F1 drops sharply away from r=0.05. Assemble the per-setup optima and stability summaries into a fresh pandas DataFrame and re-export as CSV. Rationale: the article notes that no definite protocol exists for choosing r, cabs, and theta, so robustness of the chosen combination — not just the peak F1 — is the meaningful verification target.

## Related tools

- **pandas** (Assembles per-setup optima and stability summaries into a CSV table)
- **mcfNMR** (Source of the scanned F1 landscape; its configuration exposes assignment_radius and detection_threshold as the swept parameters) — https://github.com/GeoMetabolomics-ICBM/mcfNMR
- **SciPy** (Underlying linear-optimization backend (via HiGHS interface) that produced the F1 scores being verified)

## Examples

```
python -m mcfnmr -c <config_file>
```

## Evaluation signals

- The assembled table must reproduce the reported optima exactly: C: r=0.08, theta=0.003; A: r=0.06, theta=0.003; B: r=0.05, theta=0.003; D: r=0.06, theta=0.005.
- Setup C's F1 scores must show a flat, high plateau across increasing assignment radii — no sharp drop-off comparable to setup B's.
- The maximum F1 per setup row should match the article's reported averages (C=0.83, A=0.81, B=0.80, D=0.78).
- Argmax over each setup's (r, theta) rows must equal the reported optimum; any discrepancy indicates the scan grid or CSV parsing is wrong.
- Output CSV must contain one row per MCF setup (A, B, C, D) with columns for r, theta, max F1, and a stability descriptor.

## Limitations

- The article states there is no definite answer or protocol to choose parameter values (assignment radius r, absorption cost, detection threshold ϑ) — verification is against empirically reported optima, not a principled rule.
- The F1 grid must come from a prior scan on the same library (e.g. 501-compound MetaboMiner library) and mixture set; optima may shift with a different library or mixture composition.
- Observing the dependency of detections in a blank spectrum on the threshold value is suggested as a way to choose theta, but this skill only cross-checks the tabulated optima — it does not derive them.

## Evidence

- [full_text] optimal (r, theta) pair per setup: [evidence span withheld — anchor status: not_found]
- [discussion] parameter choice protocol: "To date, we have no definite answer or protocol to choose these values"
- [readme] README configuration: "`assignment_radius`: Assignment radius: Single float (if incremental_fit==false) or a sequence of floats (if incremental_fit==true)."
