---
name: dia-lipidomics-feature-list-import
description: Use when starting a DIATAGeR triacylglycerol annotation run on data-independent
  acquisition lipidomics data and a feature list must be loaded as the first pipeline
  step.
when_to_use_negative:
- The input is already a DIATAGeR DIADataObj or a completed annotation table — import
  would discard prior processing.
- The data are MS2 spectra only (txt, msp, mgf) with no feature list; those belong
  in SpectraImport, not measureFileImport.
- The lipidomics data are not DIA-based or the target lipid class is not triacylglycerols;
  DIATAGeR handles TGs only.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DIATAGeR
  role: R package providing measureFileImport, the first of its six workflow functions,
    to read MS-DIAL or generic feature lists into a DIADataObj.
  repo: Velenosi-Lab/DIATAGeR
- name: devtools
  role: Used to install DIATAGeR from GitHub before the feature list can be imported.
  repo: Velenosi-Lab/DIATAGeR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:8a66eb5e31616ceb7ec05b09ead35f06fc247dfdfca2319635f6e5532c9fa280
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
  - devtools
  - measureFileImport
  - pROC
  - tidyverse
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# dia-lipidomics-feature-list-import

## Summary

Import a DIA lipidomics feature list (MS-DIAL alignment or generic-format txt, or MZMine-formatted features) into DIATAGeR via measureFileImport to create the DIADataObj that anchors all downstream TG annotation.

## When to use

Use when starting a DIATAGeR triacylglycerol annotation run on data-independent acquisition lipidomics data and a feature list must be loaded as the first pipeline step.

## When NOT to use

- The input is already a DIATAGeR DIADataObj or a completed annotation table — import would discard prior processing.
- The data are MS2 spectra only (txt, msp, mgf) with no feature list; those belong in SpectraImport, not measureFileImport.
- The lipidomics data are not DIA-based or the target lipid class is not triacylglycerols; DIATAGeR handles TGs only.

## Inputs

- MS-DIAL alignment feature list in txt format (e.g. Feature_List_Liver_MSDIAL.txt)
- Generic-format feature list txt
- MZMine feature list reformatted to match the generic format

## Outputs

- DIADataObj object storing the imported feature list (e.g. DIA_Pos) ready for SpectraImport

## How to apply

Call measureFileImport with the feature list path as `filename`, setting `IonMode` to "Pos" or "Neg" to match the acquisition mode and `ResultsFiletype` to "MSDIAL" for an MS-DIAL alignment txt or "Generic" for the generic format (MZMine output must first be reformatted to the generic format). The function returns a DIADataObj object that stores the feature list and is passed by name to SpectraImport, RemoveIsotopes, and LipidIdentifier. Import must precede MS2 spectra attachment because SpectraImport matches spectra to features in the DIADataObj using a retention-time tolerance (rttol, e.g. 10 s in the example workflow). Verify the loaded object exists and that the ion mode and file type arguments reflect the actual deposited data (e.g. MetaboLights MTBLS11927 files like Feature_List_Liver_MSDIAL.txt).

## Related tools

- **DIATAGeR** (R package providing measureFileImport, the first of its six workflow functions, to read MS-DIAL or generic feature lists into a DIADataObj.) — Velenosi-Lab/DIATAGeR
- **devtools** (Used to install DIATAGeR from GitHub before the feature list can be imported.) — Velenosi-Lab/DIATAGeR

## Examples

```
DIA_Pos <- measureFileImport(filename = "Feature_List_Liver_MSDIAL.txt", IonMode = "Pos", ResultsFiletype = "MSDIAL")
```

## Evaluation signals

- measureFileImport completes without error and the returned object (e.g. DIA_Pos) is non-empty and usable as DIADataObj in SpectraImport.
- IonMode and ResultsFiletype arguments match the actual deposited data (e.g. "Pos" and "MSDIAL" for MS-DIAL alignment files from MetaboLights MTBLS11927).
- No 'Troubleshooting' folder is created at this stage — that output is produced by SpectraImport, not by feature list import.
- Downstream LipidIdentifier calls succeed only if the DIADataObj was imported with the same format (MSDIAL or Generic) that the data actually are.

## Limitations

- MZMine feature lists must be reformatted to match the generic format before import; raw MZMine alignment files are not accepted directly.
- MS2 spectra for import must be exported from MS-DIAL (txt, msp, mgf) or MZMine (mgf_mzmine); other spectral formats are not supported.
- The ion mode (Pos/Neg) and file type (MSDIAL/Generic) must be specified correctly by the user; mismatched settings will misparse the feature list.

## Evidence

- [readme] measureFileImport reads in the feature list (MSDIAL Alignment file or Generic Format).: "measureFileImport reads in the feature list (MSDIAL Alignment file or Generic Format)."
- [readme] Feature list (txt format) can be an MS-DIAL alignment file or generic format.: "Feature list (txt format) can be an MS-DIAL alignment file or generic format . MS2 spectra can be in txt, msp and mgf format"
