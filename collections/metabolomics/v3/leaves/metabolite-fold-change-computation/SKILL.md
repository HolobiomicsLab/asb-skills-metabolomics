---
name: metabolite-fold-change-computation
description: Use when MetaPro Target Search quantification results (.xlsx report from
  Export Report) are available and you need SB-vs-SA log fold-change distributions
  to benchmark automatic semi-targeted quantification accuracy.
when_to_use_negative:
- Input is still raw vendor data (Thermo .raw, SCIEX .wiff) or .aird files rather
  than an exported MetaPro quantification report — Target Search and Export Report
  must run first
- The goal is manual QC inspection or library curation (Reset integration params,
  Reset TargetRT, Manual Integration) rather than quantifying automatic results
- The report contains manually reintegrated peaks, which would break comparison with
  the no-manual-inspection first-run benchmark
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaPro
  role: Runs the semi-targeted Target Search on Aird-converted batches and exports
    the quantification overview (.xlsx) whose peak areas feed the SB/SA fold-change
    computation
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts Thermo .raw and SCIEX .wiff benchmark files to Aird format (.aird
    and . index) in DDA mode prior to MetaPro import
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Started as part of the MetaPro server stack (in-memory caching/backend) supporting
    batch Target Search execution
- name: MongoDB
  role: Started as part of the MetaPro server stack persisting project/library/method
    data used in the quantification run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:c18f2fb4da69b6a55d5be951d53764277eb113fb5fd4c83389e9dd3fbd377946
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metabolite-fold-change-computation

## Summary

Computes per-compound log-transformed fold changes of signal in blank (SB) relative to analyte sample (SA) from MetaPro-exported quantification peak areas to assess quantification accuracy and stability, mirroring the Fig. 4B/4F benchmark comparison in the MetaPro paper.

## When to use

Use when MetaPro Target Search quantification results (.xlsx report from Export Report) are available and you need SB-vs-SA log fold-change distributions to benchmark automatic semi-targeted quantification accuracy.

## When NOT to use

- Input is still raw vendor data (Thermo .raw, SCIEX .wiff) or .aird files rather than an exported MetaPro quantification report — Target Search and Export Report must run first
- The goal is manual QC inspection or library curation (Reset integration params, Reset TargetRT, Manual Integration) rather than quantifying automatic results
- The report contains manually reintegrated peaks, which would break comparison with the no-manual-inspection first-run benchmark

## Inputs

- MetaPro Export Report .xlsx overview containing per-compound peak areas for SB (blank) and SA (analyte) conditions
- Analyte and internal standard libraries (.txt/.tsv, first line containing at least mz and rt items) used for the Target Search
- Context of which batches (AB SCIEX TripleTOF 6600, Thermo Q Exactive HF) each library and method was assigned to

## Outputs

- Per-compound log-transformed SB/SA fold-change values
- Fold-change distribution summaries (e.g., scatter plots/statistics) comparable to Fig. 4B/4F of the MetaPro paper

## How to apply

After running Target Search in MetaPro v1.1.8 on batches of Aird-converted files (TripleTOF 6600 and Q Exactive HF benchmark data), export the overview via Export Report to obtain peak areas as an .xlsx file. For each compound in the report, take the exported peak areas of the blank condition (SB) and the analyte condition (SA), compute the log-transformed fold change of SB relative to SA, keeping compounds grouped by their library entries (analytes and internal standards loaded from .txt/.tsv tables whose first line contains mz and rt items). Because this skill reproduces the paper's first run, apply it only to the automatic results, skipping any Manual Integration or Reset TargetRT re-curation so the distribution reflects fully automatic quantification. Judge correctness by comparing the resulting fold-change distribution before and after manual inspection: small shifts indicate that the automatic quantification was accurate and stable, as in Fig. 4B/4F.

## Related tools

- **MetaPro** (Runs the semi-targeted Target Search on Aird-converted batches and exports the quantification overview (.xlsx) whose peak areas feed the SB/SA fold-change computation) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts Thermo .raw and SCIEX .wiff benchmark files to Aird format (.aird and . index) in DDA mode prior to MetaPro import) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Started as part of the MetaPro server stack (in-memory caching/backend) supporting batch Target Search execution)
- **MongoDB** (Started as part of the MetaPro server stack persisting project/library/method data used in the quantification run)

## Examples

```
python -c "import pandas as pd,numpy as np; d=pd.read_excel('MetaPro_report.xlsx'); d['log2FC']=np.log2(d['SB']/d['SA'])"
```

## Evaluation signals

- Every compound in the exported .xlsx receives a computed log-transformed SB/SA value (or a defined exclusion when SB area is zero)
- Fold-change distributions computed from automatic results closely match those computed after manual inspection (accuracy and stability criterion of the paper)
- Values are consistent with the Fig. 4B/4F comparison conditions (SB relative to SA, same benchmark datasets: TripleTOF 6600 and Q Exactive HF)
- No re-curation steps (Reset integration params, Reset TargetRT, Manual Integration) were applied to the peaks feeding the computation
- Per-compound grouping of areas matches the analyte/internal-standard library assignments made per batch in MetaPro

## Limitations

- Reproduces only the first automatic run (no spectral library, no manual inspection); adding manual reintegration changes the result distribution
- The benchmark applies to DDA-acquired LC-MS data converted via AirdPro; other acquisition modes (DIA_SWATH, PRM, MRM) are outside this workflow
- Metabolite identification accuracy is highly dependent on spectral library quality, which can affect downstream quantification of compounds lacking curated spectra

## Evidence

- [other] Compute per-compound log-transformed fold changes of signal in blank (SB) relative to analyte sample (SA) conditions from the exported peak areas, matching the Fig. 4B/4F comparison conditions.: [evidence span withheld — anchor status: not_found]
- [other] Export the resulting overview via Export Report to obtain the quantification results as an .xlsx file.: [evidence span withheld — anchor status: not_found]
- [methods] Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
- [methods] Installation packages for all the supported systems can be downloaded at [GitHub](https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8): "Installation packages for all the supported systems can be downloaded at [GitHub](https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8)"
- [readme] This version is for academic use only. For commercial use, please contact us.: "This version is for academic use only. For commercial use, please contact us."
