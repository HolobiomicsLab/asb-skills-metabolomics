---
name: lowess-residual-scaler-fitting
description: Use when aligning two independently processed same-method nontargeted
  LC-MS datasets whose RT and m/z drift between runs, and Source features must be
  scaled to a Target dataset before match identification.
when_to_use_negative:
- Datasets were acquired with different LC-MS methods — Eclipse is designed to align
  two or more same-method nontargeted LCMS datasets
- The task is redundant-feature clustering within a single dataset rather than alignment
  across datasets — use Gravity (clustering on RT and correlation) instead
- RT drift is batch-level drift correctable by pooled technical replicates and internal
  standards — use Blueshift DriftCorrection instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Eclipse
  role: 'Implements the subalignment matching: LOWESS scaler fitting, RSE weighting,
    ±6 RSE cutoff, and penalty-score best-match ranking via MSAligner'
  repo: https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA, Feature Abundances pivot) and installable
    package containing Eclipse
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Runtime environment in which the LOWESS scaler fitting and matching are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:0781df574f86fc1d25dd3e848c3bdf6701a1c494cc6e983e86dca3a6d54b2082
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lowess-residual-scaler-fitting

## Summary

Fits survey-based inter-dataset retention-time and m/z scalers via LOWESS on shared features between two LC-MS feature tables, then uses Residual Standard Error (RSE) of each fit to weight residual distances for candidate match filtering in Eclipse pairwise subalignments.

## When to use

Use when aligning two independently processed same-method nontargeted LC-MS datasets whose RT and m/z drift between runs, and Source features must be scaled to a Target dataset before match identification.

## When NOT to use

- Datasets were acquired with different LC-MS methods — Eclipse is designed to align two or more same-method nontargeted LCMS datasets
- The task is redundant-feature clustering within a single dataset rather than alignment across datasets — use Gravity (clustering on RT and correlation) instead
- RT drift is batch-level drift correctable by pooled technical replicates and internal standards — use Blueshift DriftCorrection instead

## Inputs

- Merged Feature Metadata + Feature Abundances tables (bmxp.FMDATA joined with the Compound_ID x injection_id pivot) for each LC-MS dataset
- Feature Metadata fields Compound_ID, RT, MZ, and Intensity per dataset
- Source→Target dataset pair designation for each pairwise subalignment

## Outputs

- Fitted LOWESS RT and m/z scalers per Source→Target pair
- RSE values for each LOWESS scaler fit
- Pairwise subalignment match tables (subalignment_matches_<source>_vs_<target>.csv) with source and target Compound_ID, RT, MZ, RSE-scaled distance, and penalty rank

## How to apply

First merge bmxp.FMDATA (Feature Metadata) with the Feature Abundances pivot (Compound_ID x injection_id) for each dataset, since Eclipse requires merged tables as input. For each Source→Target dataset pair, fit LOWESS scalers for RT and m/z on the features shared between the two merged tables, capturing the systematic inter-dataset shift. Compute the Residual Standard Error (RSE) of each LOWESS fit, weight the residual RT/m/z/Intensity distances by the RSE, and retain candidate matches whose Source-scaled distances fall within ±6 RSE of the Target feature's RT, m/z, and Intensity. Rank the surviving candidate matches per feature with the penalty-score ranking and record the best-ranked one-to-one match. This RSE-based normalization makes the cutoff adaptive to how well each LOWESS scaler fits, rather than using a fixed ppm or second tolerance.

## Related tools

- **Eclipse** (Implements the subalignment matching: LOWESS scaler fitting, RSE weighting, ±6 RSE cutoff, and penalty-score best-match ranking via MSAligner) — https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- **bmxp** (Provides the shared schema (bmxp.FMDATA, Feature Abundances pivot) and installable package containing Eclipse) — https://github.com/broadinstitute/bmxp
- **Python** (Runtime environment in which the LOWESS scaler fitting and matching are executed)

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner
```

## Evaluation signals

- Each subalignment_matches_<source>_vs_<target>.csv exists and contains the required bmxp FMDATA fields (Compound_ID, RT, MZ) plus RSE-scaled distance and penalty rank columns
- Assigned best matches are one-to-one between Source and Target compound IDs (expert review of one-to-one assignment)
- Every recorded match falls within the ±6 RSE cutoff on RT, m/z, and Intensity after Source-to-Target scaling
- LOWESS scaler residuals are consistent with the reported RSE per fit (residual distances are RSE-weighted)
- Column headers match the bmxp shared schema (or the globally remapped equivalents such as Feature_ID/Filename) so cross-module remapping works

## Limitations

- Scalers are fit on shared features only, so dataset pairs with few common features may yield poorly constrained LOWESS fits and unreliable RSE estimates
- Eclipse targets same-method nontargeted LCMS datasets; cross-method alignment is out of scope
- n > 2 dataset alignment requires Eclipse's graph-based approach to handle complex matching scenarios, so pairwise subalignments alone may not capture all multi-dataset relationships
- bmxp version numbers and XIC shape-based clustering are not yet available per the README

## Evidence

- [results] In each subalignment, after the Source dataset is scaled to the Target dataset, potential matches are identified as Target features within ±6 RSE of the Source's RT, m/z, and Intensity: [evidence span withheld — anchor status: not_found]
- [readme] Eclipse aligns same-method nontargeted LCMS datasets: "Eclipse - Align two or more same-method nontargeted LCMS datasets"
- [methods] Merging FMDATA and Feature Abundances is required before Eclipse: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Shared schema terminology can be remapped globally: "These can be changed globally so that all packages will use the same terminology."
- [abstract] Graph-based handling of n>2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
