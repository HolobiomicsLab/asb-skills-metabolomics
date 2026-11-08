---
name: spectral-match-validation
description: Use when scoring or filtering MS2/MS3+ lipid spectral matches from a
  LipiDex 2 (or similar LC-MS lipidomics) pipeline against purity and dot-product
  thresholds, or when reproducing QC-filtered identification counts on deposited spectral
  match tables.
when_to_use_negative:
- Input is already a Degreaser-filtered, QC-validated feature table ready for statistical
  analysis (e.g. Metaboanalyst import) — re-filtering would double-filter identifications
- Data are raw spectra not yet processed by Peak Finder/Spectrum Searcher — this skill
  validates matches, it does not perform peak detection or library search
- The lipid class or acquisition method lacks an in silico library with MSn tree-based
  fragmentation, so no library spectrum exists to compute dot-product/reverse dot-product
  scores against
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipiDex 2 Degreaser QC module
  role: Performs automated validation of lipid identifications and features using
    spectral match validation plus 6 evidence types (ECN vs. RT, linear dynamic range,
    %RSD, PQF), aggregating 7 filtering criteria
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2 Lipid Spectral Annotator module
  role: Uses Spectrum Searcher MS2 and MS3+ spectral matches to annotate fragment
    peaks and score match purity and dot-product/reverse dot-product values
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Pipeline whose Peak Finder and Spectrum Searcher outputs (.csv) are the inputs
    being validated; rewritten in C# .NET 6 with Thermo RawFileReader support
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:3aa2e36765f8151f1eb586256af238ce55c3e5907d1fcf5dee8df3f84a07270e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  - LipiDex 2 Lipid Spectral Annotator module
  - Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# spectral-match-validation

## Summary

Quality-control skill from LipiDex 2 for validating MS2/MS3+ lipid spectral matches using spectral purity, dot-product and reverse dot-product score thresholds, plus feature-level checks (%RSD in replicate QC samples) within the Degreaser QC module, to remove spurious identifications before downstream statistics.

## When to use

Use when scoring or filtering MS2/MS3+ lipid spectral matches from a LipiDex 2 (or similar LC-MS lipidomics) pipeline against purity and dot-product thresholds, or when reproducing QC-filtered identification counts on deposited spectral match tables.

## When NOT to use

- Input is already a Degreaser-filtered, QC-validated feature table ready for statistical analysis (e.g. Metaboanalyst import) — re-filtering would double-filter identifications
- Data are raw spectra not yet processed by Peak Finder/Spectrum Searcher — this skill validates matches, it does not perform peak detection or library search
- The lipid class or acquisition method lacks an in silico library with MSn tree-based fragmentation, so no library spectrum exists to compute dot-product/reverse dot-product scores against

## Inputs

- Annotated spectral match table (.csv) exported from the LipiDex 2 pipeline (Spectrum Searcher MS2/MS3+ matches)
- Peak Finder filtered results (.csv) for the chromatographic peak table
- Compound Discoverer 3.3 .cdResult files (Peak Finder input; PQF values used as QC evidence)

## Outputs

- Scored, validated spectral match table (e.g. annotated_spectral_matches_scored.csv) with per-match purity, dot-product, reverse dot-product and pass/fail calls
- Degreaser-filtered table of identified lipids, identifications, and unidentified features (e.g. 461 identifications, 2443 unidentified features for MSV000091033)
- Single-filter attribution summary counting identifications removed by each of the 7 filtering criteria

## How to apply

Load the deposited annotated spectral match table (.csv) exported from the LipiDex 2 pipeline into pandas. For each MS2 and MS3+ match, compute spectral purity as the fraction of fragment ion intensity attributable to the matched lipid versus co-eluting isobaric species, and compute cosine dot-product and reverse dot-product scores between the experimental MSn spectrum and its in silico library spectrum. Apply the match-quality thresholds used in the LipiDex 2 LC-MS method: spectral purity >=75%, dot product >=500, reverse dot product >=700; flag matches failing any criterion. At the feature level, feed Peak Finder and Spectrum Searcher results into the Degreaser QC module, which validates spectral matches for all matches in each lipid feature alongside the ECN vs. RT model, dilution-series linear dynamic range modeling, >30% RSD in replicate QC samples, and Compound Discoverer Peak Quality Factor (PQF) criteria; Degreaser aggregates results on 7 filtering criteria. Rationale: LipiDex 1 established spectral purity to account for co-eluting isobaric lipids, and LipiDex 2 automates these post-identification QC checks to reduce spurious and redundant features — on MSV000091033 (40 Nile rat plasma samples) this raised the identification rate from 5% to 19%.

## Related tools

- **LipiDex 2 Degreaser QC module** (Performs automated validation of lipid identifications and features using spectral match validation plus 6 evidence types (ECN vs. RT, linear dynamic range, %RSD, PQF), aggregating 7 filtering criteria) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2 Lipid Spectral Annotator module** (Uses Spectrum Searcher MS2 and MS3+ spectral matches to annotate fragment peaks and score match purity and dot-product/reverse dot-product values) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Pipeline whose Peak Finder and Spectrum Searcher outputs (.csv) are the inputs being validated; rewritten in C# .NET 6 with Thermo RawFileReader support) — https://github.com/coongroup/LipiDex-2/releases/latest

## Examples

```
import pandas as pd; df = pd.read_csv('annotated_spectral_matches.csv'); keep = df[(df['spectral_purity']>=0.75)&(df['dot_product']>=500)&(df['reverse_dot_product']>=700)]; keep.to_csv('annotated_spectral_matches_scored.csv', index=False)
```

## Evaluation signals

- Per-match purity, dot-product and reverse dot-product values agree with the deposited reference annotations for the same dataset
- Only matches meeting purity >=75%, dot product >=500 and reverse dot product >=700 are retained; all failing matches are flagged
- On MSV000091033, filtered output contains exactly 254 identified lipids, 461 identifications, and 2443 unidentified features, with the identification rate rising from 5% to 19%
- Single-filter attribution table is consistent with the 173-of-254 attribution reported for the Nile rat plasma dataset
- Features with >30% RSD in replicate QC samples are flagged for removal, matching Degreaser's documented behavior

## Limitations

- Degreaser requires input from the LipiDex 2 Peak Finder and Spectrum Searcher steps (Peak Finder needs Compound Discoverer 3.3 .cdResult files), so the skill cannot be run on peak tables from arbitrary pipelines without conversion
- Thresholds (75% purity, 500/700 dot products, 30% RSD) are method-specific values from the published LC-MS lipidomics workflow and may not transfer to other acquisition methods
- If a required fragment is absent in the spectrum, the identification is downgraded to sum composition, and insufficient plasmenyl evidence downgrades the result to plasmanyl — validation can lower structural resolution rather than reject outright

## Evidence

- [results] Annotation of chromatographic features required a minimum lipid spectral purity of 75% from co-eluting isobaric lipids, a minimum MS2 spectral match dot product of 500, and a reverse dot product of 700.: [evidence span withheld — anchor status: not_found]
