---
name: technical-replicate-variability-assessment
description: Use when you have ≥2 technical replicates of a pooled QC sample across
  a targeted compound list (e.g. 8 replicates of a phospholipid QC with 163 retained
  phospholipids) and need to compare quantitation tools or deconvolution/correction
  settings by replicate-to-replicate abundance stability.
when_to_use_negative:
- Input samples are biological/batch QCs across batches rather than technical replicates
  of one QC sample — use batch-effect correction metrics (BQC regression slope/R2)
  instead of replicate STD.
- Replicates differ in sample preparation or injection, so variability reflects sample
  handling, not peak-picking — STD would confound the comparison.
- You need absolute concentrations (e.g. bile acids via standard curve at 0.025–500
  µmol/mL); replicate STD measures precision only and cannot validate calibration
  accuracy.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MRMQuant
  role: Runs the replicate QC quantitation in each deconvolution/correction mode and
    exports normalized abundances in an xlsx file for STD computation
  repo: https://github.com/kslynn128171/MRMQuant
- name: ProteoWizard msconvert
  role: Converts vendor raw replicate data into mzML, the input format MRMQuant loads
- name: Microsoft Excel
  role: Receives the MRMQuant quantitation results (xlsx) from which per-replicate
    normalized abundances are read to compute STD
- name: MRMkit
  role: Provides the example QC replicate data set and method file used as the benchmark
    inputs for the variability assessment
  repo: https://github.com/MRMkit/MRMkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:085bd87d8bb77333aee5697a99f9f13a77558c20c343f1e895475a25c389ad36
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MRMQuant
  - MRMkit
  - Microsoft Excel
  - ProteoWizard
  - ProteoWizard msconvert
  - msconvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# technical-replicate-variability-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-compound variability across technical replicates of a QC sample in targeted MRM metabolomics by computing the standard deviation of normalized abundances over replicates and averaging it across compounds, so peak-picking and correction modes can be ranked by reproducibility.

## When to use

Use when you have ≥2 technical replicates of a pooled QC sample across a targeted compound list (e.g. 8 replicates of a phospholipid QC with 163 retained phospholipids) and need to compare quantitation tools or deconvolution/correction settings by replicate-to-replicate abundance stability.

## When NOT to use

- Input samples are biological/batch QCs across batches rather than technical replicates of one QC sample — use batch-effect correction metrics (BQC regression slope/R2) instead of replicate STD.
- Replicates differ in sample preparation or injection, so variability reflects sample handling, not peak-picking — STD would confound the comparison.
- You need absolute concentrations (e.g. bile acids via standard curve at 0.025–500 µmol/mL); replicate STD measures precision only and cannot validate calibration accuracy.

## Inputs

- mzML files of technical-replicate MRM QC runs (converted from vendor raw data via msconvert in ProteoWizard)
- Method file listing targeted compounds and transitions (e.g. 184 phospholipids in 163 retained ion chromatograms)
- MRMQuant parameter settings per run mode (deconvolution off / automatic / reference-sample correction)

## Outputs

- Normalized abundance of each compound in each technical replicate
- Per-compound STD of normalized abundance across replicates
- Average STD across the compound set per run mode or tool (e.g. 0.0832 no deconvolution, 0.0828 automatic deconvolution, 0.0813 reference correction)
- Count of unquantifiable compounds per tool, as a covariate of the STD comparison

## How to apply

Convert vendor raw MRM data to mzML with msconvert in ProteoWizard, then run MRMQuant on the replicate QC sample in each mode under test (no deconvolution, automatic deconvolution, reference-sample correction) using the method file with the targeted compounds. Compute normalized abundance of each retained compound (163 phospholipids in the article's QC data set) in every replicate, take the STD of the normalized abundance per compound across the eight technical replicates, then average STD across compounds to yield one reproducibility number per run mode. When comparing tools, pair the average STD with the count of unquantifiable compounds, since a tool can attain a low average STD only by failing to quantify many compounds (as TargetLynx did). Also examine problematic peak classes explicitly: coeluted peaks (e.g. three isomers of PC 34:3 at RTs 4.72, 4.84, 4.92 min), noisy peaks, and peaks with unevenly sampled signals, since low STD alone does not prove correct peak attribution.

## Related tools

- **MRMQuant** (Runs the replicate QC quantitation in each deconvolution/correction mode and exports normalized abundances in an xlsx file for STD computation) — https://github.com/kslynn128171/MRMQuant
- **ProteoWizard msconvert** (Converts vendor raw replicate data into mzML, the input format MRMQuant loads)
- **Microsoft Excel** (Receives the MRMQuant quantitation results (xlsx) from which per-replicate normalized abundances are read to compute STD)
- **MRMkit** (Provides the example QC replicate data set and method file used as the benchmark inputs for the variability assessment) — https://github.com/MRMkit/MRMkit

## Examples

```
msconvert --mzML replicateQC.raw  # then run MRMQuant (v2.7) on the resulting mzML with the phospholipid method file; results land in an xlsx from which STD across the 8 replicates is computed
```

## Evaluation signals

- Each compound yields eight normalized abundance values across replicates before STD is computed; the retained compound count should match the data set (163 phospholipids).
- Average STD values should fall in the ~0.08 range for well-performing modes (article: 0.0813–0.0832 across the three MRMQuant modes).
- Run-mode ordering should be checked against the expected direction: reference-sample correction should give lower average STD than no deconvolution (0.0813 vs 0.0832).
- Cross-check any low-STD winner against its count of unquantifiable compounds — a low STD paired with many unquantifiable compounds is a red flag (TargetLynx pattern).
- Inspect coeluted cases such as PC 34:3 (three isomers at RTs 4.72, 4.84, 4.92 min) to confirm the tool did not overestimate abundance by attributing all coeluted peaks to one compound.

## Limitations

- Average STD alone is insufficient to rank tools: TargetLynx achieved the smallest average STD but had the highest number of unquantifiable compounds among the tested tools.
- Low STD does not guarantee correct peak attribution: MRMkit and Skyline reached low (<0.1) average STD yet mishandled coeluted, noisy, and unevenly sampled peaks.
- Results depend on MRMQuant version (README directs users to the latest version, v2.7) and on manual correction choices in the visual inspection step.

## Evidence

- [results] Average STD across replicates as benchmark: [evidence span withheld — anchor status: not_found]
