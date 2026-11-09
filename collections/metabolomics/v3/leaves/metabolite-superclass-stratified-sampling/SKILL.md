---
name: metabolite-superclass-stratified-sampling
description: Use when benchmarking in silico spectral prediction or identification
  tools across metabolomics domains (exposomics, foodomics, human metabolomics) and
  you need unbiased, domain-representative test sets drawn from a larger spectral
  database.
when_to_use_negative:
- When you need unbiased coverage of the full Metlin dataset rather than domain-matched
  subsets (e.g. overall benchmarking of a new predictor).
- When superclass (ClassyFire) annotations are unavailable or unreliable for your
  reference database, since stratification depends entirely on them.
- When evaluating on a single chemical class only — stratification is unnecessary
  and adds sampling variance.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0157
tools:
- name: CFM-ID 4.0
  role: Performs the actual QTOF ESI-MS/MS spectral prediction on each stratified
    subset so per-domain scores can be computed
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Provides the superclass taxonomy used to define the reference distributions
    (HMDB/FooDB/ContaminantDB) and to assign superclasses to Metlin molecules for
    stratified sampling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:d1a956f3feae19ca7cfd88952e1966e8158842ac5eec0370ef73002ecc327c12
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolite-superclass-stratified-sampling

## Summary

Builds domain-representative evaluation subsets of metabolites by sampling molecules so each subset matches the superclass (ClassyFire) distribution of a reference database (HMDB, FooDB, ContaminantDB), enabling fair per-domain comparison of spectral prediction methods.

## When to use

Use when benchmarking in silico spectral prediction or identification tools across metabolomics domains (exposomics, foodomics, human metabolomics) and you need unbiased, domain-representative test sets drawn from a larger spectral database.

## When NOT to use

- When you need unbiased coverage of the full Metlin dataset rather than domain-matched subsets (e.g. overall benchmarking of a new predictor).
- When superclass (ClassyFire) annotations are unavailable or unreliable for your reference database, since stratification depends entirely on them.
- When evaluating on a single chemical class only — stratification is unnecessary and adds sampling variance.

## Inputs

- Metlin 2019 QTOF ESI-MS/MS spectral dataset (SMILES + spectra at 10/20/40 eV, [M+H]+ and [M-H]-)
- ClassyFire superclass annotations for HMDB, FooDB, and ContaminantDB compounds
- Target domain definitions (human metabolomics, foodomics, exposomics) mapped to reference databases

## Outputs

- Three superclass-stratified evaluation subsets matching HMDB, FooDB, and ContaminantDB superclass distributions
- Per-domain performance tables (e.g. Dice scores) comparing predictor variants across [M+H]+ and [M-H]- modes
- Optional per-domain detail plots analogous to Supplemental Figures S1-S6

## How to apply

For each target domain, retrieve the superclass distribution (ClassyFire terms) of compounds in the reference database — HMDB for human metabolomics, FooDB for foodomics, ContaminantDB for exposomics. From the source spectral database (Metlin 2019 in this study), sample molecules so that the sampled subset's superclass composition matches that reference distribution, producing three disjoint evaluation sets. This controls for chemical-class composition so performance differences reflect the domain mixture rather than class imbalance. Predictions are then made at 10, 20, and 40 eV collision energies in both [M+H]+ and [M-H]- modes, and scored per domain (e.g. Dice coefficient with peak-matching tolerance of 0.01 Da or 10 ppm, whichever occurs first) to detect domain-specific strengths or weaknesses of a predictor.

## Related tools

- **CFM-ID 4.0** (Performs the actual QTOF ESI-MS/MS spectral prediction on each stratified subset so per-domain scores can be computed) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Provides the superclass taxonomy used to define the reference distributions (HMDB/FooDB/ContaminantDB) and to assign superclasses to Metlin molecules for stratified sampling)

## Examples

```
# stratify Metlin molecules by superclass to match FooDB distribution, then predict and score
python stratify_and_score.py --source metlin2019.smi --ref fooddb_superclasses.tsv --tool cfmid4 --modes pos,neg --energies 10,20,40
```

## Evaluation signals

- Sampled subset's superclass distribution matches the target database's distribution within a small tolerance (check per-superclass proportions).
- Each domain subset yields non-trivial counts in both ion modes and all three collision energies (10, 20, 40 eV).
- Per-domain Dice scores reproduce the expected qualitative pattern: MSML+MSRB outperforms MSML alone on foodomics and metabolomics domains, with no improvement on exposomics.
- Peak-match tolerance enforced consistently: m/z difference < 0.01 Da or 10 ppm (whichever occurs first) when comparing predicted vs. experimental spectra.

## Limitations

- Stratification is only as good as the ClassyFire superclass assignments; misannotation of a compound shifts the sampled distribution.
- Exposomics (ContaminantDB) may lack the modular lipid/carnitine/polyphenol-rich chemistry that benefits from rule-based modules, so domain conclusions may not generalize to small-molecule exposome compounds.
- Sampling variance across random draws can change per-domain scores; fix random seeds or repeat sampling for stability.
- Only superclass-level stratification was used; finer class or subclass resolution was not evaluated in this workflow.

## Evidence

- [full_text] Build three evaluation subsets by sampling molecules from the Metlin 2019 dataset so that each subset matches the superclass (ClassyFire) distribution of compounds in HMDB, FooDB, and ContaminantDB, representing exposomics, foodomics, and human metabolomics domains.: [evidence span withheld — anchor status: not_found]
- [results] Adding the rule-based MSRB module to the machine-learned module improves spectral prediction performance in the foodomics and metabolomics datasets: [evidence span withheld — anchor status: not_found]
- [full_text] Counting a peak pair as a match when the m/z difference is smaller than 0.01 Da or 10 ppm (whichever occurs first).: [evidence span withheld — anchor status: not_found]
- [methods] Validation: the MSML+MSRB combination must show higher Dice scores than MSML alone on the foodomics and metabolomics domains, while showing no improvement on the exposomics domain: [evidence span withheld — anchor status: not_found]
