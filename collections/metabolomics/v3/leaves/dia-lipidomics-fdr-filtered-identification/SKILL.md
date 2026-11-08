---
name: dia-lipidomics-fdr-filtered-identification
description: Use when DIA lipidomics data needs TG annotation to the molecular species
  level, e.g. MS-DIAL or MZMine alignment feature lists plus exported centroided MS2
  spectra requiring FDR-filtered identification.
when_to_use_negative:
- The lipid class of interest is not triacylglycerols — the package states 'DIATAGeR
  is for triacylglycerols only' (lipid='TAG')
- Data are DDA or otherwise lack multiplexed DIA MS2 spectra, since the method is
  designed for DIA-based lipidomics
- MS2 spectra are unavailable for a feature — those features are routed to the Troubleshooting
  folder rather than identified
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: DIATAGeR
  role: R package providing the full six-function TG annotation workflow (target/decoy
    identification, ML scoring, FDR cutoff)
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: measureFileImport
  role: Imports the MSDIAL Alignment or Generic feature list into the DIA data object
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: SpectraImport
  role: Matches centroid MS2 spectra (txt/msp/mgf) to features with a retention-time
    tolerance
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: RemoveIsotopes
  role: Deisotopes and denoises MS2 spectra via ppm and intensity/density cutoffs
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: LipidIdentifier
  role: Identifies TGs against the target database using MS1/MS2 precursors and fatty-acyl
    tail limits
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: ScoreLipids
  role: Performs decoy-database identification, merges target/decoy annotations, and
    applies the machine-learning score
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: FDRCutoff
  role: Applies the empirical FDR cutoff (default 0.1, 10 iterations) and outputs
    the highest molecular-species-count identification file
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: MS-DIAL
  role: Source of the alignment feature lists and MS2 spectra consumed by measureFileImport/SpectraImport
    (ResultsFiletype='MSDIAL')
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:d2353452765c122d50b6e7b3570a7c61b513193436e848d36c99c357f3596c5d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Velenosi-Lab/DIATAGeR
  tools:
  - DIATAGeR
  - FDRCutoff
  - LipidIdentifier
  - MS-DIAL
  - RemoveIsotopes
  - ScoreLipids
  - SpectraImport
  - measureFileImport
  - tidyverse
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# dia-lipidomics-fdr-filtered-identification

## Summary

End-to-end triacylglycerol (TG) annotation of DIA-based lipidomics data to the molecular species level using the DIATAGeR R package, culminating in decoy-based scoring and empirical FDR filtering of identifications.

## When to use

Use when DIA lipidomics data needs TG annotation to the molecular species level, e.g. MS-DIAL or MZMine alignment feature lists plus exported centroided MS2 spectra requiring FDR-filtered identification.

## When NOT to use

- The lipid class of interest is not triacylglycerols — the package states 'DIATAGeR is for triacylglycerols only' (lipid='TAG')
- Data are DDA or otherwise lack multiplexed DIA MS2 spectra, since the method is designed for DIA-based lipidomics
- MS2 spectra are unavailable for a feature — those features are routed to the Troubleshooting folder rather than identified

## Inputs

- MS-DIAL alignment feature list in txt format (e.g. Feature_List_Liver_MSDIAL.txt) or a generic-format feature list
- MS2 spectra exported from MS-DIAL as txt, msp, or mgf (or MZMine mgf with a generic-format feature list), stored in a folder such as Spectra_txt_Liver_MSDIAL/Centroid
- Optional: Standards CSV (Standards_File_Format_example.csv from MetaboLights MTBLS11927) for FDRCutoff standard reporting

## Outputs

- Identified_TAG_<IonMode>_<version>_<n> result file of TGs identified from the target database (e.g. Identified_TAG_Pos_September2024_1)
- Scored target-plus-decoy TG annotation object (ResultsTG) suitable for saveRDS
- FDR-filtered TG list (ResultsTG_FDR) with reported average species, molecular species, and AUC values per iteration
- A 'Troubleshooting' folder containing features with missing spectra

## How to apply

Run the six DIATAGeR functions in order: measureFileImport imports the feature list (MSDIAL Alignment or Generic format, IonMode='Pos' or 'Neg', ResultsFiletype='MSDIAL'); SpectraImport attaches MS2 spectra from txt, msp, or mgf files/folders with a retention-time tolerance (rttol=10 in the benchmark); RemoveIsotopes performs deisotoping and denoising with ppmtol=15, low.intensity.cutoff=100, density.cutoff=TRUE, density.cutoff.value=1; LipidIdentifier identifies TGs against the target database with ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, which.frag='any', and the default max.tails list '8.2, 9.0, 10.2, 11.0, 12.3, 13.1, 14.3, 15.3, 16.5, 17.3, 18.5, 19.5, 20.6, 21.5, 22.6, 23.0, 24.4, 25.0, 26.0'; ScoreLipids reruns identification against a decoy database, merges target and decoy annotations, and scores them with a machine-learning algorithm; finally FDRCutoff applies an empirical FDR cutoff=0.1 over iteration=10 and outputs the file with the highest number of molecular species. Rationale: DIA produces multiplexed MS2 spectra that challenge TG identification, so target-decoy scoring plus FDR filtering controls false annotation at the molecular-species level.

