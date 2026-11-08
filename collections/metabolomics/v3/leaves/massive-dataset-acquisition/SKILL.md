---
name: massive-dataset-acquisition
description: Use when a published molecular-cartography result must be reproduced
  from a public repository deposit (MassIVE/GNPS accession such as MSV000079825) and
  raw LC–MS runs are needed as Optimus input.
when_to_use_negative:
- The feature quantification table already exists (e.g., features_quantification_matrix.csv
  from a prior Optimus run) — skip acquisition and proceed to annotation/'ili mapping.
- The analysis targets MS/MS spectral library matching at GNPS rather than reprocessing
  raw LC–MS feature detection across runs.
- The dataset deposit lacks raw mzML/mzXML runs or the experimental design needed
  to map runs to plate positions/volunteers.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: Optimus
  role: Target processing workflow into which the downloaded MassIVE runs are loaded
    for feature detection, alignment, and quantification
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the FeatureFinderMetabo-based feature detection algorithms that Optimus
    runs on the acquired mzML/mzXML data (20 ppm m/z tolerance for QTOF data)
  repo: http://www.openms.de
- name: KNIME
  role: Workflow management system that executes the Optimus pipeline on the downloaded
    dataset
  repo: https://www.knime.org
- name: pyopenms
  role: Python dependency installed alongside six, pandas, protobuf, pymspec to support
    the Optimus environment
- name: '''ili'
  role: Downstream app in which the acquired dataset's reprocessed features are visualized
    as spatial maps over the 96-well-plate image
  repo: https://github.com/ili-toolbox/ili
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:a2497bd71c6e14af5597f226e0c2f28b2bae575e5ed1d58eac32d7b7cee8972d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - '''ili'
  - KNIME
  - OpenMS
  - Optimus
  - OptimusViewer
  - Python
  - pyopenms
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# massive-dataset-acquisition

## Summary

Locates and downloads publicly deposited LC–MS untargeted metabolomics runs (e.g., mzML/mzXML for MassIVE accession MSV000079825) so they can be reprocessed end-to-end with the Optimus KNIME workflow and mapped in 'ili, enabling independent reproduction of published molecular maps.

## When to use

Use when a published molecular-cartography result must be reproduced from a public repository deposit (MassIVE/GNPS accession such as MSV000079825) and raw LC–MS runs are needed as Optimus input.

## When NOT to use

- The feature quantification table already exists (e.g., features_quantification_matrix.csv from a prior Optimus run) — skip acquisition and proceed to annotation/'ili mapping.
- The analysis targets MS/MS spectral library matching at GNPS rather than reprocessing raw LC–MS feature detection across runs.
- The dataset deposit lacks raw mzML/mzXML runs or the experimental design needed to map runs to plate positions/volunteers.

## Inputs

- MassIVE dataset accession (e.g., MSV000079825) and its file listing/metadata
- Downloaded LC–MS runs in mzML or mzXML format (phone-swab 96-well-plate rows B–H plus blanks row A)
- Experimental design / sample-sheet mapping wells to volunteers and sample types
- Molecule list of interest for downstream mz-RT annotation (e.g., minoxidil m/z 210.136, trehalose 6-tripalmitate m/z 603.335)

## Outputs

- Local corpus of raw LC–MS runs (mzML/mzXML) organized per well/plate position
- Optimus-ready input directory consumed by the KNIME workflow
- Downstream Optimus outputs (features_quantification_matrix.csv, ili_input.csv) after processing
- Verified inventory of runs vs. deposit metadata (blank row A included)

## How to apply

Resolve the study accession (e.g., MSV000079825 on MassIVE) and download the LC–MS runs in mzML/mzXML format, keeping the experimental layout intact — for the phone-swab 96-well-plate study that means rows B–H (eight hand, two phone-front, two phone-back wells per volunteer) plus blank row A. Before processing, prepare the environment the Optimus README requires: KNIME Analytics Platform, Python 2.7 with the modules installed via 'pip install six pandas protobuf pymspec pyopenms', and the latest Optimus release zip from the GitHub Releases page (about 2.5 GB of components). Feed the downloaded runs into Optimus's FeatureFinderMetabo-based (OpenMS) detection and alignment with paper-recommended settings — m/z tolerance 20 ppm for QTOF-type data, RT tolerance ~30 s for a 10-min UHPLC gradient. Preserve the blank wells so row A can later be checked for absence of the target features (minoxidil m/z 210.136, trehalose 6-tripalmitate m/z 603.335). Validate acquisition by confirming the run inventory matches the deposit's sample sheet before launching feature detection.

## Related tools

- **Optimus** (Target processing workflow into which the downloaded MassIVE runs are loaded for feature detection, alignment, and quantification) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the FeatureFinderMetabo-based feature detection algorithms that Optimus runs on the acquired mzML/mzXML data (20 ppm m/z tolerance for QTOF data)) — http://www.openms.de
- **KNIME** (Workflow management system that executes the Optimus pipeline on the downloaded dataset) — https://www.knime.org
- **pyopenms** (Python dependency installed alongside six, pandas, protobuf, pymspec to support the Optimus environment)
- **'ili** (Downstream app in which the acquired dataset's reprocessed features are visualized as spatial maps over the 96-well-plate image) — https://github.com/ili-toolbox/ili

## Examples

```
pip install six pandas protobuf pymspec pyopenms
```

## Evaluation signals

- The set of downloaded runs matches the deposit's file listing and sample sheet, with all 96-well positions (rows A–H, blanks included) represented.
- Files are valid mzML/mzXML that Optimus/KNIME can load without format-conversion errors.
- After processing, blank row A is free of the target features (minoxidil m/z 210.136, trehalose 6-tripalmitate m/z 603.335), confirming the blank controls were carried through acquisition.
- Reproduced 'ili maps place minoxidil intensity in rows C and G and trehalose 6-tripalmitate in row H, matching Figure 2c of the source article.
- Disk usage is planned for temporary files, which 'can occupy up to few times more space than initial dataset' per the Optimus README.

## Limitations

- Optimus requires 64-bit OS and needs more than the minimal 2 GB RAM for datasets of about a hundred or more LC–MS runs; disk for temporary files can be several times the dataset size and is not deleted automatically.
- Optimus annotations are mz-RT based (MSI level 2, putatively annotated compounds); MS/MS validation of putative annotations is needed and not provided by Optimus.
- Successful reproduction depends on the deposit retaining the plate layout (which wells are blanks, hand, phone-front, phone-back per volunteer); without this mapping the volunteer-row validation cannot be performed.

## Evidence

- [other] Download the LC-MS mzML/mzXML runs for MSV000079825 from MassIVE: [evidence span withheld — anchor status: not_found]
