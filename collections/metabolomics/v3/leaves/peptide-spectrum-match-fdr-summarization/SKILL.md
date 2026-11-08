---
name: peptide-spectrum-match-fdr-summarization
description: Use when raw LC-MS files have been searched by a peptide/protein identification
  engine (e.g., FragPipe/MSFragger or DIA-NN) and you need per-file identification
  counts at a chosen FDR threshold for QC, throughput reporting, or result aggregation.
when_to_use_negative:
- MSConnect is not a standalone analysis tool — if the search engine has not yet produced
  identification results (no FragPipe/DIA-NN output), this skill cannot create them
  and only provides the wrapper/interface
- Input is already a quantified feature or protein table — this skill tabulates identifications
  passing an FDR filter, not downstream steps such as imputing missing values, normalization
  and batch correction, or differential expression
- Analyzing a single small dataset interactively where manual inspection of search-engine
  output is feasible and no automated, traceable per-file reporting is needed
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MSConnect
  role: Web-based platform whose Omics Data Processor workers queue, execute, and
    collect per-file identification results; server-side aggregation produces the
    summary table
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Proteomics_Data_Processor
  role: Processing component that downloads tasks from the MSConnect queue, invokes
    search-engine CLIs, and uploads results back to the server
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- name: FragPipe
  role: Search engine invoked per-file for DDA and wide-window DDA (WWA) acquisitions,
    producing PSMs, peptides, and protein groups for FDR-filtered tabulation
- name: DIA-NN
  role: Search engine invoked per-file for DIA acquisitions, producing identification
    results for FDR-filtered tabulation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:6474998b3c65f467a3010ef194bf16dacde6517a45ea6be900cc0b290482255f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - DIA-NN
  - FragPipe
  - MSConnect
  - PRIDE
  - Proteomics_Data_Processor
  - msconvert
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# peptide-spectrum-match-fdr-summarization

## Summary

Tabulates per-file spectrum identification statistics — PSMs, peptides, and protein groups passing a given FDR filter — produced by search engines such as FragPipe (DDA/WWA) and DIA-NN (DIA), and aggregates them into a single summary table with one row per raw LC-MS file.

## When to use

Use when raw LC-MS files have been searched by a peptide/protein identification engine (e.g., FragPipe/MSFragger or DIA-NN) and you need per-file identification counts at a chosen FDR threshold for QC, throughput reporting, or result aggregation.

## When NOT to use

- MSConnect is not a standalone analysis tool — if the search engine has not yet produced identification results (no FragPipe/DIA-NN output), this skill cannot create them and only provides the wrapper/interface
- Input is already a quantified feature or protein table — this skill tabulates identifications passing an FDR filter, not downstream steps such as imputing missing values, normalization and batch correction, or differential expression
- Analyzing a single small dataset interactively where manual inspection of search-engine output is feasible and no automated, traceable per-file reporting is needed

## Inputs

- Raw LC-MS data files (e.g., Thermo .raw / Bruker formats) from Orbitrap Astral, timsTOF Ultra, or Orbitrap Exploris 480 instruments
- Search-engine identification outputs from FragPipe (DDA/WWA) or DIA-NN (DIA), including PSM, peptide, and protein group results
- Metadata table assigning each file an acquisition mode (DDA, WWA, or DIA) and instrument origin
- FDR threshold setting applied to filter identifications before counting

## Outputs

- Per-file identification statistics table (PSM counts, peptide counts, protein group counts passing the FDR filter)
- Aggregated summary table with columns for file name, acquisition mode, search engine used, and identification statistics
- Results uploaded back to the MSConnect server by the Omics Data Processor for web-based viewing

## How to apply

Run the acquisition-mode-appropriate search engine first — FragPipe/MSFragger for DDA and wide-window DDA (WWA) data, DIA-NN for DIA data — so each raw file yields identification results. Then, in MSConnect, configure one Omics Data Processor (ODP) worker per search engine as a processing app: the ODP downloads each raw file plus its configuration file from the task queue, invokes the engine's command line interface, and uploads the results to the MSConnect server. Apply the target false discovery rate (FDR) filter to the engine output and tabulate basic spectrum identification information: PSMs, peptides, and protein groups passing that filter. Aggregate the per-file results into one table with columns for file name, acquisition mode (DDA, WWA, or DIA), engine used, and the identification statistics. Rationale: in the paper's 26-day validation on dataset PXD037527, this tabulation confirmed that all 6410 files (6048 Orbitrap Astral plus 362 from timsTOF Ultra and Orbitrap Exploris 480) were automatically analyzed without incident.

## Related tools

- **MSConnect** (Web-based platform whose Omics Data Processor workers queue, execute, and collect per-file identification results; server-side aggregation produces the summary table) — https://github.com/RTKlab-BYU/MSConnect
- **Proteomics_Data_Processor** (Processing component that downloads tasks from the MSConnect queue, invokes search-engine CLIs, and uploads results back to the server) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor
- **FragPipe** (Search engine invoked per-file for DDA and wide-window DDA (WWA) acquisitions, producing PSMs, peptides, and protein groups for FDR-filtered tabulation)
- **DIA-NN** (Search engine invoked per-file for DIA acquisitions, producing identification results for FDR-filtered tabulation)

## Evaluation signals

- Aggregated table contains exactly one row per deposited raw file (6410 rows for PXD037527: 6048 Orbitrap Astral plus 362 timsTOF Ultra/Orbitrap Exploris 480)
- Each row carries a non-null acquisition mode (DDA, WWA, or DIA) and the engine actually used, matching the metadata assignment
- Reported PSM, peptide, and protein group counts correspond to identifications passing the specified FDR filter, not unfiltered engine output
- No files in the queue were left unprocessed — every submitted processing task returned identification results and uploaded them to the MSConnect server
- Errors during processing are visible in the Django Log Viewer through the web interface, confirming no silent failures

## Limitations

- MSConnect is not a standalone data analysis tool and does not ship any data processor or analysis software/license — it provides a wrapper/interface, and third-party applications must be installed and licensed separately
- The platform only tabulates basic identification information; the reported results table does not by itself include quantification, imputation, normalization, batch correction, or differential expression
- Experimental design tracking is not yet released — a module to formally track experimental design and generate data packages is still under development
- Reproducibility depends on the configuration files supplied per task; the article notes method descriptions alone are often insufficient to ensure reproducibility of complex data analysis strategies

## Evidence
