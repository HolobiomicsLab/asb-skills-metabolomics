---
name: massive-dataset-retrieval
description: Use when a study reports LC-MS molecular maps derived from a deposited
  MassIVE accession (e.g. MSV000078556) and the goal is to download those public runs
  and reproduce or re-analyze the published feature maps.
when_to_use_negative:
- The data of interest are already available locally as a processed feature quantification
  table (e.g. the '*.db' file produced by Optimus) and no reprocessing of raw runs
  is needed.
- The study data are direct-infusion or non-LC-MS mass spectrometry not deposited
  as MassIVE mzML runs, so the accession-based retrieval path does not apply.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: Optimus
  role: KNIME workflow that consumes the downloaded mzML runs via its 'Read LC-MS
    runs' node for reprocessing and spatial mapping
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the FeatureFinderMetabo feature detection algorithms used after dataset
    retrieval
  repo: http://www.openms.de
- name: '''ili'
  role: Web app used to visualize the retrieved-and-reprocessed molecular maps on
    the 3D human body model
  repo: https://github.com/ili-toolbox/ili
- name: KNIME
  role: Workflow management platform in which the Optimus pipeline processes the retrieved
    runs
  repo: https://www.knime.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:578ed547733232c0cd21815a9189c6a158f0987806999e0a1b94d477dac601ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - '''ili'
  - KNIME
  - OpenMS
  - Optimus
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# massive-dataset-retrieval

## Summary

Retrieve deposited mass spectrometry data from a MassIVE (MassIVE USI) accession such as MSV000078556 to obtain the raw LC-MS runs (mzML) needed for independent reprocessing of published molecular cartography results.

## When to use

Use when a study reports LC-MS molecular maps derived from a deposited MassIVE accession (e.g. MSV000078556) and the goal is to download those public runs and reproduce or re-analyze the published feature maps.

## When NOT to use

- The data of interest are already available locally as a processed feature quantification table (e.g. the '*.db' file produced by Optimus) and no reprocessing of raw runs is needed.
- The study data are direct-infusion or non-LC-MS mass spectrometry not deposited as MassIVE mzML runs, so the accession-based retrieval path does not apply.

## Inputs

- MassIVE accession ID (e.g. MSV000078556)
- Downloaded mzML LC-MS run files from the accession
- Sample coordinate/radius CSV and 3D model (STL) for downstream 'ili visualization

## Outputs

- Set of mzML LC-MS run files locally available for Optimus
- Reproduced spatial molecular maps (e.g. m/z 355.215 SLES, 225.075 sinapic acid, 300.289 C18-sphingosine) in ili_input.csv / mapping-table form for 'ili

## How to apply

Identify the MassIVE accession from the source card or publication (here MSV000078556, the human skin surface LC-MS dataset), then download the LC-MS runs in mzML format from the MassIVE repository. Load the downloaded mzML files into the Optimus KNIME workflow via the 'Read LC-MS runs' node, and generate the experimental design template with Optimus so that blank swab runs are marked as BLANK before feature detection. Configure feature detection (OpenMS FeatureFinderMetabo) with m/z tolerance 20 ppm and noise threshold 5,000 counts as recommended for QTOF data, then proceed with alignment, blank filtering (minimum intensity ratio 3.0 vs blanks), normalization, and 'ili visualization to check reproduction of the published maps.

## Related tools

- **Optimus** (KNIME workflow that consumes the downloaded mzML runs via its 'Read LC-MS runs' node for reprocessing and spatial mapping) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the FeatureFinderMetabo feature detection algorithms used after dataset retrieval) — http://www.openms.de
- **'ili** (Web app used to visualize the retrieved-and-reprocessed molecular maps on the 3D human body model) — https://github.com/ili-toolbox/ili
- **KNIME** (Workflow management platform in which the Optimus pipeline processes the retrieved runs) — https://www.knime.org

## Evaluation signals

- The downloaded mzML runs from MSV000078556 all load successfully into the Optimus 'Read LC-MS runs' node without missing-run errors.
- Blank swab runs are correctly labeled BLANK in the generated experimental design template, enabling blank filtering at intensity ratio >= 3.0.
- The reproduced maps show sodium lauryl sulfate (m/z 355.215) localized on scalp and hands, sinapic acid (m/z 225.075) on hands, and C18-sphingosine (m/z 300.289) on the sole of the foot, matching the paper's Fig. 2b distributions.
- For each target molecule, the highest-intensity spots in the 'ili rendering fall on the stated anatomical regions, and rendered snapshots are exported.

## Limitations

- Reproduction depends on the deposited raw data retaining its original quality; mass resolution, LC run-time, and instrument differences affect Optimus runtime and RAM needs (2 GB minimum is insufficient for ~100+ LC-MS runs).
- Optimus feature annotation is only putative (MSI level 2, mz-RT matching) and requires MS/MS validation, which Optimus itself does not provide; reported molecular identities should be treated accordingly.
- Temporary files from Optimus execution can occupy several times the initial dataset size and are not deleted automatically.

## Evidence

- [intro] task card: retrieval of MSV000078556: [evidence span withheld — anchor status: not_found]
- [intro] feature detection parameters for retrieved QTOF data: [evidence span withheld — anchor status: not_found]
- [intro] blank filtering threshold: [evidence span withheld — anchor status: not_found]
- [intro] expected reproduction pattern: [evidence span withheld — anchor status: not_found]
