---
name: adduct-filtering-spectrum-selection
description: Use when a raw spectral library contains multiple adducts, collision
  energies, and replicate spectra per compound, and you need one consistent binary
  per-compound target spectrum for binary peak-presence prediction.
when_to_use_negative:
- Predicting peak intensities rather than peak presence/absence — SingleFrag deliberately
  targets binary presence since intensity is less robust across experimental conditions
- Working with spectra from adducts other than [M+H] (e.g. [M-H]-, [M+Na]+), which
  are filtered out during this step
- Library matching against curated empirical spectra (e.g. NIST MS/MS) where no binarized
  target construction is needed
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SingleFrag
  role: Source of the raw spectral dataset and the downstream per-MZ-position peak-presence
    classifiers that consume the binarized spectra
  repo: https://github.com/MaribelPR/SingleFrag
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:b5018ecd06f74734f6a70b24b10821e94b6d38e449a60ab60be89cfea1493ab4
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# adduct-filtering-spectrum-selection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Filters a raw MS/MS spectral library down to experimental [M+H] precursor spectra and merges all spectra of the same compound into a single binarized 0.01 m/z bin presence/absence spectrum, producing the per-compound targets used to train SingleFrag's per-bin peak classifiers.

## When to use

Use when a raw spectral library contains multiple adducts, collision energies, and replicate spectra per compound, and you need one consistent binary per-compound target spectrum for binary peak-presence prediction.

## When NOT to use

- Predicting peak intensities rather than peak presence/absence — SingleFrag deliberately targets binary presence since intensity is less robust across experimental conditions
- Working with spectra from adducts other than [M+H] (e.g. [M-H]-, [M+Na]+), which are filtered out during this step
- Library matching against curated empirical spectra (e.g. NIST MS/MS) where no binarized target construction is needed

## Inputs

- Raw MS/MS spectral library from the SingleFrag Figshare dataset (2,291,119 spectra with adduct/precursor annotations)
- Compound structure identifiers (InChIKey → canonical SMILES) for deduplication

## Outputs

- Binarized per-compound spectra table (0.01 m/z bins, peak presence/absence) for 30,591 compounds
- Train/validation/test split files (80/10/10 by compound)

## How to apply

Download the SingleFrag dataset from Figshare (2,291,119 raw spectra) and filter to the 434,480 experimental [M+H]+ spectra using the adduct/precursor annotation in the dataset, since the models target positive-mode protonated molecules. Resolve canonical SMILES for each compound via its InChIKey to remove duplicate and non-standardized structures. Merge all filtered spectra belonging to the same compound into a single binary spectrum in which an m/z bin is set to 1 if a peak is present in at least one of the compound's spectra, and 0 otherwise; then discretize into 0.01 m/z bins. Split the resulting 30,591 compounds into train/validation/test sets at 80/10/10 before training the per-MZ-position ANN, GNN, or COM models.

## Related tools

- **SingleFrag** (Source of the raw spectral dataset and the downstream per-MZ-position peak-presence classifiers that consume the binarized spectra) — https://github.com/MaribelPR/SingleFrag

## Examples

```
python scripts/merge_binarize_spectra.py --input data/raw_spectra.pkl --adduct '[M+H]+' --bin-width 0.01 --output data/binarized_compound_spectra.csv
```

## Evaluation signals

- Filtered set contains exactly 434,480 experimental [M+H] spectra from the original 2,291,119
- Merged binarized table covers 30,591 unique compounds after InChIKey-based canonical SMILES deduplication
- Every m/z bin value is strictly binary (0 or 1), binned at 0.01 m/z resolution
- Split files partition compounds 80/10/10 with no compound appearing in more than one set
- A compound's bin is 1 iff the peak appears in at least one of its filtered spectra (union, not intersection)

## Limitations

- Only [M+H] spectra are retained, so the merged targets do not represent fragments from other precursor adducts
- Merging by peak-presence union can yield spectra broader than any single experimental spectrum, and the underlying public libraries (GNPS, MSDial, MassBank) often contain spectra at only one or a few collision energies per compound
- Peak presence rather than intensity is modeled, so relative abundance information is discarded

## Evidence

- [methods] In the merged binary spectrum, an m/z bin is set to 1 if a peak is present in at least one of the compound's available spectra, and set to 0 otherwise.: [evidence span withheld — anchor status: not_found]
- [intro] rather than predicting the intensity of peaks in the spectrum, SingleFrag focuses on predicting the presence or absence of each peak: "rather than predicting the intensity of peaks in the spectrum, SingleFrag focuses on predicting the presence or absence of each peak"
- [discussion] other spectral libraries such as GNPS, MSDial, or MassBank often lack sufficient curation and typically contain spectra acquired at only one or a few collision energies per compound: "other spectral libraries such as GNPS, MSDial, or MassBank often lack sufficient curation and typically contain spectra acquired at only one or a few collision energies per compound"
- [readme] The following step would be to download from **Fighare** our dataset called "SingleFrag": "The following step would be to download from **Fighare** our dataset called "SingleFrag" (https://figshare.com/articles/dataset/SingleFrag/27266787), which contains the data files necessary"
