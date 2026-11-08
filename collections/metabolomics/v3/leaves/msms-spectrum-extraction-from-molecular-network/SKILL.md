---
name: msms-spectrum-extraction-from-molecular-network
description: Use when unannotated MS/MS features in a molecular-network subnetwork
  need their consensus spectra exported (e.g. as .mgf or .ms) for in-silico classification
  by tools like SIRIUS, starting from public GNPS/MassIVE untargeted metabolomics
  data.
when_to_use_negative:
- When the features are already annotated by spectral matching against the GNPS reference
  libraries (Level 2/3 annotations) and no in-silico classification is needed.
- When the target spectra are absent from the molecular network because of poor fragmentation
  or missing fragments during MS/MS acquisition — extraction cannot recover missing
  spectral pairs.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Source of the public untargeted metabolomics MS/MS data and reference libraries
    used to build the cross-tissue molecular network from which spectra are extracted.
- name: SIRIUS
  role: Downstream consumer of the exported .mgf/.ms consensus spectra, used to generate
    putative molecular formulae and compound-class (e.g. fatty-acyl carnitine) assignments.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:90753ad45df88274e017f803099c78d06dd08ffe3f8c951867b577bb34260b88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# msms-spectrum-extraction-from-molecular-network

## Summary

Extracts consensus MS/MS spectra for specific precursor m/z features (e.g. the unannotated spectra at m/z 266.138, 280.153, 294.169) from the cross-tissue molecular network built in the 3xTg/5xFAD multi-organ metabolomics study, and exports them as .mgf or .ms input files for downstream in-silico tools such as SIRIUS.

## When to use

Use when unannotated MS/MS features in a molecular-network subnetwork need their consensus spectra exported (e.g. as .mgf or .ms) for in-silico classification by tools like SIRIUS, starting from public GNPS/MassIVE untargeted metabolomics data.

## When NOT to use

- When the features are already annotated by spectral matching against the GNPS reference libraries (Level 2/3 annotations) and no in-silico classification is needed.
- When the target spectra are absent from the molecular network because of poor fragmentation or missing fragments during MS/MS acquisition — extraction cannot recover missing spectral pairs.

## Inputs

- Public untargeted metabolomics MS/MS data (GNPS/MassIVE repository) from the 3xTg multi-organ study
- Cross-tissue molecular network (MS/MS spectral matching graph) identifying subnetworks such as the carnitine subnetwork
- Precursor m/z targets, e.g. 266.138, 280.153, 294.169

## Outputs

- Consensus MS/MS spectra exported to SIRIUS-compatible .mgf or .ms files, each carrying precursor mass and MS/MS peak list

## How to apply

Retrieve the untargeted metabolomics MS/MS data for the 3xTg study from the public GNPS/MassIVE repository and locate the consensus MS/MS spectra for the features of interest (e.g. precursor masses m/z 266.138, 280.153, 294.169) within the carnitine subnetwork of the cross-tissue molecular network, which the paper builds by MS/MS spectral matching. Export each consensus spectrum to a SIRIUS-compatible format — an .mgf file or individual .ms files — carrying the precursor mass and its MS/MS peak list. For each exported spectrum, record precursor m/z and peak list so downstream tools can compute formulae and compound classes. Validate the extraction by confirming that each exported file contains the expected precursor m/z and a non-empty MS/MS peak list, and that the spectra fall within the targeted molecular-network subnetwork (here the carnitine subnetwork).

## Related tools

- **GNPS** (Source of the public untargeted metabolomics MS/MS data and reference libraries used to build the cross-tissue molecular network from which spectra are extracted.)
- **SIRIUS** (Downstream consumer of the exported .mgf/.ms consensus spectra, used to generate putative molecular formulae and compound-class (e.g. fatty-acyl carnitine) assignments.)

## Examples

```
sirius --input spectra.mgf --output sirius_results --c compound-class; sirius --input 266_138.ms 280_153.ms 294_169.ms --c compound-class
```

## Evaluation signals

- Each exported .mgf/.ms file contains the expected precursor m/z (e.g. 266.138, 280.153, 294.169) in its header.
- Each exported file has a non-empty MS/MS peak list.
- The extracted spectra belong to the intended subnetwork of the cross-tissue molecular network (here the carnitine subnetwork).
- The extracted consensus spectra are traceable back to the public GNPS/MassIVE untargeted metabolomics data for the 3xTg study.
- Downstream SIRIUS run accepts the exported files without format errors and returns candidate formulae/classes.

## Limitations

- Spectral annotation via GNPS reference libraries yields only Level 2 or 3 annotations under the Metabolomics Standards Initiative, so exported spectra may remain structurally unconfirmed.
- Some spectral pairs are missing from the cross-tissue molecular network due to poor fragmentation or missing fragments during MS/MS acquisition, so targeted spectra may be absent.
- Hundreds of differentially abundant molecules remain non-annotated, so extraction from a subnetwork does not guarantee a confident identification.

## Evidence

- [results] The three unannotated spectra were identified as possible carnitines because they fell within the carnitine subnetwork and were putatively classified as fatty-acyl carnitines via SIRIUS.: [evidence span withheld — anchor status: not_found]
- [discussion] Cross-tissue molecular networking is based on MS/MS spectral matching.: "Cross-tissue molecular networking is based on MS/MS spectral matching."
- [discussion] MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative.: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative"
- [discussion] it is possible that some spectral pairs are missing due to the poor fragmentation or missing fragments during the acquisition of the MS/MS spectra: "it is possible that some spectral pairs are missing due to the poor fragmentation or missing fragments during the acquisition of the MS/MS spectra"
- [intro] Via machine learning models24,25 and molecular networking26, we identify cross-tissue molecular dysregulation of carnitines, bile acids, B vitamins, N-acyl lipids, indoles, polyamines, and many other: "Via machine learning models24,25 and molecular networking26, we identify cross-tissue molecular dysregulation of carnitines, bile acids, B vitamins, N-acyl lipids, indoles, polyamines, and many other"
