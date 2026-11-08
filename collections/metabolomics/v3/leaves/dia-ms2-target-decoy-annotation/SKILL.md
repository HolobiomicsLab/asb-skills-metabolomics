---
name: dia-ms2-target-decoy-annotation
description: Use when DIA MS2 lipidomics features need molecular-species-level TG
  annotation with confidence estimation, i.e. decoy-based FDR control is required
  to suppress false positives from multiplexed MS2 interference.
when_to_use_negative:
- The data are not triacylglycerols — DIATAGeR is for TGs only (lipid="TAG") and will
  not annotate other lipid classes
- Input is raw LC-MS vendor data rather than an MS-DIAL/MZMine-derived feature list
  plus exported MS2 spectra (txt/msp/mgf); the upstream measureFileImport/SpectraImport
  steps must be run first
- Acquisition is data-dependent rather than DIA, where multiplexed MS2 spectra are
  the core challenge this decoy-scoring design addresses
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DIATAGeR
  role: R package providing the target-decoy TG annotation, scoring, and FDR workflow
    (allDecoys, DecoyIdentifier, ScoreLipids, FDRCutoff)
  repo: Velenosi-Lab/DIATAGeR
- name: allDecoys
  role: Generates the decoy lipid database from identified TGs prior to decoy searching
  repo: Velenosi-Lab/DIATAGeR
- name: DecoyIdentifier
  role: Runs TG identification against the decoy database with the same search parameters
    as the target search (ppmtol=15, rttol=5, ms1/ms2.precursors=1)
  repo: Velenosi-Lab/DIATAGeR
- name: ScoreLipids
  role: Merges target and decoy annotations and scores them with the logistic regression
    machine learning algorithm
  repo: Velenosi-Lab/DIATAGeR
- name: FDRCutoff
  role: Computes the empirical target-decoy FDR (cutoff 0.1, 10 iterations) and outputs
    the list with the highest number of molecular species
  repo: Velenosi-Lab/DIATAGeR
- name: caret
  role: Provides the machine learning (logistic regression) backend used to score
    merged target/decoy annotations
- name: pROC
  role: Used for AUC computation in scoring/FDR QC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:d9b72f12490ea1a99dd6f217da483261e31736c9a796eda7f80d20b8aa48a7d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Velenosi-Lab/DIATAGeR
  tools:
  - DIATAGeR
  - DecoyIdentifier
  - FDRCutoff
  - ScoreLipids
  - allDecoys
  - caret
  - pROC
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# dia-ms2-target-decoy-annotation

## Summary

Target-decoy FDR-based confidence scoring of triacylglycerol (TG) annotations in DIA lipidomics, as implemented in DIATAGeR. It merges target and decoy TG identifications, scores them with a logistic regression machine learning model, and applies an empirical FDR cutoff to limit false positives from ion interference.

## When to use

Use when DIA MS2 lipidomics features need molecular-species-level TG annotation with confidence estimation, i.e. decoy-based FDR control is required to suppress false positives from multiplexed MS2 interference.

## When NOT to use

- The data are not triacylglycerols — DIATAGeR is for TGs only (lipid="TAG") and will not annotate other lipid classes
- Input is raw LC-MS vendor data rather than an MS-DIAL/MZMine-derived feature list plus exported MS2 spectra (txt/msp/mgf); the upstream measureFileImport/SpectraImport steps must be run first
- Acquisition is data-dependent rather than DIA, where multiplexed MS2 spectra are the core challenge this decoy-scoring design addresses

## Inputs

- DIADataObj feature list (MSDIAL alignment, Generic, or formatted MZMine export) with imported centroid MS2 spectra (txt/msp/mgf/mgf_mzmine) attached by SpectraImport
- Decoy TG database produced by allDecoys()
- Optional standards list CSV (Standards.csv) for FDRCutoff validation

## Outputs

- ResultsTG.RDS — merged target+decoy TG annotations scored by the logistic regression model
- ResultsTG_FDR.RDS — final annotation list passing the empirical FDR cutoff (default 0.1), retaining the output with the highest number of molecular species
- Reported average number of species, molecular species, and AUC values across FDRCutoff iterations

