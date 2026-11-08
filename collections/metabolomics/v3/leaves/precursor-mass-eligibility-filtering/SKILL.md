---
name: precursor-mass-eligibility-filtering
description: Use when assembling analog compound-pair benchmarks from MS/MS libraries
  — you have enumerated pairs and must enforce precursor mass and adduct eligibility
  before substructure matching.
when_to_use_negative:
- Your goal is to filter spectral peaks within a single spectrum (e.g. removing peaks
  <1% of base peak intensity) — that is peak preprocessing, not pair eligibility screening.
- You have already identified modification sites via RDKit GetSubstructMatch and edge
  counting; this filter runs earlier, before structural matching.
- Precursor masses exceed 2000 Da or the mass difference exceeds 50% of the smaller
  precursor — such pairs are excluded by design, not candidates for further processing.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ModiFinder
  role: Benchmark built by this filtering step is the evaluation set for ModiFinder's
    modification site localization
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: RDKit
  role: Convert SMILES to molecule objects and run GetSubstructMatch to confirm a
    single-modification substructure relationship after mass filtering
- name: MS/MS library search
  role: Source of the annotated MS/MS spectra, precursor masses, and adducts that
    this filter screens
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:2b6316fddbca3490e6a983d5086cc91e4679d17eeedc46bee83c678afc3aaba6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MS/MS library search
  - ModiFinder
  - RDKit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# precursor-mass-eligibility-filtering

## Summary

Filters within-library compound pairs by precursor mass constraints (mass < 2000 Da, mass difference < 50% of the smaller precursor mass, and matching M+H adducts) to define an analog benchmark of single-modification pairs, yielding 12,909 pairs from four reference MS/MS libraries.

## When to use

Use when assembling analog compound-pair benchmarks from MS/MS libraries — you have enumerated pairs and must enforce precursor mass and adduct eligibility before substructure matching.

## When NOT to use

- Your goal is to filter spectral peaks within a single spectrum (e.g. removing peaks <1% of base peak intensity) — that is peak preprocessing, not pair eligibility screening.
- You have already identified modification sites via RDKit GetSubstructMatch and edge counting; this filter runs earlier, before structural matching.
- Precursor masses exceed 2000 Da or the mass difference exceeds 50% of the smaller precursor — such pairs are excluded by design, not candidates for further processing.

## Inputs

- Enumerated within-library compound pairs from four public reference MS/MS libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB)
- Precursor masses and adduct annotations for each library compound
- SMILES strings for each compound

## Outputs

- Filtered set of eligible analog pairs (12,909 pairs with M+H adducts differing by a single structural modification)
- Benchmark pair table annotated with source library

## How to apply

Enumerate every within-library compound pair from the reference MS/MS libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, and BERKELEY-LAB), then apply the paper's four benchmark eligibility checks: (i) confirm both precursor masses are less than 2000 Da, (ii) require that the difference in precursor masses be less than 50% of the smaller compound's precursor mass, and (iii) confirm both members share the same M+H adduct. Rationale: these filters restrict the candidate pool to plausible analogs whose spectra were collected under matched conditions and whose structural relationship is a single modification, which is the precondition for ModiFinder-style substructure matching via RDKit GetSubstructMatch followed by counting edges between matched and unmatched atoms (pairs with more than one edge are discarded). Validate the filtered pair set totals 12,909 analog pairs.

## Related tools

- **ModiFinder** (Benchmark built by this filtering step is the evaluation set for ModiFinder's modification site localization) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **RDKit** (Convert SMILES to molecule objects and run GetSubstructMatch to confirm a single-modification substructure relationship after mass filtering)
- **MS/MS library search** (Source of the annotated MS/MS spectra, precursor masses, and adducts that this filter screens)

## Examples

```
python build_benchmark.py --libraries GNPS-MSMLS GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE BERKELEY-LAB --max-mass 2000 --max-diff-frac 0.5 --adduct M+H
```

## Evaluation signals

- Filtered pair count equals exactly 12,909 analog pairs across the four libraries.
- Every retained pair has both members sharing the same M+H adduct.
- No retained pair has a precursor mass ≥ 2000 Da.
- No retained pair has a precursor mass difference ≥ 50% of the smaller compound's precursor mass.
- After RDKit GetSubstructMatch, every retained pair has exactly one edge between matched-substructure atoms and atoms outside the match (single modification site).

## Limitations

- Filter assumes adduct annotation accuracy in the source MS/MS libraries; misannotated adducts would propagate.
- Pairs differing by more than one structural modification (multiple edges between matched and unmatched atoms) are discarded, so the benchmark only covers single-modification analogs.
- Only M+H adducts are retained, excluding analog pairs observed under other ionization modes or adduct types.

## Evidence

- [methods] 12,909 analog pairs: [evidence span withheld — anchor status: not_found]
- [readme] ModiFinder benchmark purpose: [evidence span withheld — anchor status: not_found]
