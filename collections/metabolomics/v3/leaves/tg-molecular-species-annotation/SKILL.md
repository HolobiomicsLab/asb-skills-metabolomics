---
name: tg-molecular-species-annotation
description: Use when processing DIA-lipidomics MS-DIAL or MZMine feature lists with
  multiplexed MS2 spectra and confident, FDR-controlled TG molecular-species annotations
  are needed for downstream differential analysis.
when_to_use_negative:
- The analyte class is not triacylglycerols — the README states 'DIATAGeR is for triacylglycerols
  only' (lipid='TAG')
- Input is raw vendor mass-spectrometry data rather than an exported MS-DIAL/MZMine
  feature list with MS2 spectra files
- Only MS1 precursor information is available with no MS2 spectra — TG identification
  and scoring depend on multiplexed MS2 fragment matching
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DIATAGeR
  role: 'R package providing the full TG annotation workflow: measureFileImport, SpectraImport,
    RemoveIsotopes, LipidIdentifier, ScoreLipids, FDRCutoff'
  repo: Velenosi-Lab/DIATAGeR
- name: caret
  role: machine-learning framework used within ScoreLipids to train the logistic-regression
    target/decoy scorer
- name: pROC
  role: used with ScoreLipids/FDRCutoff to compute AUC values of the scoring model
- name: MS-DIAL
  role: source of the alignment feature list and exported centroid MS2 spectra consumed
    by measureFileImport and SpectraImport
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:19171e6b0a184c08880392523b5fea29c20c64de77906d78f86aec7f3e20e617
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Velenosi-Lab/DIATAGeR
  tools:
  - DIATAGeR
  - DecoyIdentifier
  - FDRCutoff
  - LipidIdentifier
  - MS-DIAL
  - RemoveIsotopes
  - ScoreLipids
  - SpectraImport
  - allDecoys
  - caret
  - measureFileImport
  - pROC
  - tidyverse
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6071428571428571
n_claims: 28
---

# tg-molecular-species-annotation

## Summary

Annotate triacylglycerols to the molecular-species (fatty-acyl combination) level in DIA-lipidomics data using the DIATAGeR R package, with target-decoy logistic-regression scoring and empirical FDR control at a cutoff of 0.1.

## When to use

Use when processing DIA-lipidomics MS-DIAL or MZMine feature lists with multiplexed MS2 spectra and confident, FDR-controlled TG molecular-species annotations are needed for downstream differential analysis.

## When NOT to use

- The analyte class is not triacylglycerols — the README states 'DIATAGeR is for triacylglycerols only' (lipid='TAG')
- Input is raw vendor mass-spectrometry data rather than an exported MS-DIAL/MZMine feature list with MS2 spectra files
- Only MS1 precursor information is available with no MS2 spectra — TG identification and scoring depend on multiplexed MS2 fragment matching

## Inputs

- MS-DIAL alignment feature list (txt), e.g. Feature_List_Liver_MSDIAL.txt, or a generic-format feature list; MZMine lists must be reformatted to the generic format
- Centroid MS2 spectra folder in txt format (also supports msp, mgf, mgf_mzmine exported from MSDIAL/MZMine), e.g. Spectra_txt_Liver_MSDIAL/Centroid
- Optional list of TG standards (Standards.csv) for FDR/standards-recovery reporting

## Outputs

- DIADataObj feature-list object with attached, deisotoped/denoised MS2 spectra
- Target-database TG annotation file (e.g. Identified_TAG_Pos_September2024_1 in an ion-mode folder)
- ResultsTG.RDS: scored merged target/decoy TG annotations from ScoreLipids
- ResultsTG_FDR.RDS: FDR-filtered TG molecular-species annotations at cutoff 0.1, plus reported average species/molecular-species counts, AUC values, and standards recovered

## How to apply

Import the MS-DIAL alignment feature list (e.g. Feature_List_Liver_MSDIAL.txt) with measureFileImport (IonMode='Pos', ResultsFiletype='MSDIAL'), then attach centroid MS2 spectra via SpectraImport (spectra.file.type='txt', rttol=10; features without spectra are diverted to a 'Troubleshooting' folder). Clean spectra with RemoveIsotopes (ppmtol=15, low.intensity.cutoff=100, density.cutoff=TRUE, density.cutoff.value=1), then identify TGs with LipidIdentifier (lipid='TAG', ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, which.frag='any', default max.tails covering fatty acyls from 8.2 to 26.0 carbons/double bonds). Run the decoy identification, merge target and decoy annotations, and train the logistic-regression scorer with ScoreLipids (caret/pROC), then apply empirical target-decoy FDR correction with FDRCutoff (iteration=10, cutoff=0.1, rttol=10) against a Standards.csv file. The function reports average species counts, molecular-species counts, AUC values, number of standards recovered, and outputs the annotation file with the highest number of molecular species.

## Related tools

- **DIATAGeR** (R package providing the full TG annotation workflow: measureFileImport, SpectraImport, RemoveIsotopes, LipidIdentifier, ScoreLipids, FDRCutoff) — Velenosi-Lab/DIATAGeR
- **caret** (machine-learning framework used within ScoreLipids to train the logistic-regression target/decoy scorer)
- **pROC** (used with ScoreLipids/FDRCutoff to compute AUC values of the scoring model)
- **MS-DIAL** (source of the alignment feature list and exported centroid MS2 spectra consumed by measureFileImport and SpectraImport)

## Examples

```
ResultsTG_FDR <- FDRCutoff(identified.lipids = ResultsTG, standard.file = "Standards.csv", iteration=10, cutoff = 0.1, rttol=10); saveRDS(ResultsTG_FDR, file = "ResultsTG_FDR.RDS")
```

## Evaluation signals

- All 9 supplied TG standards are annotated at empirical FDR < 0.1 in the liver and plasma datasets, matching the reported benchmark
- FDRCutoff reports average species counts, molecular-species counts, and AUC values across 10 iterations, and returns the output file with the highest number of molecular species
- Features lacking spectra appear in the 'Troubleshooting' folder after SpectraImport rather than being silently dropped
- Saved objects ResultsTG.RDS and ResultsTG_FDR.RDS exist and contain the merged scored annotations and the FDR-filtered set respectively
- Identification used consistent parameters across LipidIdentifier and ScoreLipids (ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, which.frag='any', same max.tails list)

## Limitations

- DIATAGeR is restricted to triacylglycerols (lipid='TAG'); no other lipid classes are supported
- The density.cutoff.value parameter (default 1) should be 'Change[d] with caution' per the README
- Features with missing spectra are diverted to a 'Troubleshooting' folder and are not annotated
- The article's discussion section is absent from the source material, so no article-stated limitations or reproducibility gaps were extracted

## Evidence
