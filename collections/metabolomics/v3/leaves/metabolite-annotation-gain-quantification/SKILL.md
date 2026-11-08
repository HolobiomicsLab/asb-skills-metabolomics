---
name: metabolite-annotation-gain-quantification
description: 'Decision support for whether a predicted-structure library actually
  expands annotation coverage: you hold a large set of experimental MS/MS spectra
  (e.g. from MetaboLights blood analyses) and want the fold-change in matched spectra
  versus a known-metabolite-only search.'
when_to_use_negative:
- You want to annotate a single spectrum or small cohort with putative structure IDs
  rather than quantify library-level coverage gain — use a spectral search/annotation
  skill instead.
- Your candidate structures come from a database of confirmed metabolites only, with
  no predicted structures to add — there is no added-library contrast to quantify.
- Your MS/MS data are deconvoluted peak lists with no precursor masses or the predicted
  library cannot be scored with cosine similarity under consistent settings, making
  the two-library comparison incommensurable.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: DeepMet
  role: Generates the anticipated-metabolite structures whose in silico MS/MS spectra
    form the added library that produces the annotation gain
  repo: https://github.com/skinniderlab/CLM
- name: Python
  role: Used to build the two spectral libraries, run cosine-similarity matching against
    all experimental spectra, and write the comparison table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:34ca724fef16bdf5db86065ff1a33897ef277da7f9e56db47dad2c6ec6436c6f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolite-annotation-gain-quantification

## Summary

Quantifies the increase in experimental MS/MS spectra that can be annotated with a chemical structure when an in silico predicted-spectrum library (e.g. from DeepMet) is added to a known-metabolite library (e.g. HMDB), using cosine-similarity spectral matching at a fixed threshold.

## When to use

Decision support for whether a predicted-structure library actually expands annotation coverage: you hold a large set of experimental MS/MS spectra (e.g. from MetaboLights blood analyses) and want the fold-change in matched spectra versus a known-metabolite-only search.

## When NOT to use

- You want to annotate a single spectrum or small cohort with putative structure IDs rather than quantify library-level coverage gain — use a spectral search/annotation skill instead.
- Your candidate structures come from a database of confirmed metabolites only, with no predicted structures to add — there is no added-library contrast to quantify.
- Your MS/MS data are deconvoluted peak lists with no precursor masses or the predicted library cannot be scored with cosine similarity under consistent settings, making the two-library comparison incommensurable.

## Inputs

- Set of experimental tandem MS/MS spectra (e.g. 29.1 million spectra compiled from human blood analyses on MetaboLights)
- In silico predicted MS/MS spectral library for known metabolite structures (e.g. HMDB)
- In silico predicted MS/MS spectral library for anticipated metabolites from DeepMet (2.4 million structures)
- Cosine similarity threshold (>= 0.8) used to define a match

## Outputs

- Count of experimental spectra matched against the known-metabolite-only library
- Count of experimental spectra matched against the combined (known + predicted) library
- Fold-change (gain factor) in matched spectra attributable to the predicted library
- msms_annotation_comparison.csv with match counts and fold-gain comparison

## How to apply

Assemble or download the experimental MS/MS spectra resource (in the paper, 29.1 million spectra compiled from 4,510 human blood analyses deposited on MetaboLights). Construct two in silico MS/MS spectral libraries from chemical structures: one for known metabolites only (HMDB structures) and one for known plus DeepMet-anticipated metabolites (the paper used ~2.4 million DeepMet structures). Predict fragment spectra for each candidate structure, then search every experimental spectrum against each library independently using cosine-similarity spectral matching under identical scoring settings. Count experimental spectra matched at cosine similarity >= 0.8 for each library, and report both absolute match counts and the fold-gain of the combined-library search over the known-only search. The same cosine threshold and scoring algorithm must be applied to both searches so the comparison isolates the effect of adding predicted structures.

## Related tools

- **DeepMet** (Generates the anticipated-metabolite structures whose in silico MS/MS spectra form the added library that produces the annotation gain) — https://github.com/skinniderlab/CLM
- **Python** (Used to build the two spectral libraries, run cosine-similarity matching against all experimental spectra, and write the comparison table)

## Examples

```
python scripts/annotate_and_compare.py --spectra metabolights_blood.ms2 --hmdb_lib hmdb_predicted.mgf --deepmet_lib deepmet_2.4M_predicted.mgf --threshold 0.8 --out msms_annotation_comparison.csv
```

## Evaluation signals

- Match counts are computed with an identical cosine threshold (>= 0.8) and identical scoring settings across both library searches.
- The combined-library match count is >= the known-only match count (fold-change >= 1); a result below 1 indicates a pipeline error.
- The same experimental spectrum set (e.g. 29.1 million spectra) is used for both searches, so the delta is attributable only to the added library.
- Outputs are written to msms_annotation_comparison.csv with match counts and the computed fold-change.

## Limitations

- The gain reflects annotations at cosine similarity >= 0.8, which may include false positives; structure-level confidence requires additional validation.
- Predicting MS/MS spectra from chemical structures remains an open problem, so in silico library quality bounds the achievable gain.
- The reported gain depends on the specific experimental spectrum resource (human blood analyses from MetaboLights); gains may differ for other tissues or acquisition methods.

## Evidence

- [discussion] Fold-gain from adding DeepMet library to HMDB library was approximately 3x: [evidence span withheld — anchor status: not_found]