## Related tools

- **DIATAGeR** (R package providing the full six-function TG annotation workflow (target/decoy identification, ML scoring, FDR cutoff)) — https://github.com/Velenosi-Lab/DIATAGeR
- **measureFileImport** (Imports the MSDIAL Alignment or Generic feature list into the DIA data object) — https://github.com/Velenosi-Lab/DIATAGeR
- **SpectraImport** (Matches centroid MS2 spectra (txt/msp/mgf) to features with a retention-time tolerance) — https://github.com/Velenosi-Lab/DIATAGeR
- **RemoveIsotopes** (Deisotopes and denoises MS2 spectra via ppm and intensity/density cutoffs) — https://github.com/Velenosi-Lab/DIATAGeR
- **LipidIdentifier** (Identifies TGs against the target database using MS1/MS2 precursors and fatty-acyl tail limits) — https://github.com/Velenosi-Lab/DIATAGeR
- **ScoreLipids** (Performs decoy-database identification, merges target/decoy annotations, and applies the machine-learning score) — https://github.com/Velenosi-Lab/DIATAGeR
- **FDRCutoff** (Applies the empirical FDR cutoff (default 0.1, 10 iterations) and outputs the highest molecular-species-count identification file) — https://github.com/Velenosi-Lab/DIATAGeR
- **MS-DIAL** (Source of the alignment feature lists and MS2 spectra consumed by measureFileImport/SpectraImport (ResultsFiletype='MSDIAL'))

## Examples

```
ResultsTG <- ScoreLipids(DIADataObj = DIA_Pos, format = "MSDIAL", lipid = "TAG", ion.mode = "Pos", ppmtol = 15, rttol = 5, ms1.precursors = 1, ms2.precursors = 1, spectra.file.type = "txt", spectra.file = "Spectra_txt_Liver_MSDIAL/Centroid"); ResultsTG_FDR <- FDRCutoff(identified.lipids = ResultsTG, iteration = 10, cutoff = 0.1, rttol = 10)
```

## Evaluation signals

- All six functions ran in sequence and each wrote its expected artifact (Pos/Neg result folder, ResultsTG.RDS, ResultsTG_FDR.RDS)
- FDRCutoff reports average species, molecular-species, and AUC values across iteration=10, and the final file is the one with the highest molecular-species count
- Features with no matched spectra appear in the Troubleshooting folder rather than silently dropping out
- Identified TG annotations carry fatty-acyl compositions consistent with the max.tails carbon/double-bond limits (e.g. 8:2 up to 26:0)
- Benchmark reproducibility: even-carbon fatty-acyl percentage differences vs MS-DIAL reproduce ~18% (liver) and ~12% (plasma) when the same parameters are used

## Limitations

- density.cutoff.value (default 1) should be 'Change with caution' per the README
- Features with missing spectra are excluded from identification (routed to a Troubleshooting folder)
- MZMine inputs require the feature list to be reformatted to match the generic format before use
- FDRCutoff's standard-based reporting requires a standard TGs CSV; without it only species/molecular-species/AUC averages are reported

## Evidence

- [readme] DIATAGeR includes 6 functions: measureFileImport, SpectraImport, RemoveIsotopes, LipidIdentifier, ScoreLipids and FDRCutoff.: "DIATAGeR includes 6 functions: measureFileImport, SpectraImport, RemoveIsotopes, LipidIdentifier, ScoreLipids and FDRCutoff."
- [readme] A separate TG identification is performed with the decoy database, then the targeted and decoy annotations are merged and scored using the machine learning algorithm.: "A separate TG identification is performed with the decoy database, then the targeted and decoy annotations are merged and scored using the machine learning algorithm."
- [readme] The function outputs the file containing the highest number of molecular species.: "The function outputs the file containing the highest number of molecular species."
- [readme] DIA generates multiplexed tandem mass spectra (MS2) that pose challenges to identifying TGs.: "However, DIA generates multiplexed tandem mass spectra (MS2) that pose challenges to identifying TGs."
- [readme] Feature list (txt format) can be an MS-DIAL alignment file or generic format . MS2 spectra can be in txt, msp and mgf format (exported from MSDIAL).: "Feature list (txt format) can be an MS-DIAL alignment file or generic format . MS2 spectra can be in txt, msp and mgf format (exported from MSDIAL)."
