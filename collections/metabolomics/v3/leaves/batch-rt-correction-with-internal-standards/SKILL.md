---
name: batch-rt-correction-with-internal-standards
description: Use when multi-batch DDA LC-MS data (e.g. converted TripleTOF or QE HF
  vendor files) show retention-time shifts for targeted compounds between batches
  and quantification peaks must be re-integrated at corrected TargetRT values batch-wide.
when_to_use_negative:
- Input is already a processed feature table or peak list with final RTs — no EIC-level
  re-integration is possible in MetaPro
- Data were not acquired in DDA mode and not converted to the Aird format, since MetaPro
  supports MS data acquired with the DDA method
- Only a single sample needs RT adjustment with no batch context — per-compound manual
  inspection without batch re-application is more appropriate
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaPro
  role: Web application hosting the batch QC inspection interface where 'Reset TargetRT'
    and batch-wide re-integration are performed; v1.1.8 is the reference release
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts vendor files to the Aird computational format that MetaPro's RT correction
    and re-integration operate on; v1.1.0.0 is the reference release
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Started by the MetaPro launcher as part of the runtime backing the web application
    during batch analysis
- name: MongoDB
  role: Started by the MetaPro launcher to persist project/library/method state used
    across batch QC sessions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:4feb07d9ecced6782518cda3eac8738ced2bf1be0162de40573032a72c57d388
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Batch RT correction with internal standards

## Summary

Corrects per-batch retention-time drift in semi-targeted LC-MS quantification by resetting the TargetRT of each compound against analyte and internal-standard libraries and re-applying corrected integration parameters across all samples in a batch, as implemented in MetaPro's batch QC inspection workflow.

## When to use

Use when multi-batch DDA LC-MS data (e.g. converted TripleTOF or QE HF vendor files) show retention-time shifts for targeted compounds between batches and quantification peaks must be re-integrated at corrected TargetRT values batch-wide.

## When NOT to use

- Input is already a processed feature table or peak list with final RTs — no EIC-level re-integration is possible in MetaPro
- Data were not acquired in DDA mode and not converted to the Aird format, since MetaPro supports MS data acquired with the DDA method
- Only a single sample needs RT adjustment with no batch context — per-compound manual inspection without batch re-application is more appropriate

## Inputs

- Converted LC-MS data (.aird files plus . index files, e.g. TripleTOF dataset with 8 batches/16 files or QE HF dataset with 10 batches/20 files)
- Analyte library table (.txt/.tsv with at least mz and rt columns)
- Internal standard library table (.txt/.tsv with at least mz and rt columns)
- MetaPro Method parameter set created on the Method page

## Outputs

- Batch-wide corrected TargetRT values and re-integrated compound peaks per overview
- Per-overview targeted detection results across all batches
- Quantification results exported as a named .xlsx report

## How to apply

Import converted .aird/.aird index files (produced by AirdPro from DDA vendor data) into a MetaPro batch, and load analyte and internal-standard libraries as .txt/.tsv tables containing at least mz and rt columns. Create a Method with parameterized settings (EIC smoothing, noise estimation, peak selection, integration, screening, identification options) and run Target Search on the selected batches. In the batch QC inspection interface, inspect each compound's EIC peak; when the actual peak deviates from the library RT, use 'Reset integration params' and 'Reset TargetRT' so the integration window follows the observed peak. Rationale: MetaPro's semi-targeted design combines artificial judgment with algorithm inference, so a manual RT correction made on one representative compound can be applied batch-wide, and manual two-point re-integration ('Manual Integration', choosing two data points) handles edge cases where automatic peak selection fails. Corrected peaks then feed the eight-step quantification pipeline (EIC extraction through identification) and export to an .xlsx report.

## Related tools

- **MetaPro** (Web application hosting the batch QC inspection interface where 'Reset TargetRT' and batch-wide re-integration are performed; v1.1.8 is the reference release) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts vendor files to the Aird computational format that MetaPro's RT correction and re-integration operate on; v1.1.0.0 is the reference release) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Started by the MetaPro launcher as part of the runtime backing the web application during batch analysis)
- **MongoDB** (Started by the MetaPro launcher to persist project/library/method state used across batch QC sessions)

## Examples

```
cd INSTALLATION_PATH/MetaproForUbuntu && bash MetaproStart.sh  # then open http://localhost:8080, load .aird batch + mz/rt libraries, run Target Search, Reset TargetRT per compound
```

## Evaluation signals

- Corrected TargetRT places the integration window on the observed EIC apex in both positive and negative inspection views
- The RT correction applied to one compound propagates to all samples in the selected batch (batch-wide application succeeds without per-sample edits)
- Re-running Target Search after correction produces non-empty per-overview targeted detection results for all batches of both the TripleTOF and QE HF datasets
- The final export produces a named, non-empty .xlsx quantification report
- Manual Integration with two selected data points yields start/end boundaries bracketing the peak consistently across replicates

## Limitations

- RT correction is anchored to library RT columns of homemade/curated tables; accuracy depends on spectral/RT library quality
- Automatic peak selection may still fail after RT reset, requiring two-point Manual Integration per compound
- MetaPro v1.1.8 is for academic use only; commercial use requires contacting the authors

## Evidence

- [methods] Reset integration params / Reset TargetRT: "users can `Reset integration params` and `Reset TargetRT` of a compound."
- [methods] Batch-wide Target Search: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
- [abstract] Semi-targeted workflow rationale: "A semi-targeted analysis workflow is designed based on the concept of combining artificial judgment and algorithm inference."
- [methods] Manual two-point integration: "Click on `Manual Integration`, you can choose two data points to conduct manual integration functions."
- [methods] DDA support constraint: "MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
