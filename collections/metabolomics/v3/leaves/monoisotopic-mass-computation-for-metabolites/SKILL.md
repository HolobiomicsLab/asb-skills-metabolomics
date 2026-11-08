---
name: monoisotopic-mass-computation-for-metabolites
description: Use when a differential MS1 feature table contains dysregulated m/z peaks,
  no reference-standard spectra are available, and a library of parent structures
  plus biotransformation reaction rules can be exploited to compute theoretical candidate
  masses for preannotation.
when_to_use_negative:
- MS2 spectra match a reference library or authentic standard is available — traditional
  bottom-up library-based annotation is more direct and should be preferred
- No reaction rules or parent structures exist for the analyte class, so theoretical
  monoisotopic masses cannot be enumerated
- The feature table has not been filtered for dysregulated features (no DMS layer),
  making an omics-scale candidate-vs-feature cross-reference uninformative
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: CMDN
  role: 'Framework hosting the RCS layer: applies reaction rules in silico to parents
    and preannotates candidates against DMS features at 5 ppm'
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: BioTransformer 3.0
  role: Compatible in silico biotransformation prediction tool for enumerating metabolite
    candidates from parent structures
- name: OrgMassSpecR
  role: R package (listed in CMDN README) for computing monoisotopic m/z of enumerated
    derivative structures
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: MS-Dial (ver. 4.80)
  role: Compatible MS data processing application used upstream for feature detection
    feeding the DMS layer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:d91849d6db9b48feed868e0ef447859c225e58868af51093fdc1bb69212342bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - BioTransformer 3.0
  - CMDN
  - MS-Dial (ver. 4.80)
  - OrgMassSpecR
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# monoisotopic-mass-computation-for-metabolites

## Summary

Enumerates theoretical biotransformed metabolites by applying reaction rules in silico to parent xenobiotics (here 1,021 pesticides), computes their monoisotopic m/z values to form a pseudo-MS1 candidature space, and preannotates them against dysregulated MS1 features by accurate-mass matching at ≤ 5 ppm.

## When to use

Use when a differential MS1 feature table contains dysregulated m/z peaks, no reference-standard spectra are available, and a library of parent structures plus biotransformation reaction rules can be exploited to compute theoretical candidate masses for preannotation.

## When NOT to use

- MS2 spectra match a reference library or authentic standard is available — traditional bottom-up library-based annotation is more direct and should be preferred
- No reaction rules or parent structures exist for the analyte class, so theoretical monoisotopic masses cannot be enumerated
- The feature table has not been filtered for dysregulated features (no DMS layer), making an omics-scale candidate-vs-feature cross-reference uninformative

## Inputs

- Parent compound structure library (e.g., 1,021-pesticide structure library)
- Curated biotransformation reaction rules (290 nonisomeric phase-I rules from KEGG and curated literature)
- DMS-flagged dysregulated feature table (differential metabolite space m/z features)
- R environment with tidyverse, OrgMassSpecR, readxl; optional BioTransformer 3.0 for in silico biotransformation enumeration

## Outputs

- rcs_candidate_features.csv with per-feature theoretical m/z, parent pesticide identity, applied reaction rule, and m/z error in ppm
- Nonredundant theoretical derivative set (expected scale: 2,886 nonredundant biotransformed derivatives of 1,021 pesticides)

## How to apply

Load the parent structure library (the 1,021-pesticide set in this study) and the curated reaction rules (290 nonisomeric phase-I rules sourced from KEGG and curated literature, adopted in CMDN). Apply each rule in silico — R scripts in the CMDN repo using OrgMassSpecR, with BioTransformer 3.0 as a compatible in silico enumeration tool — to every parent compound to enumerate theoretical biotransformed derivatives and compute their monoisotopic m/z values, forming the reaction-based pseudo-MS1 candidature space (RCS). Take the dysregulated features flagged by the differential metabolite space (DMS) layer and match theoretical candidate m/z values against them using a mass-error tolerance of 5 ppm. Emit one row per preannotated feature containing the theoretical m/z, parent pesticide identity, applied reaction rule, and m/z error in ppm. Use the resulting candidate table as input to downstream MS2 similarity network (SSN) scoring rather than as final annotation, since accurate mass alone is only a preannotation step.

## Related tools

- **CMDN** (Framework hosting the RCS layer: applies reaction rules in silico to parents and preannotates candidates against DMS features at 5 ppm) — https://github.com/LeaveMeNotTonight/CMDN
- **BioTransformer 3.0** (Compatible in silico biotransformation prediction tool for enumerating metabolite candidates from parent structures)
- **OrgMassSpecR** (R package (listed in CMDN README) for computing monoisotopic m/z of enumerated derivative structures) — https://github.com/LeaveMeNotTonight/CMDN
- **MS-Dial (ver. 4.80)** (Compatible MS data processing application used upstream for feature detection feeding the DMS layer)

## Examples

```
OrgMassSpecR::MonoisotopicMass(formula = C10H10ClNO2S)  # R session with CMDN repo packages: tidyverse, OrgMassSpecR, readxl
```

## Evaluation signals

- Every cross-referenced entry satisfies the 5 ppm mass-error criterion
- The enumerated nonredundant candidate set is consistent in scale with the reported annotation outcome (2,886 nonredundant derivatives of 1,021 pesticides)
- Each output row in rcs_candidate_features.csv carries a parent pesticide identity, an applied reaction rule, and an m/z error in ppm (schema completeness check)
- No preannotated feature maps to a parent-rule pair absent from the 290-rule set; rules applied are nonisomeric phase-I transformations
- Candidate features are restricted to DMS-flagged dysregulated features rather than the full feature list

## Limitations

- In silico biotransformation prediction platforms 'often fall short of providing direct evidence under real-world, omics-scale conditions' — mass preannotation is not structural proof
- Scarcity of authentic standards means candidates require multichannel validation (retention time prediction, standard verification, murine studies, time-course logistic modeling) beyond the 5 ppm match
- Coverage is bounded by the adopted rule set (290 nonisomeric phase-I rules); biotransformations outside these rules are missed

## Evidence

- [full_text] Theoretical candidate features were matched to DMS features at 5 ppm: [evidence span withheld — anchor status: not_found]
