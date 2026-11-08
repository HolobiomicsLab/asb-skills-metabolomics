---
name: mz-bin-discretization
description: Use when building training targets for per-fragment or per-bin MS/MS
  prediction models from multiple experimental spectra per compound, or when a presence/absence
  (binary) peak representation at 0.01 m/z resolution is required instead of raw intensities.
when_to_use_negative:
- When peak intensities or collision-energy-resolved information must be preserved
  — the binary merge deliberately discards intensity and collapses across all available
  spectra of a compound
- When spectra are only available at integer m/z resolution (e.g. HMDB-style spectra)
  — the binning works at 0.01 m/z resolution and integer inputs lose that precision
- When comparing two individual spectra for similarity scoring rather than building
  per-compound training targets
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: SingleFrag
  role: Source of the raw spectral library (Figshare dataset) that is filtered to
    434,480 [M+H] spectra and binarized; the resulting bins define the per-position
    classifiers
  repo: https://figshare.com/articles/dataset/SingleFrag/27266787
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:8b966b87e43c1573918fb3071f89f0ac824958b29098705ded1ad9a4bc93d764
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - SingleFrag
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mz-bin-discretization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Binarizes MS/MS spectra into fixed 0.01 m/z bins with per-bin presence/absence labels, merging all spectra of a compound into a single binary target spectrum. This is the target-encoding step behind SingleFrag's per-m/z-bin classifiers, which predict peak presence rather than intensity.

## When to use

Use when building training targets for per-fragment or per-bin MS/MS prediction models from multiple experimental spectra per compound, or when a presence/absence (binary) peak representation at 0.01 m/z resolution is required instead of raw intensities.

## When NOT to use

- When peak intensities or collision-energy-resolved information must be preserved — the binary merge deliberately discards intensity and collapses across all available spectra of a compound
- When spectra are only available at integer m/z resolution (e.g. HMDB-style spectra) — the binning works at 0.01 m/z resolution and integer inputs lose that precision
- When comparing two individual spectra for similarity scoring rather than building per-compound training targets

## Inputs

- Raw MS/MS spectra with adduct/precursor annotations (e.g. the SingleFrag Figshare dataset with 2,291,119 spectra)
- Per-compound InChIKeys for canonical SMILES resolution and deduplication

## Outputs

- Merged binarized per-compound spectra table (0/1 values per 0.01 m/z bin) for 30,591 compounds
- Train/validation/test split files (80/10/10 over compounds)

## How to apply

Start from the raw spectra library (the SingleFrag Figshare dataset contains 2,291,119 spectra) and filter to the experimental [M+H] spectra (434,480) using the adduct/precursor annotations. Standardize compounds by resolving canonical SMILES from each InChIKey to remove duplicates, yielding 30,591 unique compounds. Merge all spectra belonging to the same compound into a single binary spectrum: an m/z bin is set to 1 if a peak is present in at least one of the compound's available spectra, and 0 otherwise. Discretize the merged spectrum into 0.01 m/z bins, indexing each position as m/z * 100 (the convention SingleFrag uses, e.g. ModelVal files named by mz*100 and thresholds defined per the most frequented 1,000 MZ positions). Finally split compounds 80/10/10 into train/validation/test sets so that binarization never leaks across splits.

## Related tools

- **SingleFrag** (Source of the raw spectral library (Figshare dataset) that is filtered to 434,480 [M+H] spectra and binarized; the resulting bins define the per-position classifiers) — https://figshare.com/articles/dataset/SingleFrag/27266787

## Examples

```
df['mz_bin'] = (df['mz'] * 100).round().astype(int); merged = df.groupby('inchikey')['mz_bin'].agg(lambda s: sorted(set(s)))
```

## Evaluation signals

- Every m/z bin value in the merged table is strictly binary (0 or 1); no intensities remain
- Bin count matches 0.01 m/z resolution: position index equals m/z * 100 (e.g. m/z 73.05 -> position 7305)
- A bin is 1 exactly when at least one of the compound's contributing spectra had a peak there; spot-check against contributing raw spectra
- Deduplicated compound count equals 30,591 after InChIKey/SMILES canonicalization, with no compound appearing in more than one of the 80/10/10 splits
- Only the 434,480 experimental [M+H] spectra were included; the remaining ~1.86M spectra of other adduct types are excluded

## Limitations

- OR-merging presence across spectra ignores collision-energy diversity — spectra from libraries like GNPS, MSDial, or MassBank often contain spectra acquired at only one or a few collision energies, so merged targets may be incomplete or less informative
- The binary target discards peak intensity, which SingleFrag deliberately trades away for robustness across experimental conditions
- In practice only the most frequented 1,000 m/z positions in the training set are modeled, so rare bins are dropped from downstream prediction

## Evidence

- [methods] In the merged binary spectrum, an m/z bin is set to 1 if a peak is present in at least one of the compound's available spectra, and set to 0 otherwise.: [evidence span withheld — anchor status: not_found]
