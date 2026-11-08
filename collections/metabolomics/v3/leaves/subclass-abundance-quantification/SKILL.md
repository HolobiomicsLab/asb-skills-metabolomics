---
name: subclass-abundance-quantification
description: Use when you have MAW's merged candidate results for a metabolomics dataset
  (e.g. file_id_mergedResults-with-one-Candidates.csv) and need to quantify and plot
  which chemical subclasses dominate the classified top-ranked annotations.
when_to_use_negative:
- You need per-feature structural annotations rather than class-level distribution
  summaries — use MAW's candidate selection output instead.
- Your candidates lack ClassyFire/CANOPUS classification output (e.g. classification
  step was not run in MAW-R), since no ChemOnt assignments can be aggregated.
- The candidate table contains multiple candidates per feature and you have not yet
  reduced it to the top-ranking annotation per feature.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: MAW-R
  role: R segment of the MAW workflow that performs the computational classification
    of candidates using CANOPUS and ClassyFire, producing the class assignments to
    be summarized
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-Py
  role: Python segment that produces the final merged candidates results (one top-ranked
    candidate per feature) used as input for abundance summarization
  repo: https://github.com/zmahnoor14/MAW
- name: CANOPUS
  role: Predicts compound chemical classes from fragments/molecular fingerprints,
    feeding the ClassyFire ChemOnt assignments aggregated here
- name: ClassyFire
  role: Provides the ChemOnt hierarchy (superclass/class/subclass) onto which assignments
    are mapped and aggregated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:b3af9d9cf87229a3b2e73db9f91841e85514f218cdada6acf73a1fbb9120c073
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - CANOPUS
  - ClassyFire
  - MAW-Py
  - MAW-R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# subclass-abundance-quantification

## Summary

Summarize the chemical class assignments of MAW's top-ranking annotations by mapping ClassyFire/CANOPUS classifications onto the ChemOnt hierarchy and computing abundance percentages at superclass, class, and subclass levels, visualized as a sunburst chart.

## When to use

Use when you have MAW's merged candidate results for a metabolomics dataset (e.g. file_id_mergedResults-with-one-Candidates.csv) and need to quantify and plot which chemical subclasses dominate the classified top-ranked annotations.

## When NOT to use

- You need per-feature structural annotations rather than class-level distribution summaries — use MAW's candidate selection output instead.
- Your candidates lack ClassyFire/CANOPUS classification output (e.g. classification step was not run in MAW-R), since no ChemOnt assignments can be aggregated.
- The candidate table contains multiple candidates per feature and you have not yet reduced it to the top-ranking annotation per feature.

## Inputs

- MAW final merged candidates results CSV (file_id_mergedResults-with-one-Candidates.csv) with one top-ranking candidate per feature
- Chemical class assignments from MAW's computational classification step (CANOPUS / ClassyFire)
- ClassyFire ChemOnt class hierarchy mapping

## Outputs

- subclass_abundances.csv with subclass-level percentage abundances of annotated features
- class_sunburst.png sunburst chart of the superclass/class/subclass hierarchy with segments sized by abundance

## How to apply

Load the final MAW merged candidates results (file_id_mergedResults-with-one-Candidates.csv) for the dataset and keep only the top-ranking annotation per feature. Extract the chemical class assignments produced by MAW's computational classification step, which classifies candidates using CANOPUS and ClassyFire. Map each assignment onto the ClassyFire ChemOnt hierarchy and aggregate counts at the superclass, class, and subclass levels. Compute subclass abundances as percentages of annotated features, write them to subclass_abundances.csv, and render the superclass/class/subclass hierarchy as a sunburst chart (class_sunburst.png) with subclass segments sized by abundance. Validate by checking reproduced abundances against reported values within rounding tolerance and confirming all key subclass labels appear in the output.

## Related tools

- **MAW-R** (R segment of the MAW workflow that performs the computational classification of candidates using CANOPUS and ClassyFire, producing the class assignments to be summarized) — https://github.com/zmahnoor14/MAW
- **MAW-Py** (Python segment that produces the final merged candidates results (one top-ranked candidate per feature) used as input for abundance summarization) — https://github.com/zmahnoor14/MAW
- **CANOPUS** (Predicts compound chemical classes from fragments/molecular fingerprints, feeding the ClassyFire ChemOnt assignments aggregated here)
- **ClassyFire** (Provides the ChemOnt hierarchy (superclass/class/subclass) onto which assignments are mapped and aggregated)

## Examples

```
python scripts/class_sunburst.py --input file_id_mergedResults-with-one-Candidates.csv --out subclass_abundances.csv --plot class_sunburst.png
```

## Evaluation signals

- Reproduced subclass abundances match reported values — Lipids 33.92%, Organic acids 17.02%, Benzenoids 13.94%, Phenylpropanoid and polyketides 11.34% — within rounding tolerance
- All four key subclass labels (Lipids, Organic acids, Benzenoids, Phenylpropanoid and polyketides) appear in subclass_abundances.csv
- Each feature in the input contributes at most one top-ranking annotation; subclass percentages sum to at most 100% of annotated features
- class_sunburst.png renders the superclass/class/subclass hierarchy with segment sizes proportional to subclass abundances

## Limitations

- Only top-ranking annotations are summarized; features left unannotated by MAW (e.g. bryophyte natural product classes experimentally challenging to acquire with tandem MS) are excluded from the abundance denominators.
- SIRIUS lipid class annotation can produce false positives due to the complex nature of lipids and co-eluting lipid isomers, which can inflate Lipid subclass abundances.
- The low number of annotated features caused by mismatch between experimental spectra in spectral databases and structures in compound databases limits how representative the subclass distribution is.

## Evidence

- [methods] Extract the chemical class assignments produced by the workflow's computational classification step, which classifies data using CANOPUS and ClassyFire.: [evidence span withheld — anchor status: not_found]
- [results] In the sunburst plot of classified top-ranking bryophytes annotations, the most abundant subclasses are Lipids (33.92%), Organic acids (17.02%), Benzenoids (13.94%), and Phenylpropanoid and polyketides (11.34%).: [evidence span withheld — anchor status: not_found]
- [abstract] spectral and compound database matching with computational classification: "spectral and compound database matching with computational classification"
- [readme] The classification of the tentative candidates from the input data are classified using [CANOPUS](https://bio.informatik.uni-jena.de/software/canopus/) and: [evidence span withheld — anchor status: external_doc]