## How to apply

First generate the decoy TG database with allDecoys(Identified, version, max.tails, exact.tails, ion.mode), then run DecoyIdentifier(DIADataObj, lipid="TAG", ion.mode, format, ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, max.tails="8.2, 9.0, ..., 26.0", exact.tails=NULL) to perform identification against decoys. The scoring mechanism works by performing a separate TG identification with the decoy database, merging the target and decoy annotations, and scoring the combined set with the machine learning algorithm (logistic regression from caret, with pROC used for AUC). Call ScoreLipids(DIADataObj, format="MSDIAL", lipid="TAG", ion.mode, version="September2024", ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, spectra.file.type="txt", spectra.file="Spectra_txt_Liver_MSDIAL/Centroid") to obtain the merged scored table (ResultsTG), then apply FDRCutoff(identified.lipids=ResultsTG, standard.file="Standards.csv", iteration=10, cutoff=0.1, rttol=10) to compute empirical target-decoy FDR over 10 iterations and keep the annotation list with the highest number of molecular species. Save ResultsTG.RDS and the FDR-filtered ResultsTG_FDR.RDS with saveRDS.

## Related tools

- **DIATAGeR** (R package providing the target-decoy TG annotation, scoring, and FDR workflow (allDecoys, DecoyIdentifier, ScoreLipids, FDRCutoff)) — Velenosi-Lab/DIATAGeR
- **allDecoys** (Generates the decoy lipid database from identified TGs prior to decoy searching) — Velenosi-Lab/DIATAGeR
- **DecoyIdentifier** (Runs TG identification against the decoy database with the same search parameters as the target search (ppmtol=15, rttol=5, ms1/ms2.precursors=1)) — Velenosi-Lab/DIATAGeR
- **ScoreLipids** (Merges target and decoy annotations and scores them with the logistic regression machine learning algorithm) — Velenosi-Lab/DIATAGeR
- **FDRCutoff** (Computes the empirical target-decoy FDR (cutoff 0.1, 10 iterations) and outputs the list with the highest number of molecular species) — Velenosi-Lab/DIATAGeR
- **caret** (Provides the machine learning (logistic regression) backend used to score merged target/decoy annotations)
- **pROC** (Used for AUC computation in scoring/FDR QC)

## Examples

```
ResultsTG <- ScoreLipids(DIADataObj = DIA_Pos, format = "MSDIAL", lipid = "TAG", ion.mode = "Pos", version = "September2024", ppmtol = 15, rttol = 5, ms1.precursors = 1, ms2.precursors = 1, spectra.file.type = "txt", spectra.file = "Spectra_txt_Liver_MSDIAL/Centroid"); ResultsTG_FDR <- FDRCutoff(identified.lipids = ResultsTG, standard.file = "Standards.csv", iteration = 10, cutoff = 0.1, rttol = 10)
```

## Evaluation signals

- All listed TG standards in Standards.csv are found at empirical FDR < 0.1 (cutoff=0.1, rttol=10 between standard and result retention times)
- FDRCutoff reports average species counts, molecular-species counts, and AUC values across 10 iterations as QC of the scored table
- The retained annotation list is the one with the highest number of molecular species among tested FDR cutoffs
- Decoy search used the identical LipidIdentifier parameters as the target search (ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, same max.tails/exact.tails), ensuring a fair target-decoy comparison
- Both ResultsTG.RDS and ResultsTG_FDR.RDS objects exist and were written by saveRDS

## Limitations

- DIATAGeR is limited to triacylglycerols only (lipid="TAG"); other lipid classes are out of scope
- The Discussion section was absent from the source extract, so reported limitations/reproducibility caveats from the paper text are unavailable
- Standards file is optional — without it, FDRCutoff cannot report the number of standards found or the FDR required to find all standards, weakening validation
- Features with missing spectra are diverted to a 'Troubleshooting' folder and are not scored

## Evidence

- [methods] decoy database generation: [evidence span withheld — anchor status: not_found]
