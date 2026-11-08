---
name: lipid-class-frequency-thresholding
description: Use when compiling a reference MS/MS training library from a public spectral
  database (e.g. LipidMaps-derived .msp blocks) and you must decide which compound
  classes have enough spectra to support a data-centric model.
when_to_use_negative:
- When the library already contains only well-populated classes and per-class counts
  are uniformly high — the filter is a no-op
- 'When building an evaluation/hold-out set rather than a training set: the paper''s
  ≥50-spectra threshold was applied to the ST002044 training compilation, not to the
  independent test spectra'
- When class balance is achieved by resampling or class-weighted loss rather than
  by excluding classes
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: IDSL_MINT
  role: Framework that consumes the thresholded .msp training library; its YAML configuration
    and MSP parsing (via `comment:`/`comments:` and `SMILES:`/`InChI:` entries) operate
    on the frequency-filtered class set
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:5ef8e297a42929fe71ac96f003232f216b2cab4aee5a0b1105ed0910dbb41f93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lipid-class-frequency-thresholding

## Summary

A training-set curation step from the IDSL_MINT case study in which spectra compiled from the LipidMaps database are counted per lipid class and classes with fewer than 50 MS/MS spectra are dropped, so that deep learning models for ST002044 are trained only on lipid classes with sufficient spectral support.

## When to use

Use when compiling a reference MS/MS training library from a public spectral database (e.g. LipidMaps-derived .msp blocks) and you must decide which compound classes have enough spectra to support a data-centric model.

## When NOT to use

- When the library already contains only well-populated classes and per-class counts are uniformly high — the filter is a no-op
- When building an evaluation/hold-out set rather than a training set: the paper's ≥50-spectra threshold was applied to the ST002044 training compilation, not to the independent test spectra
- When class balance is achieved by resampling or class-weighted loss rather than by excluding classes

## Inputs

- Curated .msp spectral blocks from the LipidMaps database (e.g. lipidmaps_curated_training.msp derived from Zenodo file#2 lipidmaps_msp.zip), with positive and negative modes kept separate
- Per-spectrum lipid class labels or compound identifiers that map each MSP block to a lipid class

## Outputs

- A filtered training .msp containing only lipid classes with at least 50 spectra (6 well-covered classes in the ST002044 case study)
- A frequency table of spectra per lipid class documenting which classes were dropped

## How to apply

After assembling the curated training set (excluding LipidBlast- and NIST 20-covered compounds), parse the .msp blocks and count how many spectra belong to each lipid class represented in the library. Retain only classes meeting the frequency cutoff of at least 50 spectra; in the ST002044 case study this retained 6 of the 8 lipid classes present in the training data. Keep positive and negative ionization modes separate throughout, since separate ESI+ and ESI− models are trained. This thresholding protects model validity by preventing under-represented classes from diluting the training signal, mirroring the paper's stated goal of 'carefully compiling the training dataset for the case study (ST002044) to ensure a high validity of the resulting model'.

## Related tools

- **IDSL_MINT** (Framework that consumes the thresholded .msp training library; its YAML configuration and MSP parsing (via `comment:`/`comments:` and `SMILES:`/`InChI:` entries) operate on the frequency-filtered class set) — https://github.com/idslme/IDSL_MINT

## Examples

```
python -c "import collections; blocks = parse_msp('lipidmaps_curated_training.msp'); keep = {c for c, n in collections.Counter(b['class'] for b in blocks).items() if n >= 50}; write_msp([b for b in blocks if b['class'] in keep], 'training_filtered.msp')"
```

## Evaluation signals

- Per-class spectrum counts in the retained set are ≥ 50; classes below the cutoff are absent from the output .msp
- In the ST002044 reproducibility check, the filtered training set contains exactly 6 lipid classes out of the original 8
- Positive and negative ESI modes are handled as separate training files, matching the 6.96% (ESI+) and 4.58% (ESI−) coverage figures reported for the case-study training set
- Output .msp blocks still satisfy IDSL_MINT parsing requirements (PrecursorMZ, SMILES or InChI, Num Peaks entries present)

## Limitations

- The 50-spectra cutoff is a heuristic calibrated for the LipidMaps-derived lipid library; other domains or smaller libraries may need different thresholds
- Thresholding discards whole classes, so predictions for rare lipid classes cannot be made by the resulting model — the paper only reports accuracy for the 6 well-covered classes
- Class labels must be reliably derivable from MSP metadata (Name, Synon, SMILES/InChI); misassigned classes would corrupt the counts

## Evidence

- [results] Next, we carefully compiled the training dataset for the case study (ST002044) to ensure a high validity of the resulting model (Fig. 3).: "Next, we carefully compiled the training dataset for the case study (ST002044) to ensure a high validity of the resulting model (Fig. 3)."
- [results] Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1).: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [results] Our case study training set included 6.96% and 4.58% of all the compounds in the LipidMaps database in positive and negative modes: "Our case study training set included 6.96% and 4.58% of all the compounds in the LipidMaps database in positive and negative modes, respectively"
