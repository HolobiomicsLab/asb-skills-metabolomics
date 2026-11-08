---
name: decoy-based-fdr-estimation
description: Use when DIA-lipidomics TG annotations need empirical confidence estimation
  — i.e., after LipidIdentifier target identification and a parallel DecoyIdentifier/allDecoys
  run, when decoy and target hits must be scored and filtered at a fixed FDR (e.g.,
  0.1) with standards-based validation.
when_to_use_negative:
- The lipid class is not triacylglycerols — DIATAGeR is explicitly for TGs only ('DIATAGeR
  is for triacylglycerols only', lipid='TAG')
- Input is raw spectra or an unannotated feature list — target identification (LipidIdentifier)
  and spectra import (SpectraImport) must run first before decoy scoring and FDR filtering
- Data are not DIA-based lipidomics feature lists / MS2 spectra from MS-DIAL, Generic-format,
  or MZMine exports
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ScoreLipids
  role: Merges target and decoy TG annotations and trains the machine learning (logistic
    regression, caret/pROC) scorer used before FDR filtering
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: FDRCutoff
  role: Applies the target-decoy empirical FDR cutoff (iteration=10, cutoff=0.1, rttol=10)
    and reports standards recovery, species counts, and AUC values
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: DecoyIdentifier/allDecoys
  role: Performs the parallel decoy-database TG identification that supplies the empirical
    null for FDR estimation
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: caret / pROC
  role: R packages used by ScoreLipids to train and evaluate the logistic regression
    scoring model on target vs decoy annotations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:d43893932a74b0d95de2c398324a41d63f57b5f63e9ebe7c4423d30ff82f7d7d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Velenosi-Lab/DIATAGeR
  tools:
  - DecoyIdentifier
  - DecoyIdentifier/allDecoys
  - FDRCutoff
  - LipidIdentifier
  - RemoveIsotopes
  - ScoreLipids
  - SpectraImport
  - allDecoys
  - caret
  - caret / pROC
  - measureFileImport
  - pROC
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# decoy-based-fdr-estimation

## Summary

Target-decoy empirical FDR estimation for DIA-lipidomics triacylglycerol annotation using DIATAGeR: decoy-database identifications are merged with target annotations, scored by a caret/pROC logistic regression model (ScoreLipids), and filtered to a pre-determined empirical FDR cutoff (FDRCutoff) against a TG standards list.

## When to use

Use when DIA-lipidomics TG annotations need empirical confidence estimation — i.e., after LipidIdentifier target identification and a parallel DecoyIdentifier/allDecoys run, when decoy and target hits must be scored and filtered at a fixed FDR (e.g., 0.1) with standards-based validation.

## When NOT to use

- The lipid class is not triacylglycerols — DIATAGeR is explicitly for TGs only ('DIATAGeR is for triacylglycerols only', lipid='TAG')
- Input is raw spectra or an unannotated feature list — target identification (LipidIdentifier) and spectra import (SpectraImport) must run first before decoy scoring and FDR filtering
- Data are not DIA-based lipidomics feature lists / MS2 spectra from MS-DIAL, Generic-format, or MZMine exports

## Inputs

- Scored/combined target + decoy TG annotation list (ResultsTG object from ScoreLipids, saved as RDS)
- DIATAGeR DIADataObj (e.g., DIA_Pos from measureFileImport/SpectraImport) for the decoy identification run
- MS2 spectra folder for all samples (e.g., Spectra_txt_Liver_MSDIAL/Centroid, txt/msp/mgf format)
- Standards file listing known TG standards (Standards.csv, Standards_File_Format_example.csv format)
- Feature list parameters: format='MSDIAL' or 'Generic', lipid='TAG', ion.mode, ppmtol=15, rttol=5, max.tails fatty acyl range 8.2–26.0 carbons

## Outputs

- ResultsTG_FDR.RDS — TG list passing the pre-determined empirical FDR (e.g., 0.1), containing the file with the highest number of molecular species
- Reported metrics: average species counts, molecular species counts, AUC values over iterations (default 10), number of standards found, and the FDR required to find all standards

## How to apply

After LipidIdentifier identifies TGs against the target database (e.g., lipid='TAG', ion.mode='Pos', format='MSDIAL', ppmtol=15, rttol=5), run a separate parallel identification against a decoy database with DecoyIdentifier/allDecoys, merge the target and decoy annotations, and train a logistic regression scorer with ScoreLipids (using caret/pROC; spectra.file points to the all-samples spectra folder, spectra.file.type='txt', version='September2024'). Then call FDRCutoff on the combined scored annotations with iteration=10, cutoff=0.1, and rttol=10 against the standards file (Standards.csv), which reports average species counts, molecular species counts, and AUC values over iterations, plus the number of standards recovered and the FDR needed to recover all standards. FDRCutoff outputs the result file containing the highest number of molecular species; save it as an RDS (e.g., ResultsTG_FDR.RDS). The rationale is that decoy hits provide an empirical null for estimating the false discovery rate of DIA-derived TG molecular-species annotations, since multiplexed MS2 spectra complicate identification.

## Related tools

- **ScoreLipids** (Merges target and decoy TG annotations and trains the machine learning (logistic regression, caret/pROC) scorer used before FDR filtering) — https://github.com/Velenosi-Lab/DIATAGeR
- **FDRCutoff** (Applies the target-decoy empirical FDR cutoff (iteration=10, cutoff=0.1, rttol=10) and reports standards recovery, species counts, and AUC values) — https://github.com/Velenosi-Lab/DIATAGeR
- **DecoyIdentifier/allDecoys** (Performs the parallel decoy-database TG identification that supplies the empirical null for FDR estimation) — https://github.com/Velenosi-Lab/DIATAGeR
- **caret / pROC** (R packages used by ScoreLipids to train and evaluate the logistic regression scoring model on target vs decoy annotations)

## Examples

```
ResultsTG_FDR <- FDRCutoff(identified.lipids = ResultsTG, standard.file = "Standards.csv", iteration=10, cutoff = 0.1, rttol=10)
```

## Evaluation signals

- All 9 supplied TG standards are annotated at FDR < 0.1 on the liver dataset, matching the paper's reported benchmark (ResultsTG_FDR.RDS)
- FDRCutoff reports average species counts, molecular species counts, and AUC values across the 10 iterations
- The decoy identification runs with the same parameters as the target run (ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, max.tails 8.2–26.0) so decoy hits form a valid empirical null
- FDRCutoff returns the output file with the highest number of molecular species at the chosen cutoff
- Features lacking spectra should have been diverted to the 'Troubleshooting' folder upstream; a large fraction there indicates broken inputs to the scoring step

## Limitations

- FDR filtering depends on a supplied standards list; without Standards.csv, FDRCutoff still outputs the highest-molecular-species file but cannot report standards recovery or the FDR needed to find all standards
- The decoy identification and scoring must repeat the exact LipidIdentifier parameters (version='September2024', ppmtol, rttol, max.tails); mismatched parameters invalidate the empirical FDR
- FDRCutoff's rttol (default 10 s) governs matching between standards retention times and results — overly tight or loose tolerances change which standards are counted
- Density-based spectral denoising (density.cutoff.value=1) should be changed with caution, as it affects the spectra feeding the scorer

## Evidence

- [readme] A separate TG identification is performed with the decoy database, then the targeted and decoy annotations are merged and scored using the machine learning algorithm.: "A separate TG identification is performed with the decoy database, then the targeted and decoy annotations are merged and scored using the machine learning algorithm."
- [readme] The average number of species, molecular species and AUC values after multiple iterations are reported. The function outputs the file containing the highest number of molecular species.: "The average number of species, molecular species and AUC values after multiple iterations are reported. The function outputs the file containing the highest number of molecular species."
- [readme] If a list of standard TGs is provided, the number of standards found and FDR required to find all standards are reported.: "If a list of standard TGs is provided, the number of standards found and FDR required to find all standards are reported."
- [intro] DIA generates multiplexed tandem mass spectra (MS2) that pose challenges to identifying TGs: "DIA generates multiplexed tandem mass spectra (MS2) that pose challenges to identifying TGs"
- [other] Running the full DIATAGeR pipeline on the deposited liver and plasma DIA-lipidomics data annotates all 9 TG standards at an FDR below 0.1, matching the reported benchmark.: [evidence span withheld — anchor status: not_found]
