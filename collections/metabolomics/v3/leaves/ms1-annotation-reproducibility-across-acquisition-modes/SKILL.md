---
name: ms1-annotation-reproducibility-across-acquisition-modes
description: Use when benchmarking or reproducing ms1-id annotation yield across LC-MS
  acquisition modes on the NIST fecal dataset (MSV000095787), comparing DDA MS/MS
  against MS1-only full-scan annotation.
when_to_use_negative:
- Input files contain true MS/MS scans you want to search directly — use standard
  MS/MS library search rather than pseudo MS/MS reconstruction from MS1 in-source
  fragments
- Input is MALDI imaging data (imzML) — ms1-id provides a separate msi workflow (gnps_minmz100.pkl,
  spectral usage cutoff 0.05) rather than the LC-MS route benchmarked here
- Input is already an aligned feature table or pseudo MS/MS mgf without chromatographic
  profiles — correlation-based feature clustering (±1.5 s RT, ≥4 consecutive shared
  scans) requires raw MS1 scan data
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms1-id
  role: Python package that annotates mzML/mzXML LC-MS files by generating pseudo
    MS/MS from in-source fragments and running precursor-tolerant reverse spectral
    matching against indexed GNPS libraries
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Feature-detection backend (ver 1.0.11) used by ms1-id to extract metabolic
    features and to flag isotope peaks that are excluded from pseudo MS/MS generation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:0240cf81a4bc4d0eecd5f1f6822638ec9f4a01fa89991901f0035258d6dbbb51
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - MassCube
  - ms1-id
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ms1-annotation-reproducibility-across-acquisition-modes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce ms1-id structural annotation counts of NIST reference human fecal LC-MS data (MSV000095787) acquired in five modes (DDA MS/MS 42 eV, MS1 DDA, and MS1 full-scan at 0/10/20 eV isCID), yielding 567, 306, 484, 511, and 604 spectral library matches respectively.

## When to use

Use when benchmarking or reproducing ms1-id annotation yield across LC-MS acquisition modes on the NIST fecal dataset (MSV000095787), comparing DDA MS/MS against MS1-only full-scan annotation.

## When NOT to use

- Input files contain true MS/MS scans you want to search directly — use standard MS/MS library search rather than pseudo MS/MS reconstruction from MS1 in-source fragments
- Input is MALDI imaging data (imzML) — ms1-id provides a separate msi workflow (gnps_minmz100.pkl, spectral usage cutoff 0.05) rather than the LC-MS route benchmarked here
- Input is already an aligned feature table or pseudo MS/MS mgf without chromatographic profiles — correlation-based feature clustering (±1.5 s RT, ≥4 consecutive shared scans) requires raw MS1 scan data

## Inputs

- mzML/mzXML LC-MS files of NIST reference human fecal samples in five acquisition modes (DDA MS/MS 42 eV; MS1 DDA; MS1 full-scan 0 eV, 10 eV, 20 eV isCID) from MassIVE MSV000095787 (Q Exactive)
- Pre-indexed GNPS MS/MS spectral library (July 17, 2024 version): db/gnps.pkl and db/gnps_k10.pkl from the ms1_id GitHub releases

## Outputs

- nist_fecal_match_counts.csv with per-mode spectral library match counts (DDA MS/MS 42 eV: 567; MS1 DDA: 306; MS1 0 eV: 484; MS1 10 eV: 511; MS1 20 eV: 604)
- Per-mode annotated feature lists mapping each accepted annotation to its metabolic feature via pseudo MS/MS retention time and precursor m/z
- ms1-id aligned_feature_table.tsv with combined MS1 and MS/MS annotations

## How to apply

Download the NIST human fecal LC-MS dataset (MSV000095787) from GNPS/MassIVE with its DDA MS/MS (42 eV), MS1 DDA, and full-scan files at 0, 10, and 20 eV isCID energies, then fetch the pre-indexed GNPS MS/MS library (July 17, 2024 version) from the ms1_id GitHub releases. Run the ms1-id pipeline with the MassCube backend (ver 1.0.11) to extract metabolic features and build pseudo MS/MS spectra: pair features within ±1.5 s retention time, require at least 4 consecutive shared MS1 scans with Pearson correlation ≥0.80, and exclude isotope peaks determined by MassCube. Perform precursor-tolerant reverse spectral search of each pseudo MS/MS against the indexed GNPS library with LC-MS cutoffs of minimum reverse cosine score 0.7, minimum 3 matched peaks, and minimum spectral usage 0.20, using mass-dependent peak intensity scaling (k = 10) with square-root (0.5-power) transformation. Link accepted annotations back to features via pseudo MS/MS retention time and precursor mass, keeping the top 1 hit per unique precursor m/z, then count matches per acquisition mode and write counts plus per-mode annotated feature lists to nist_fecal_match_counts.csv.

## Related tools

- **ms1-id** (Python package that annotates mzML/mzXML LC-MS files by generating pseudo MS/MS from in-source fragments and running precursor-tolerant reverse spectral matching against indexed GNPS libraries) — https://github.com/Philipbear/ms1_id
- **MassCube** (Feature-detection backend (ver 1.0.11) used by ms1-id to extract metabolic features and to flag isotope peaks that are excluded from pseudo MS/MS generation)

## Examples

```
ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- Per-mode match counts fall within ±10% of the reported values: DDA MS/MS 42 eV ≈ 567, MS1 DDA ≈ 306, MS1 0 eV ≈ 484, MS1 10 eV ≈ 511, MS1 20 eV ≈ 604
- Relative ordering is preserved: MS1 full-scan 20 eV isCID yields the most annotations and MS1 DDA the fewest
- All five acquisition modes produce non-empty annotated feature lists keyed by precursor m/z with at most one (top-1) annotation per unique precursor m/z
- Search parameters match the paper's LC-MS cutoffs: minimum reverse cosine 0.7, minimum 3 matched peaks, minimum spectral usage 0.20, peak scaling k = 10 with square-root intensity transformation
- Per-file LC-MS runtime is ~5-7 min; exceeding 10 min signals misconfigured mass_detect_int_tol (default 2e5 for Orbitraps)

## Limitations

- Results depend on the indexed GNPS library version (July 17, 2024); library/tool-version drift may shift counts, hence the ±10% tolerance
- The ms1-id package version itself is not stated in the article (only MassCube ver 1.0.11 and the GNPS2 workflow version 2024.09.20 are given), complicating exact reproduction
- Only the gnps.pkl library is used for MS/MS annotation while both gnps.pkl and gnps_k10.pkl are used for MS1 annotation, so mode-specific library handling must be respected

## Evidence

- [readme] Package role: [evidence span withheld — anchor status: external_doc]
- [readme] Demo script and data setup: [evidence span withheld — anchor status: external_doc]
