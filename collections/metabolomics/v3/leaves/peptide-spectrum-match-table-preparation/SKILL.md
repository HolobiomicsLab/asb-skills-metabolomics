---
name: peptide-spectrum-match-table-preparation
description: Use when building a ground-truth identification table (per-scan peptide
  assignments) to evaluate MS/MS clustering quality via DB-search-based purity, e.g.
  for PXD021518 or PXD023047 proteomics data.
when_to_use_negative:
- The input is metabolomics MS/MS data where FDR-controlled identifications are not
  currently practical — use the MS-RT retention-time-based evaluation instead
- Clustering tool outputs are already available and only need post-processing into
  consistent column formats (use the msCluster/Falcon/MaRaCluster post-processing
  scripts instead)
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS-GF+
  role: Performs the target-decoy database search of mzML spectra against the Arabidopsis
    thaliana reference proteome to produce candidate PSMs with q-values
- name: msconvert
  role: ProteoWizard tool that converts vendor-specific Thermo .raw files into mzML
    format prior to database searching
  repo: http://proteowizard.sourceforge.net/download.html
- name: ThermoRawFileParser
  role: Cross-platform alternative to msconvert for converting Thermo RAW files to
    mzML or MGF without vendor software
  repo: https://github.com/compomics/ThermoRawFileParser
- name: ProteoWizard
  role: Toolkit providing the msconvert converter used in the data preparation pipeline
  repo: http://proteowizard.sourceforge.net/download.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:b120ab14e7accfc02075da891425de70f938b13d92dc52e491e665f09ad43abe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-GF+
  - ProteoWizard
  - Python
  - ThermoRawFileParser
  - msconvert
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# peptide-spectrum-match-table-preparation

## Summary

Prepares the identified-PSM reference table used to evaluate MS/MS clustering on proteomics datasets: raw Thermo .raw files are converted to mzML, searched with MS-GF+ against a target-decoy database, and PSMs are filtered to 1% FDR, yielding per-spectrum peptide identifications that serve as the ground-truth purity labels.

## When to use

Use when building a ground-truth identification table (per-scan peptide assignments) to evaluate MS/MS clustering quality via DB-search-based purity, e.g. for PXD021518 or PXD023047 proteomics data.

## When NOT to use

- The input is metabolomics MS/MS data where FDR-controlled identifications are not currently practical — use the MS-RT retention-time-based evaluation instead
- Clustering tool outputs are already available and only need post-processing into consistent column formats (use the msCluster/Falcon/MaRaCluster post-processing scripts instead)

## Inputs

- Thermo .raw mass spectrometry files from ProteomeXchange datasets (e.g. PXD021518, PXD023047)
- mzML-converted spectra files
- Arabidopsis thaliana UniProt reference proteome (target-decoy)

## Outputs

- Filtered identified-PSM table (filename, scan, precursor mass, retention time, peptide/protein identification) written to data/DB-Search
- 1% FDR-passing PSM set used as the purity reference for clustering evaluation

## How to apply

Download the target proteomics raw datasets (e.g. PXD021518 and PXD023047) from ProteomeXchange and convert vendor-specific Thermo .raw files to mzML using msconvert (ProteoWizard) or ThermoRawFileParser. Run MS-GF+ (version 2023.01.12) against the Arabidopsis thaliana reference proteome in target-decoy mode with trypsin specificity, NumMods 2, and a 10 ppm precursor tolerance for PXD021518 (20 ppm for PXD023047). Filter the PSM results to 1% false discovery rate using the target-decoy q-value assignment from the search output. Write the filtered identified-PSM table (filename, scan, precursor mass, retention time, peptide/protein identification) into the project's data/DB-Search directory so the clustering_eval_compare_DB comparison step can consume it. The resulting table is the purity reference against which clustering tools (msCluster, Falcon, MaRaCluster) are scored, mirroring the article's validation of the MS-RT method against established proteomics clustering evaluation approaches.

## Related tools

- **MS-GF+** (Performs the target-decoy database search of mzML spectra against the Arabidopsis thaliana reference proteome to produce candidate PSMs with q-values)
- **msconvert** (ProteoWizard tool that converts vendor-specific Thermo .raw files into mzML format prior to database searching) — http://proteowizard.sourceforge.net/download.html
- **ThermoRawFileParser** (Cross-platform alternative to msconvert for converting Thermo RAW files to mzML or MGF without vendor software) — https://github.com/compomics/ThermoRawFileParser
- **ProteoWizard** (Toolkit providing the msconvert converter used in the data preparation pipeline) — http://proteowizard.sourceforge.net/download.html

## Examples

```
msconvert PXD021518.raw --mzML && java -Xmx8G -jar MSGFPlus.jar -s PXD021518.mzML -d arabidopsis_target_decoy.fasta -o PXD021518.mzid -t 10ppm -e 1 -m 0 && python filter_psms_1pct_FDR.py PXD021518.mzid > data/DB-Search/PXD021518_psms.tsv
```

## Evaluation signals

- The output table exists in data/DB-Search and contains the required per-spectrum identification fields (filename, scan, precursor mass, retention time, peptide/protein)
- All retained PSMs pass the 1% FDR threshold implied by the target-decoy q-value filter
- Precursor mass tolerance used matches the dataset-specific setting (10 ppm for PXD021518, 20 ppm for PXD023047)
- The table can be consumed directly by the clustering_eval_compare_DB step without schema rework

## Limitations

- Ground-truth labels only exist for proteomics datasets — metabolomics lacks FDR-controlled identifications, so DB-search-based purity evaluation is not applicable there
- Precursor mass tolerance must be adjusted per dataset (10 ppm vs 20 ppm); reusing one setting across datasets can degrade search sensitivity
- The search uses a single organism reference (Arabidopsis thaliana UniProt) — the workflow as documented is specific to that proteome

## Evidence

- [supplementary] The proteomics datasets (PXD023047, PXD021518) were searched against the Arabidopsis thaliana reference database in a target-decoy manner, and PSMs were filtered to 1% FDR at the PSM level: [evidence span withheld — anchor status: not_found]
- [supplementary] run MS-GF+ (version 2023.01.12) against the Arabidopsis thaliana UniProt reference proteome in target-decoy mode with trypsin specificity, NumMods 2: [evidence span withheld — anchor status: not_found]
