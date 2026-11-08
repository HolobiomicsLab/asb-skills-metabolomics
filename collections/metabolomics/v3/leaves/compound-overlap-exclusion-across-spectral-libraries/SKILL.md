---
name: compound-overlap-exclusion-across-spectral-libraries
description: Use when evaluating a spectral-prediction model on an external public
  library (e.g., MoNA Q-TOF) while the model was trained on licensed libraries (Agilent
  PCDL/NIST20) and train/test overlap must be eliminated.
when_to_use_negative:
- When the model was trained only on public data that is already disjoint from the
  evaluation library, so no overlap exclusion is needed
- When the goal is training-data curation rather than building a leakage-free external
  evaluation set
- When input structures lack InChIKeys or fall outside the model's supported atom
  types (C, O, N, H, P, S, F, Cl, B, Br, I) or precursor types
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: molnetpack
  role: 'Runs prediction and evaluation on the disjoint test half: MolNet.pred_msms
    with --instrument qtof writes MGF outputs, and MolNet.evaluate computes cosine
    similarities against the held-out spectra'
  repo: https://github.com/JosieHong/3DMolMS
- name: RDKit
  role: Derives the non-stereochemical InChIKey skeleton used as the compound-level
    split unit
- name: Pandas
  role: Tabulates average cosine similarities per ion mode in the benchmark CSV (e.g.,
    fig6_benchmark.csv)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:36fc26d62db48ee9811de1fae76e7b63a839a7e264fef500794c4f53fed0003b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - NumPy
  - Pandas
  - PyTorch
  - RDKit
  - molnetpack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# compound-overlap-exclusion-across-spectral-libraries

## Summary

Removes compounds from a public MS/MS evaluation set (e.g., the MoNA Q-TOF subset) that overlap licensed spectral libraries (Agilent PCDL/NIST20), so held-out test spectra are guaranteed disjoint from training data when measuring cross-instrument generalization.

## When to use

Use when evaluating a spectral-prediction model on an external public library (e.g., MoNA Q-TOF) while the model was trained on licensed libraries (Agilent PCDL/NIST20) and train/test overlap must be eliminated.

## When NOT to use

- When the model was trained only on public data that is already disjoint from the evaluation library, so no overlap exclusion is needed
- When the goal is training-data curation rather than building a leakage-free external evaluation set
- When input structures lack InChIKeys or fall outside the model's supported atom types (C, O, N, H, P, S, F, Cl, B, Br, I) or precursor types

## Inputs

- Public MoNA Q-TOF MS/MS spectra subset (~6000 compounds) from the Fiehn Lab MoNA downloads page
- Licensed Agilent DPCL and NIST20/NIST23 library compound lists (for overlap lookup)
- Non-stereochemical InChIKey skeletons per compound (split key)

## Outputs

- Disjoint train/test compound split (50/50) of the filtered MoNA compounds
- Held-out non-Agilent test-half MGF spectra and predictions
- Per-compound cosine similarities stratified by ion mode (positive/negative), e.g., fig6_benchmark.csv

## How to apply

Download the public MoNA Q-TOF spectra subset (~6000 compounds) from the Fiehn Lab MoNA downloads page, then exclude any compound present in the licensed Agilent PCDL/NIST20 libraries so the evaluation set is disjoint from training data. Split the remaining spectra 50/50 by compound, using the non-stereochemical InChIKey skeleton as the split unit (as in scripts/build_msms_dataset.py), and hold out the non-Agilent half as the test set. The rationale is that any precursor shared between the licensed training libraries and the public test set would inflate generalization scores; skeleton-level (not full InChIKey) splitting additionally prevents stereochemistry-only duplicates from leaking across the split. After exclusion, predict spectra on the held-out half with scripts/predict.py --task msms --instrument qtof and CFM-ID 4.0, compute per-compound cosine similarities with scripts/eval.py stratified by positive and negative ion modes, and tabulate results (e.g., fig6_benchmark.csv) to check reproduction of the Fig. 6 benchmark values.

## Related tools

- **molnetpack** (Runs prediction and evaluation on the disjoint test half: MolNet.pred_msms with --instrument qtof writes MGF outputs, and MolNet.evaluate computes cosine similarities against the held-out spectra) — https://github.com/JosieHong/3DMolMS
- **RDKit** (Derives the non-stereochemical InChIKey skeleton used as the compound-level split unit)
- **Pandas** (Tabulates average cosine similarities per ion mode in the benchmark CSV (e.g., fig6_benchmark.csv))

## Examples

```
python scripts/predict.py --task msms --instrument qtof
```

## Evaluation signals

- No compound appears in both the licensed-library training set and the filtered MoNA test set (precursor-level overlap check returns zero)
- No InChIKey skeleton is shared across the 50/50 split boundary
- Average cosine similarities on the held-out half reproduce the Fig. 6 benchmark values: 0.506/0.430 (fine-tuned, pos/neg), 0.445/0.236 (from-scratch), 0.076/0.291 (CFM-ID)
- Per-compound cosine similarities are computed and stratified cleanly into positive and negative ion modes
- Results table (fig6_benchmark.csv) matches the reported values within the reported spread, e.g., 0.506(±0.316) in positive mode

## Limitations

- Held-out MoNA scores drop relative to in-library performance (fine-tuned 0.506/0.430 vs. the 0.691/0.478 reported on METLIN-SMRT-style evaluation), reflecting true cross-instrument difficulty rather than a bug
- Predicted spectra never contain peaks above the precursor m/z (no isotope envelope), which biases cosine similarity comparisons against experimental spectra containing isotopes
- Only compounds with supported atom types and precursor types can be evaluated; unsupported formats/structures are excluded during loading

## Evidence

- [other] Exclude compounds overlapping the Agilent PCDL/NIST20 licensed libraries so the evaluation set is disjoint from training data.: [evidence span withheld — anchor status: not_found]
- [other] Split the remaining spectra 50/50 by compound (using the non-stereochemical InChIKey skeleton as the split unit, as in scripts/build_msms_dataset.py): [evidence span withheld — anchor status: not_found]
- [methods] MoNA <https://mona.fiehnlab.ucdavis.edu/downloads>`_, publicly available.: "`MoNA <https://mona.fiehnlab.ucdavis.edu/downloads>`_, publicly available."
- [readme] Evaluate against ground truth
results_df = molnet_engine.evaluate(: "Evaluate against ground truth
results_df = molnet_engine.evaluate("
- [abstract] 3DMolMS model can be generalized to the prediction of MS/MS spectra acquired by different labs on different instruments through minor fine-tuning on a small set of spectra: "3DMolMS model can be generalized to the prediction of MS/MS spectra acquired by different labs on different instruments through minor fine-tuning on a small set of spectra"
