---
name: chemical-class-annotation-aggregation
description: Use when you have a final merged MAW candidates table (file_id_mergedResults-with-one-Candidates.csv)
  with CANOPUS/ClassyFire class assignments per feature and need to summarize which
  chemical classes and subclasses dominate the annotated features.
when_to_use_negative:
- You only have raw MS2 .mzML input data — classification requires that MAW has already
  run preprocessing, spectral/compound dereplication, and candidate selection.
- Features lack any CANOPUS/ClassyFire class assignment (e.g. candidates with no predicted
  fingerprint), so they cannot be placed on the ChemOnt hierarchy.
- The question is about candidate ranking or annotation quality rather than the class
  distribution of annotated features.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: MAW-R
  role: Runs the computational classification segment of the workflow that produces
    the CANOPUS/ClassyFire class assignments being aggregated
  repo: https://github.com/zmahnoor14/MAW
- name: CANOPUS
  role: Predicts chemical classes for the tentative candidates from SIRIUS fingerprints,
    feeding the ChemOnt hierarchy
  repo: https://github.com/zmahnoor14/MAW
- name: ClassyFire
  role: Provides the ChemOnt superclass/class/subclass hierarchy onto which assignments
    are mapped and aggregated
  repo: https://github.com/zmahnoor14/MAW
- name: SIRIUS
  role: Compound database dereplication tool whose fingerprints underpin the class
    predictions used here
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:a5c092a9500ebe28d10fc71f93260a7fe28091c90e1cdadfa0312f71ad2d0ee3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - CANOPUS
  - ClassyFire
  - MAW-R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# chemical-class-annotation-aggregation

## Summary

Aggregates the computational chemical classifications produced by the MAW workflow (CANOPUS/ClassyFire) for top-ranked annotated MS2 features into a ChemOnt superclass/class/subclass hierarchy, computing subclass abundances and rendering them as a sunburst chart.

## When to use

Use when you have a final merged MAW candidates table (file_id_mergedResults-with-one-Candidates.csv) with CANOPUS/ClassyFire class assignments per feature and need to summarize which chemical classes and subclasses dominate the annotated features.

## When NOT to use

- You only have raw MS2 .mzML input data — classification requires that MAW has already run preprocessing, spectral/compound dereplication, and candidate selection.
- Features lack any CANOPUS/ClassyFire class assignment (e.g. candidates with no predicted fingerprint), so they cannot be placed on the ChemOnt hierarchy.
- The question is about candidate ranking or annotation quality rather than the class distribution of annotated features.

## Inputs

- MAW final merged candidates table (file_id_mergedResults-with-one-Candidates.csv) with one top-ranked candidate per MS2 feature
- CANOPUS/ClassyFire chemical class assignments attached to the tentative candidates
- ClassyFire ChemOnt class hierarchy

## Outputs

- subclass_abundances.csv (subclass-level counts and percentages of annotated features)
- class_sunburst.png (superclass/class/subclass sunburst chart with subclass segments sized by abundance)

## How to apply

Load the final MAW merged candidates results for the dataset (e.g. the bryophytes file_id_mergedResults-with-one-Candidates.csv) and keep only the top-ranking annotation per feature, since MAW prioritizes one final candidate per feature from the candidate list. Extract the chemical class assignments produced by MAW's computational classification step, which classifies tentative candidates using CANOPUS and ClassyFire. Map each assignment onto the ClassyFire ChemOnt hierarchy and aggregate counts at the superclass, class, and subclass levels. Compute subclass abundances as percentages of annotated features and write them to subclass_abundances.csv, then render the hierarchy as a sunburst chart (class_sunburst.png) with subclass segments sized by abundance. Rationale: MAW explicitly performs chemical compound classification as its fifth step after candidate selection, so class-level summaries are the natural downstream view of the prioritized candidates.

## Related tools

- **MAW-R** (Runs the computational classification segment of the workflow that produces the CANOPUS/ClassyFire class assignments being aggregated) — https://github.com/zmahnoor14/MAW
- **CANOPUS** (Predicts chemical classes for the tentative candidates from SIRIUS fingerprints, feeding the ChemOnt hierarchy) — https://github.com/zmahnoor14/MAW
- **ClassyFire** (Provides the ChemOnt superclass/class/subclass hierarchy onto which assignments are mapped and aggregated) — https://github.com/zmahnoor14/MAW
- **SIRIUS** (Compound database dereplication tool whose fingerprints underpin the class predictions used here) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- Reproduced subclass abundances match reported values (Lipids 33.92%, Organic acids 17.02%, Benzenoids 13.94%, Phenylpropanoid and polyketides 11.34%) within rounding tolerance
- All four subclass labels (Lipids, Organic acids, Benzenoids, Phenylpropanoid and polyketides) appear in subclass_abundances.csv
- Sunburst renders all three ChemOnt levels (superclass, class, subclass) with subclass segments sized by abundance
- Every counted feature has exactly one top-ranked annotation, consistent with MAW prioritizing one final candidate per feature
- Percentages of classified annotations sum to 100% within rounding tolerance

## Limitations

- SIRIUS lipid class annotation can produce false positives due to the complex nature of lipids, so the dominant Lipids subclass share should be interpreted cautiously
- Only a subset of features receive class assignments; the mismatch between spectra in spectral databases and structures in compound databases limits the number of annotated features that contribute to the distribution
- MAW's first version integrates only SIRIUS (CSI:FingerID) for compound databases, so classification coverage depends on SIRIUS predictions

## Evidence

- [readme] The classification of the tentative candidates from the input data are classified using [CANOPUS](https://bio.informatik.uni-jena.de/software/canopus/): [evidence span withheld — anchor status: external_doc]
- [intro] MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication, (4) candidate selection, and (5) chemical compound classification using various: "MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication, (4) candidate selection, and (5) chemical compound classification"
- [intro] It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature.: "It provides a list of top-scoring candidates as a network and also prioritizes a final candidate from the list, for each feature."
- [discussion] the annotation of lipid classes using SIRIUS can lead to false positives due to the complex nature of lipids: "the annotation of lipid classes using SIRIUS can lead to false positives due to the complex nature of lipids"
