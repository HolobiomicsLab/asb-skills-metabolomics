---
name: metabolite-annotation-hmdb-metlin-matching
description: Use when a shortlist of tumor-co-localized m/z peaks (Pearson r ≥ 0.7
  against a massNet tumor-class prediction map) needs tentative molecular formula
  and adduct assignment via exact-mass search of HMDB and METLIN.
when_to_use_negative:
- Peaks have not passed the co-localization filter (Pearson < 0.7 vs. the tumor-class
  map) — annotating all 85,062 m/z variables would be untargeted and wasteful.
- Confirmed (not tentative) metabolite identification is required — exact-mass matching
  to HMDB/METLIN alone gives no structural confirmation.
- Data come from a lower mass-accuracy instrument where ppm errors cannot discriminate
  among candidate formulas and adducts.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: massNet
  role: Its VAE decoder reconstructs spatial images of identified m/z peaks and supplies
    the tumor-class prediction map used to pre-filter peaks (Pearson ≥ 0.7) before
    database annotation
  repo: https://github.com/wabdelmoula/massNet
- name: msiPL
  role: 'Peak-identification step: Pearson correlation on VAE spatial patterns yields
    the highly co-localized m/z peaks that feed the HMDB/METLIN search'
- name: HMDB
  role: Metabolite database searched for tentative formula/adduct assignment with
    recorded ppm error (e.g., C9H13N2O9P [M+2K-H]+, -2.1 ppm)
- name: METLIN
  role: Alternative metabolite database for exact-mass annotation of retained peaks
    (e.g., C10H16N5O13P3 [M+K]+, 0.36 ppm)
- name: Python
  role: Environment for running the database search and exporting/ranking the tumor_correlated_features.tsv
    table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:74ce2071254a9edcd2dccad8912a5eb0f060310cb58f85861db12656ed1259c1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - HMDB
  - METLIN
  - Python
  - massNet
  - msiPL
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolite-annotation-hmdb-metlin-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates tumor-correlated m/z peaks from MALDI FT-ICR mass spectrometry imaging with tentative molecular formulas and positive-mode adducts ([M+K]+, [M+2K-H]+) by exact-mass searching the HMDB and METLIN databases and recording the ppm error of each assignment, enabling Table 2-style reporting of tumor-delineation metabolite features.

## When to use

Use when a shortlist of tumor-co-localized m/z peaks (Pearson r ≥ 0.7 against a massNet tumor-class prediction map) needs tentative molecular formula and adduct assignment via exact-mass search of HMDB and METLIN.

## When NOT to use

- Peaks have not passed the co-localization filter (Pearson < 0.7 vs. the tumor-class map) — annotating all 85,062 m/z variables would be untargeted and wasteful.
- Confirmed (not tentative) metabolite identification is required — exact-mass matching to HMDB/METLIN alone gives no structural confirmation.
- Data come from a lower mass-accuracy instrument where ppm errors cannot discriminate among candidate formulas and adducts.

## Inputs

- Tumor-co-localized m/z peak list from massNet VAE decoder / msiPL (Pearson correlation ≥ 0.7 vs. the tumor-class prediction map)
- High-resolution positive-ion m/z values from MALDI 9.4 Tesla FT-ICR MSI data (HDF5 format, 85,062 m/z variables)
- HMDB and METLIN metabolite databases for exact-mass / formula / adduct search

## Outputs

- Ranked annotation table tumor_correlated_features.tsv with columns m/z, correlation, molecular formula, adduct, database, ppm error
- Table 2-style tentative assignments with ppm errors (e.g., C9H13N2O9P [M+2K-H]+ at -2.1 ppm in HMDB; C10H16N5O13P3 [M+K]+ at 0.36 ppm in Metlin)

## How to apply

Start from the m/z peaks that survived the msiPL co-localization filter — those whose spatial patterns reconstructed by the massNet VAE decoder from the 85,062-variable HDF5 MALDI FT-ICR data reached Pearson correlation ≥ 0.7 with the tumor-class prediction map. Search HMDB and METLIN for exact-mass matches in positive ion mode, allowing potassium adducts such as [M+K]+ and [M+2K-H]+, and record the ppm error for every assignment; the 9.4 Tesla SolariX FT-ICR mass accuracy makes single-ppm-level errors the expected regime. Anchor assignments to the paper's examples: the top tumor-correlated peak at m/z 400.9546 ± 0.01 matches C9H13N2O9P [M+2K-H]+ in HMDB at -2.1 ppm, and m/z 545.9589 matches C10H16N5O13P3 [M+K]+ in Metlin at 0.36 ppm. Treat all matches as tentative since the workflow provides no MS/MS or standard-based confirmation. Rank the annotated features by descending Pearson correlation and export the table (m/z, correlation, formula, adduct, database, ppm error) as tumor_correlated_features.tsv.

## Related tools

- **massNet** (Its VAE decoder reconstructs spatial images of identified m/z peaks and supplies the tumor-class prediction map used to pre-filter peaks (Pearson ≥ 0.7) before database annotation) — https://github.com/wabdelmoula/massNet
- **msiPL** (Peak-identification step: Pearson correlation on VAE spatial patterns yields the highly co-localized m/z peaks that feed the HMDB/METLIN search)
- **HMDB** (Metabolite database searched for tentative formula/adduct assignment with recorded ppm error (e.g., C9H13N2O9P [M+2K-H]+, -2.1 ppm))
- **METLIN** (Alternative metabolite database for exact-mass annotation of retained peaks (e.g., C10H16N5O13P3 [M+K]+, 0.36 ppm))
- **Python** (Environment for running the database search and exporting/ranking the tumor_correlated_features.tsv table)

## Examples

```
import pandas as pd; df = pd.read_csv('tumor_correlated_features.tsv', sep='\t'); assert (df.correlation >= 0.7).all(); top = df.sort_values('correlation', ascending=False)
```

## Evaluation signals

- Exported tumor_correlated_features.tsv contains only rows with Pearson correlation ≥ 0.7, sorted by descending correlation.
- Table schema includes all six expected columns: m/z, correlation, formula, adduct, database, ppm error.
- Reproduces Table 2-style anchors: C9H13N2O9P [M+2K-H]+ at m/z 400.9554553 with -2.1 ppm (HMDB) and C10H16N5O13P3 [M+K]+ at 545.9589 with 0.36 ppm (Metlin).
- Recorded ppm errors are small (roughly sub-2 ppm magnitude), consistent with 9.4 T FT-ICR mass accuracy, and adducts are plausible positive-mode potassium forms ([M+K]+, [M+2K-H]+).
- Every retained peak carries both a database source and a ppm error — no unannotated or missing-database rows among the correlation ≥ 0.7 features.

## Limitations

- Assignments are tentative exact-mass matches only; the workflow provides no MS/MS or authentic-standard confirmation.
- ppm-level discrimination depends on FT-ICR mass accuracy and is not transferable to lower-resolution instruments.
- The candidate set is bounded by the upstream msiPL/massNet co-localization filter (Pearson ≥ 0.7); changes to the classifier or threshold change which peaks get annotated.

## Evidence
