---
name: isotope-peak-filtering
description: Use when importing centroided MS2 spectra (txt, msp, or mgf exported
  from MS-DIAL, or mgf from MZmine) into a DIATAGeR DIADataObj and before LipidIdentifier/ScoreLipids
  TG annotation, to strip isotopes and noise peaks from multiplexed DIA spectra.
when_to_use_negative:
- The input is raw uncentroided profile MS2 spectra or a feature table that has not
  yet had spectra attached via SpectraImport — DIATAGeR expects a DIADataObj with
  centroid spectra.
- The goal is lipid-class identification other than triacylglycerols — DIATAGeR is
  explicitly for TGs only (lipid='TAG').
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DIATAGeR
  role: R package providing the RemoveIsotopes function used for MS2 deisotoping and
    denoising within the TG annotation workflow
  repo: Velenosi-Lab/DIATAGeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:0f81ff5f1d2d785dda6950753bdb4b9be55b15735d117e1f5b2dd587b59cead0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Velenosi-Lab/DIATAGeR
  tools:
  - DIATAGeR
  - DecoyIdentifier
  - FDRCutoff
  - LipidIdentifier
  - RemoveIsotopes
  - ScoreLipids
  - SpectraImport
  - allDecoys
  - caret
  - measureFileImport
  - pROC
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# isotope-peak-filtering

## Summary

Deisotope and denoise multiplexed DIA-MS2 spectra prior to triacylglycerol annotation using DIATAGeR's RemoveIsotopes function, which removes isotope peaks and low-intensity/density-outlier peaks so downstream TG identification against target and decoy databases operates on clean fragment spectra.

## When to use

Use when importing centroided MS2 spectra (txt, msp, or mgf exported from MS-DIAL, or mgf from MZmine) into a DIATAGeR DIADataObj and before LipidIdentifier/ScoreLipids TG annotation, to strip isotopes and noise peaks from multiplexed DIA spectra.

## When NOT to use

- The input is raw uncentroided profile MS2 spectra or a feature table that has not yet had spectra attached via SpectraImport — DIATAGeR expects a DIADataObj with centroid spectra.
- The goal is lipid-class identification other than triacylglycerols — DIATAGeR is explicitly for TGs only (lipid='TAG').

## Inputs

- DIATAGeR DIADataObj containing the MS-DIAL alignment feature list and attached centroid MS2 spectra (e.g. from Spectra_txt_Liver_MSDIAL/Centroid, txt format, Pos ion mode)

## Outputs

- Deisotoped and denoised DIADataObj ready for LipidIdentifier/ScoreLipids TG annotation

## How to apply

After measureFileImport loads the feature list (e.g. Feature_List_Liver_MSDIAL.txt with ResultsFiletype='MSDIAL', IonMode='Pos') and SpectraImport attaches centroid MS2 spectra with rttol=10, run RemoveIsotopes on the DIADataObj with ppmtol=15, low.intensity.cutoff=100, density.cutoff=TRUE, and density.cutoff.value=1. The ppm tolerance governs isotope peak matching, peaks with intensity below the low.intensity.cutoff (default 100) are removed, and the density-based cutoff (value 1, to be changed 'with caution' per the README) suppresses spurious density outliers. The cleaned object then feeds LipidIdentifier (ppmtol=15, rttol=5) and the parallel DecoyIdentifier/allDecoys run before ScoreLipids logistic-regression scoring; consistent ppmtol between filtering and identification keeps fragment matching coherent.

## Related tools

- **DIATAGeR** (R package providing the RemoveIsotopes function used for MS2 deisotoping and denoising within the TG annotation workflow) — Velenosi-Lab/DIATAGeR

## Examples

```
DIA_Pos <- RemoveIsotopes(DIADataObj = DIA_Pos, ppmtol = 15, low.intensity.cutoff = 100, density.cutoff = TRUE, density.cutoff.value = 1)
```

## Evaluation signals

- Features with missing spectra were diverted to a 'Troubleshooting' folder during SpectraImport; confirm the filtering input contains spectra-attached features only.
- Filtering parameters match the published benchmark: ppmtol=15, low.intensity.cutoff=100, density.cutoff=TRUE, density.cutoff.value=1.
- Downstream validation: all 9 supplied TG standards are annotated at empirical FDR < 0.1 in the liver dataset after filtering (ResultsTG_FDR.RDS from FDRCutoff with iteration=10, cutoff=0.1, rttol=10).
- The same ppmtol=15 tolerance is used in both RemoveIsotopes and the subsequent LipidIdentifier/ScoreLipids calls, keeping fragment matching consistent.

## Limitations

- The density-based cutoff value (default 1) should be 'changed with caution' according to the README; inappropriate values risk removing genuine fragment peaks.
- DIA generates multiplexed tandem mass spectra that complicate identification; filtering quality directly affects target-decoy FDR estimates.
- The article's discussion section content is absent from the source document, so no explicit limitation text beyond the README could be extracted.

## Evidence
