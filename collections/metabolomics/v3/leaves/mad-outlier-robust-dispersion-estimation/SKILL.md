---
name: mad-outlier-robust-dispersion-estimation
description: Use when benchmarking normalization approaches (e.g., tGAM, rGAM, rLOESS,
  SERFF, TIGER, QC-RLSC, QC-RSC) on metabolomics data containing replicate sample
  pairs run in different batches, to identify which method harmonizes intensities
  across batches.
when_to_use_negative:
- The dataset contains no replicate pairs (identical biological samples re-run in
  subsequent batches), so cross-batch replicate concordance cannot be computed
- The data are still raw/unnormalized — normalization (e.g., via the Metanorm R package)
  must be applied before MAD concordance can be assessed
- The goal is differential metabolite abundance testing rather than benchmarking normalization
  performance
tools:
- name: Metanorm
  role: R package producing the normalized intensity outputs (tGAM, rGAM, rLOESS,
    QC-RLSC, QC-RSC) that feed the replicate MAD concordance computation
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Runtime environment for Metanorm and for computing the MAD of rescaled replicate
    differences (>= 4.4.0 required)
  repo: https://cloud.r-project.org/index.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:7197ac3bdf2db26177b5875d0b777b6079d55ad4b0f076a074745d38edb4ca43
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# mad-outlier-robust-dispersion-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify cross-batch replicate concordance after metabolomics normalization by computing the MAD of rescaled intensity differences between between-batch replicate runs, then express each method's improvement as a percentage MAD reduction versus unnormalized data.

## When to use

Use when benchmarking normalization approaches (e.g., tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, QC-RSC) on metabolomics data containing replicate sample pairs run in different batches, to identify which method best harmonizes intensities across batches.

## When NOT to use

- The dataset contains no replicate pairs (identical biological samples re-run in subsequent batches), so cross-batch replicate concordance cannot be computed
- The data are still raw/unnormalized — normalization (e.g., via the Metanorm R package) must be applied before MAD concordance can be assessed
- The goal is differential metabolite abundance testing rather than benchmarking normalization performance

## Inputs

- BioHEART replicate-run intensity data: pairs of identical biological samples run in subsequent batches
- Normalized intensity matrices from tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, and QC-RSC (Metanorm-processed outputs)
- Unnormalized baseline intensities, log-transformed (base e) prior to normalization
- Replicate-pair and batch mapping indicating which runs form between-batch replicate pairs

## Outputs

- replicate_concordance_mad.tsv: per-method MAD of rescaled replicate differences and percentage MAD reduction versus unnormalized data

## How to apply

Load the Metanorm-processed replicate-run intensities for each normalization method plus the unnormalized baseline, with all intensities log-transformed (base e) prior to normalization. For each replicate pair and compound, calculate the difference between the two observed intensities rescaled by their mean intensity. Compute the MAD of these rescaled differences per normalization approach in R. Then compute each method's relative reduction in MAD versus the unnormalized baseline as a percentage. Lower MAD indicates better cross-batch correspondence and superior normalization; in the BioHEART benchmark, robust methods (tGAM, rGAM, rLOESS) reached 49–52% MAD reduction, outperforming SERFF (41%), TIGER (40%), and QC-RLSC/QC-RSC (18–22%).

## Related tools

- **Metanorm** (R package producing the normalized intensity outputs (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC) that feed the replicate MAD concordance computation) — https://github.com/UGent-LIMET/Metanorm
- **R** (Runtime environment for Metanorm and for computing the MAD of rescaled replicate differences (>= 4.4.0 required)) — https://cloud.r-project.org/index.html

## Examples

```
mad((rep1 - rep2) / ((rep1 + rep2) / 2))  # per replicate pair and compound, in R, on Metanorm-processed log(e) intensities
```

## Evaluation signals

- Robust methods (tGAM, rGAM, rLOESS) show a 49–52% MAD reduction versus unnormalized data
- SERFF shows 41% and TIGER shows 40% MAD reduction, matching reported values
- QC-RLSC and QC-RSC show only 18–22% MAD reduction, clearly below the robust methods
- Lower MAD consistently corresponds to better correspondence of intensities across batches — verify the ranking of methods is monotonic in MAD
- All intensities were log-transformed (base e) before normalization; the rescaled difference for each compound is (difference of the two replicate intensities) divided by their mean intensity

## Limitations

- Care should be taken to ensure neutrality in benchmarking, i.e., not potentially favoring any of the benchmarked methods
- Common benchmarking pitfalls exist, such as using a reduction of the relative standard deviation of QC samples or an increased number of differentially abundant metabolites as proxies for good normalization — MAD replicate concordance is the preferred alternative
- Outliers in metabolite intensities can deteriorate the performance of existing QC-sample-based normalization methods, which is why outlier-robust methods and MAD (a robust dispersion measure) are used

## Evidence

- [results] Robust normalization methods improved replicate concordance and reduced drift and batch effects: [evidence span withheld — anchor status: not_found]
- [readme] Metanorm implements the robust normalization methods used in this workflow: [evidence span withheld — anchor status: external_doc]
