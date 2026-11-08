---
name: tandem-ms-formula-identification
description: Use when you have unfiltered high-resolution MS/MS spectra (plus MS1
  isotope pattern) of a small molecule and need its molecular formula and candidate
  structure without a reference mass spectral database.
when_to_use_negative:
- A reference mass spectral database already gives a confident library match — SIRIUS/CSI:FingerID
  is designed for cases 'beyond elemental composition without requiring compound structures
  or a mass spectral database'.
- Input is a complex LC-MS/MS feature table needing peak picking/alignment first;
  SIRIUS expects preprocessed peak lists per compound, not raw vendor files of an
  entire run.
- The query is a structure with no chance of being structure-disjoint from CSI:FingerID's
  training data, so performance numbers will not be comparable to published benchmarks.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS 4
  role: performs isotope pattern analysis and fragmentation tree computation to rank
    candidate molecular formulas from the MS/MS and isotope data
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: receives fragmentation trees and spectra uploaded from SIRIUS and searches
    molecular structure databases to return ranked candidate structures
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-019-0344-8
    title: 'SIRIUS 4: a rapid tool for turning tandem mass spectra into metabolite
      structure information'
  source_content_hash: sha256:a97a027dd40f8b091a120607794dd35c1e86ad404b6b6e8e974a6cffab16298b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - SIRIUS
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# tandem-ms-formula-identification

## Summary

SIRIUS-based workflow that identifies the molecular formula of an unknown compound from high-resolution LC-MS/MS data by combining isotope pattern analysis of the parent peak with fragmentation tree computation, then hands ranked candidates to CSI:FingerID for structure search.

## When to use

Use when you have unfiltered high-resolution MS/MS spectra (plus MS1 isotope pattern) of a small molecule and need its molecular formula and candidate structure without a reference mass spectral database.

## When NOT to use

- A reference mass spectral database already gives a confident library match — SIRIUS/CSI:FingerID is designed for cases 'beyond elemental composition without requiring compound structures or a mass spectral database'.
- Input is a complex LC-MS/MS feature table needing peak picking/alignment first; SIRIUS expects preprocessed peak lists per compound, not raw vendor files of an entire run.
- The query is a structure with no chance of being structure-disjoint from CSI:FingerID's training data, so performance numbers will not be comparable to published benchmarks.

## Inputs

- high-resolution MS/MS peak lists (tandem mass spectra) of small compounds
- MS1 isotope pattern of the parent ion (high-resolution mass spectrometry)
- ionization mode information (positive/negative)

## Outputs

- ranked candidate molecular formulas (top-k, e.g. top 1 and top 2)
- hypothetical fragmentation tree with fragment formula annotations and losses
- ranked candidate structures from CSI:FingerID structure database search

## How to apply

Run SIRIUS 4 on the peak list: first, isotope pattern analysis ranks candidate elemental compositions from the MS1 isotope envelope of the parent ion, supporting both positive and negative ionization modes; if MS1 data is available, the parent peak from MS1 is used to decompose the ion. Second, SIRIUS computes a hypothetical fragmentation tree from the MS/MS spectrum, in which nodes are annotated with molecular formulas of the fragments and arcs represent fragmentation events (losses). Third, the fragmentation trees and spectra are directly uploaded from SIRIUS to the CSI:FingerID web service, which searches molecular structure databases and returns ranked candidate structures; restrict the search to a biomolecular database when the sample is of metabolomic origin. Finally, evaluate identification quality by top-k ranks (top 2 for formulas, top 10 for structures, top 3 when restricted to the biomolecular database) and exclude queries whose correct structure is not structure-disjoint from CSI:FingerID's training data.

## Related tools

- **SIRIUS 4** (performs isotope pattern analysis and fragmentation tree computation to rank candidate molecular formulas from the MS/MS and isotope data) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (receives fragmentation trees and spectra uploaded from SIRIUS and searches molecular structure databases to return ranked candidate structures) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input challenge.ms --project casmi2016 --isotope-pattern-analysis --fragmentation-tree --positive --csi-fingerid-web --db biomolecular
```

## Evaluation signals

- Published benchmark: 93.8% of CASMI 2016 compounds received the correct molecular formula at top 1 and 96.2% at top 2 using MS/MS plus isotope pattern data — recompute these rates on the same challenge set as a sanity check.
- Overall structure identification rates should exceed 70% on challenging metabolomics datasets when using SIRIUS 4.
- Every ranked formula must be consistent with the parent ion mass and isotope pattern from the MS1 spectrum (the parent peak from MS1 is used to decompose the ion when MS1 is available).
- Fragmentation tree nodes must be annotated with fragment molecular formulas and edges must correspond to plausible fragmentation losses.
- Verify that query structures are structure-disjoint from CSI:FingerID training data before reporting identification rates.

## Limitations

- Support for in-source fragmentations and adducts is labeled experimental.
- Isotope pattern analysis was rewritten in this release and is 'hopefully more stable than before' — older versions had bugs in isotope pattern selection.
- Bugs in penalizing molecular formulas on intrinsically charged mode and in isotope pattern selection affected earlier versions; use an up-to-date release.
- Performance can degrade when the correct structure is not structure-disjoint from CSI:FingerID's training data.

## Evidence

- [methods] isotope pattern analysis ranks candidates from MS1: "SIRIUS deduces molecular formulas of small compounds by ranking isotope patterns from mass spectra of high resolution."
- [methods] fragmentation tree annotation: "a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments and arcs (edges) represent fragmentation events (losses)."
- [methods] CSI:FingerID integration: "Fragmentation trees and spectra can be directly uploaded from SIRIUS to the CSI:FingerID, CANOPUS and MSNovelist web services."
- [discussion] MS1 parent peak decomposition: "if MS1 data is available, SIRIUS will now always use the parent peak from MS1 to decompose the"
- [results] CASMI 2016 benchmark numbers: [evidence span withheld — anchor status: not_found]
- [abstract] >70% identification rate: "Using SIRIUS 4, we achieved identification rates of more than 70% on challenging metabolomics datasets."
- [discussion] experimental adduct support: "Experimental support for in-source fragmentations and adducts"
- [methods] no mass spectral database needed: "automated and high-throughput analysis of small-compound MS data beyond elemental composition without requiring compound structures or a mass spectral database."
- [discussion] isotope pattern stability caveat: "isotope pattern analysis is now rewritten and hopefully more stable than before"
