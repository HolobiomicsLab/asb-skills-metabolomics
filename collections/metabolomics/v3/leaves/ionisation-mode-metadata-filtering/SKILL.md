---
name: ionisation-mode-metadata-filtering
description: Use when importing an annotated MS/MS spectral library (e.g. GNPS MGF)
  and the downstream model or pipeline requires spectra restricted to one ionisation
  mode and valid metadata such as ionmode and precursor m/z.
when_to_use_negative:
- The pipeline must handle both positive and negative ionisation modes jointly (e.g.
  the MS2DeepScore 2.0 cross-ionisation-mode model), so discarding one mode would
  remove valid data
- Spectra lack reliable ionmode metadata, so mode-based selection cannot be verified
- The goal is peak-level cleaning only (e.g. intensity thresholds) and no metadata-based
  selection is needed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Python library used to import, handle, and filter mass spectra including metadata-based
    (ionisation mode) selection
  repo: https://github.com/matchms/ms2deepscore
- name: MS2DeepScore
  role: Downstream Siamese neural network for which the ionisation-mode-consistent
    spectra are curated as training input
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:2db2351e83369df1abbc23712887da5be6180b7a048ca92ab384c7de70a13849
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ionisation-mode-metadata-filtering

## Summary

Curating step for GNPS MS/MS library spectra in which spectra not matching the target ionisation mode are discarded before downstream preprocessing and model training, ensuring that only spectra with consistent, usable metadata enter the cleaned dataset.

## When to use

Use when importing an annotated MS/MS spectral library (e.g. GNPS MGF) and the downstream model or pipeline requires spectra restricted to one ionisation mode and valid metadata such as ionmode and precursor m/z.

## When NOT to use

- The pipeline must handle both positive and negative ionisation modes jointly (e.g. the MS2DeepScore 2.0 cross-ionisation-mode model), so discarding one mode would remove valid data
- Spectra lack reliable ionmode metadata, so mode-based selection cannot be verified
- The goal is peak-level cleaning only (e.g. intensity thresholds) and no metadata-based selection is needed

## Inputs

- Annotated GNPS MS/MS library spectra (MGF format, 210,407 entries from Zenodo record 4699300)
- Per-spectrum metadata containing ionisation mode (ionmode) and peak lists

## Outputs

- Subset of spectra in the target ionisation mode with >= 5 peaks in the 10-1000 m/z range, metadata preserved
- Cleaned spectra MGF file (cleaned_spectra.mgf) used for downstream training and evaluation

## How to apply

Load the deposited annotated GNPS spectra (Zenodo record 4699300, 210,407 library entries) with matchms and inspect each spectrum's metadata for ionisation mode. Discard spectra that are not in positive ionisation mode, and additionally require at least 5 peaks between 10 and 1000 m/z so that only spectra with sufficient peak content survive. After this metadata-based filtering, the remaining spectra can undergo the peak-level cleaning steps (removing peaks below 0.1% of maximum intensity, capping at the 1000 highest-intensity peaks, square-root transforming intensities) and be written to a cleaned_spectra.mgf file with annotated metadata preserved. The rationale is that the MS2DeepScore Siamese network trains on spectrum pairs, so mixing inconsistent or missing ionisation-mode metadata would corrupt the structural similarity signal the model learns.

## Related tools

- **matchms** (Python library used to import, handle, and filter mass spectra including metadata-based (ionisation mode) selection) — https://github.com/matchms/ms2deepscore
- **MS2DeepScore** (Downstream Siamese neural network for which the ionisation-mode-consistent spectra are curated as training input) — https://github.com/matchms/ms2deepscore

## Examples

```
from matchms.importing import load_from_mgf; spectra = [s for s in load_from_mgf('gnps_spectra.mgf') if s.get('ionmode') == 'positive' and len(s.peaks) >= 5]
```

## Evaluation signals

- No spectrum without positive ionisation mode metadata remains in the cleaned set
- All retained spectra have at least 5 peaks within the 10-1000 m/z window
- Annotated metadata (InChIKey, SMILES and/or InChI) is preserved through the filtering step
- The cleaned dataset matches the published scale (210,407 spectra retrieved, of which 184,698 annotated) after all curation steps

## Limitations

- Filtering by ionisation mode discards negative-mode spectra, which are excluded from the original MS2DeepScore 0.2.0 training set
- Spectra from GNPS come from different instrumentation types, are of varying quality, and may still be unbalanced with regard to represented compound classes after filtering
- Metadata quality in public libraries varies, so ionmode annotations themselves may be imperfect

## Evidence

- [other] workflow step: [evidence span withheld — anchor status: not_found]
- [discussion] dataset: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: https://zenodo.org/record/4699300."
- [results] tool: "MS2DeepScore is integrated in matchms, a recently developed Python library for mass spectra import, handling and comparisons"
- [intro] training data: "The model was trained using a dataset of 109,734 MS/MS spectra, which was built through curating and cleaning spectra obtained from GNPS [16]"
- [readme] readme cross-mode note: [evidence span withheld — anchor status: external_doc]
