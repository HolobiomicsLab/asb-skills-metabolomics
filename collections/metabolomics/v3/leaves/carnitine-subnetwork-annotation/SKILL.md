---
name: carnitine-subnetwork-annotation
description: Use when hundreds of non-annotated, differentially abundant MS/MS features
  fall inside a molecular-network subnetwork with known annotated neighbors (e.g.
  carnitines), and you need putative class-level identities via in-silico tools like
  SIRIUS.
when_to_use_negative:
- When the MS/MS spectra already have Level 2 or 3 annotations via spectral matching
  against the public GNPS reference libraries — subnetwork inference plus SIRIUS is
  only needed for non-annotated features.
- When a candidate feature's precursor is not a member of a class-coherent molecular-network
  subnetwork (e.g. carnitine subnetwork); SIRIUS classification alone without subnetwork
  context is weaker and not the workflow described here.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Used to build the cross-tissue molecular network and retrieve/store the consensus
    MS/MS spectra (via GNPS/MassIVE public repository) for the unannotated features
- name: SIRIUS
  role: Runs in-silico formula and compound-class prediction on the exported consensus
    MS/MS spectra to putatively classify them as fatty-acyl carnitines
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:11d1ad4f19c574255ecd6a156bf2fbe4d0962ba74600d52cb5a83b49de4ddc70
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# carnitine-subnetwork-annotation

## Summary

Annotates unannotated MS/MS spectra by leveraging membership in the carnitine subnetwork of a cross-tissue molecular network plus SIRIUS compound-class prediction, e.g. assigning the 3xTg spectra at m/z 266.138, 280.153, and 294.169 as putative fatty-acyl carnitines.

## When to use

Use when hundreds of non-annotated, differentially abundant MS/MS features fall inside a molecular-network subnetwork with known annotated neighbors (e.g. carnitines), and you need putative class-level identities via in-silico tools like SIRIUS.

## When NOT to use

- When the MS/MS spectra already have Level 2 or 3 annotations via spectral matching against the public GNPS reference libraries — subnetwork inference plus SIRIUS is only needed for non-annotated features.
- When a candidate feature's precursor is not a member of a class-coherent molecular-network subnetwork (e.g. carnitine subnetwork); SIRIUS classification alone without subnetwork context is weaker and not the workflow described here.

## Inputs

- Consensus MS/MS spectra (precursor m/z + peak lists) for unannotated features within the carnitine subnetwork of the cross-tissue molecular network
- Cross-tissue molecular network built from the 3xTg untargeted metabolomics dataset (2,271 samples; brain, serum, liver, cecal content, feces) via GNPS spectral matching
- SIRIUS-compatible input files (.mgf or individual .ms files) carrying precursor mass and MS/MS peak lists

## Outputs

- Summary table of precursor m/z, top molecular formula, top compound class, and whether the class is fatty-acyl carnitine for each unannotated spectrum
- Putative class-level annotations (e.g. fatty-acyl carnitine) for spectra at m/z 266.138, 280.153, and 294.169

## How to apply

First build a cross-tissue molecular network from untargeted metabolomics MS/MS data (e.g. via GNPS spectral matching) and identify subnetworks whose annotated members belong to a recognizable chemical class such as carnitines. Next, retrieve the consensus MS/MS spectra of the unannotated features inside that subnetwork (here the features with precursor m/z 266.138, 280.153, and 294.169) from the public GNPS/MassIVE repository and export them in a SIRIUS-compatible format (MGF or individual .ms files) carrying precursor mass and peak lists. Run SIRIUS with the compound-class/CSI:FingerID module to generate putative molecular formulae and structural candidates for each precursor, then inspect the candidate lists and record whether top-ranked assignments fall in the fatty-acyl carnitine class. Finally, aggregate the classifications into a table of precursor m/z, top formula, and top compound class, and corroborate the annotation using subnetwork context (differential abundance across tissues in the 3xTg model) rather than treating SIRIUS output as a fully validated Level 1 structure.

## Related tools

- **GNPS** (Used to build the cross-tissue molecular network and retrieve/store the consensus MS/MS spectra (via GNPS/MassIVE public repository) for the unannotated features)
- **SIRIUS** (Runs in-silico formula and compound-class prediction on the exported consensus MS/MS spectra to putatively classify them as fatty-acyl carnitines)

## Examples

```
sirius --input spectra.mgf --output sirius_results --compound-class --csi-finger-id
```

## Evaluation signals

- SIRIUS returns a compound-class assignment consistent with fatty-acyl carnitine for each of the spectra at m/z 266.138, 280.153, and 294.169.
- Each targeted precursor falls within the carnitine subnetwork of the cross-tissue molecular network built from the 3xTg study.
- The exported spectra carry both precursor mass and MS/MS peak lists in SIRIUS-compatible format (.mgf or .ms) without losing peaks.
- The aggregated summary table lists precursor m/z, top formula, and top compound class for all three spectra, matching the paper's statement that they were 'putatively classified as fatty-acyl carnitines via SIRIUS'.
- Resulting annotations are reported as putative class-level (fatty-acyl carnitine) assignments rather than fully validated Level 1 structures, consistent with the MSI Level 2 or 3 framing.

## Limitations

- Annotations remain putative class-level assignments, not fully validated structures; MS/MS annotation via spectral matching against the public GNPS reference libraries is only MSI Level 2 or 3.
- Spectra may be absent or poorly characterized in the cross-tissue molecular network due to poor fragmentation or missing fragments during MS/MS acquisition.
- Hundreds of differentially abundant molecules remain non-annotated, so this workflow does not close the full annotation gap in the dataset.
- Feature detection across reference datasets can be influenced by study-specific extraction methods, workflows, and instrumentation; lack of detection does not necessarily mean absence in tissue.

## Evidence

- [abstract] Systems-level analyses of 3xTg and 5xFAD mice reveal clusters of dysregulated molecular classes across tissues including carnitines, bile acids, B vitamins, and neurotransmitters: "Systems-level analyses of 3xTg and 5xFAD mice reveal clusters of dysregulated molecular classes across tissues including carnitines"
- [results] The three unannotated spectra were identified as possible carnitines because they fell within the carnitine subnetwork and were putatively classified as fatty-acyl carnitines via SIRIUS.: [evidence span withheld — anchor status: not_found]
- [discussion] we recognize that hundreds of non-annotated molecules are present and differentially abundant between WT and AD mouse models: "we recognize that hundreds of non-annotated molecules are present and differentially abundant between WT and AD mouse models"
- [discussion] MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative"
- [discussion] it is possible that some spectral pairs are missing due to the poor fragmentation or missing fragments during the acquisition of the MS/MS spectra: "it is possible that some spectral pairs are missing due to the poor fragmentation or missing fragments during the acquisition of the MS/MS spectra"
