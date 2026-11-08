---
name: metabolite-library-count-verification
description: Use when a published LC-MS metabolomics workflow reports a specific reference-library
  size (e.g. '571 metabolites') and a library table or supplementary deposit is available
  whose row count must be independently verified.
when_to_use_negative:
- The artifact is not a row-per-metabolite library table (e.g. it is raw mzXML/mzML
  chromatogram files or peak-quality metric tables), so a row count would not correspond
  to metabolites
- The goal is to re-evaluate peak quality or re-derive the validation column rather
  than verify an already-computed library size
- The publication reports no explicit library size to compare against, leaving no
  reference value for the match/mismatch check
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: MetaMOPE
  role: Workflow that produced the reference library being verified; its validation
    column decides whether metabolites in the sample mixture pass user-specified thresholds
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: MetaHILIC
  role: Repository cloned to obtain all source code and the HILIC LC-MS reference
    library artifact to be counted
  repo: https://github.com/CMDM-Lab/MetaHILIC
- name: R
  role: Language in which MetaMOPE's metric functions (MCQ, maximum intensity, RSD
    of retention time, Std/Blk, asymmetry factor, FWHM, modality) are written, contextualizing
    the validation column used to define library membership
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:d6caabcb25402b13b2e99ace54711eec485c7eda24ae9e1893d591ef51330909
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - MetaHILIC
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolite-library-count-verification

## Summary

A verification skill for counting metabolite entries in an LC-MS reference library artifact (e.g. the MetaMOPE/MetaHILIC library) and comparing the count against a published total, such as the 571 metabolites reported for the HILIC LC-MS platform.

## When to use

Use when a published LC-MS metabolomics workflow reports a specific reference-library size (e.g. '571 metabolites') and a library table or supplementary deposit is available whose row count must be independently verified.

## When NOT to use

- The artifact is not a row-per-metabolite library table (e.g. it is raw mzXML/mzML chromatogram files or peak-quality metric tables), so a row count would not correspond to metabolites
- The goal is to re-evaluate peak quality or re-derive the validation column rather than verify an already-computed library size
- The publication reports no explicit library size to compare against, leaving no reference value for the match/mismatch check

## Inputs

- Reference library table (CSV/TSV or supplementary file) from the MetaMOPE/MetaHILIC deposit, one row per metabolite
- Published expected library size (e.g. 571 metabolites for the HILIC LC-MS platform) from the article's results section

## Outputs

- library_count_verification.csv containing the counted metabolite total and a match/mismatch comparison against the reported value
- A single integer count of metabolite entries in the library table

## How to apply

Obtain the reference library artifact from the workflow's repository deposit (for MetaMOPE, the CMDM-Lab/MetaHILIC repo cloned via `git clone git@github.com:CMDM-Lab/MetaHILIC.git`). Load the library table with pandas, treating each row as one metabolite entry, and confirm the table carries a validation column that marks whether each entry passed the user-specified thresholds (Std/Blk intensity ratio > threshold r, retention-time RSD across the three replicates < threshold c, and peak intensity > threshold I). Count the number of metabolite rows to obtain the empirical library size. Compare this count to the published figure — for the MetaMOPE HILIC LC-MS platform the expected value is 571 metabolites — and record both the count and the match/mismatch result in an output CSV such as library_count_verification.csv. A mismatch should prompt checking whether unvalidated entries or duplicates inflate/deflate the row count.

## Related tools

- **MetaMOPE** (Workflow that produced the reference library being verified; its validation column decides whether metabolites in the sample mixture pass user-specified thresholds) — https://github.com/CMDM-Lab/MetaMOPE
- **MetaHILIC** (Repository cloned to obtain all source code and the HILIC LC-MS reference library artifact to be counted) — https://github.com/CMDM-Lab/MetaHILIC
- **R** (Language in which MetaMOPE's metric functions (MCQ, maximum intensity, RSD of retention time, Std/Blk, asymmetry factor, FWHM, modality) are written, contextualizing the validation column used to define library membership) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && python -c "import pandas as pd; lib = pd.read_csv('reference_library.csv'); print(len(lib)); open('library_count_verification.csv','w').write('count,expected,match\\n%d,571,%s' % (len(lib), len(lib)==571))"
```

## Evaluation signals

- The counted number of metabolite rows equals the published total: 571 for the MetaMOPE HILIC LC-MS platform
- library_count_verification.csv exists and contains exactly the counted total and a match/mismatch result
- Every counted row corresponds to a metabolite with a defined validation status (the validation column determines whether metabolites in the sample mixture are validated by user-specified thresholds)
- The count is derived from the deposit's library table rather than restated from the paper text, so recounting from the same artifact is reproducible

## Limitations

- No changelog was found in the document, so limitations, missing artifacts, or reproducibility gaps could not be extracted from the discussion section
- If the library artifact is absent from the deposit, the count cannot be independently verified and must be taken from the article text alone
- Rows failing the user-specified thresholds (Std/Blk > r, RSD of retention time < c, intensity > I) may or may not be excluded from the shipped table, so an inflated count can occur if unvalidated entries are retained

## Evidence

- [results] This workflow was successfully applied to construct a reference library of 571 metabolites for the HILIC LC-MS platform: "This workflow was successfully applied to construct a reference library of 571 metabolites for the HILIC LC-MS platform"
- [abstract] The validation column decides whether metabolites are validated by user-specified thresholds: "The column of validation decides whether the metabolites in the sample mixture are validated by the user-specified thresholds."
- [readme] Clone the MetaHILIC repository to obtain all source code: "Run `git clone git@github.com:CMDM-Lab/MetaHILIC.git` to get all source code"
- [abstract] Peak recording criteria: Std/Blk ratio greater than threshold r, RSD of retention time less than threshold c, and peak intensity larger than threshold I: "Only the peaks that satisfy the following criteria are recorded: the ratio of peak intensity to blank intensity (Std/Blk) is greater than a user-specified threshold r"
