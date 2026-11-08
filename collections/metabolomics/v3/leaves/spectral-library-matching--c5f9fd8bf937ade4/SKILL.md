---
name: spectral-library-matching
description: Use when a semi-targeted LC-MS analysis requires assigning compound identities
  to detected peaks, especially when curated authentic or high-quality MS/MS spectra
  are available for the targets of interest.
when_to_use_negative:
- The data are untargeted feature tables without MS/MS spectra — MetaPro targets semi-targeted
  (DDA) analysis anchored on mz/rt target libraries
- No authentic or high-quality reference spectra exist for the metabolites of interest
  — identification accuracy is highly dependent on spectral library quality
- Input is vendor raw data not yet converted by AirdPro to the .aird/. computational
  format
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetaPro
  role: Web application whose Method object exposes three peak identification strategies,
    including spectral comparison, applied in the identification step of the eight-step
    quantification pipeline
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: GUI client that converts vendor DDA files to the Aird computational format
    (.aird/.) consumed by the quantification pipeline before identification
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: HMDB
  role: Curated public spectral database used as reference for spectrum comparison
    during metabolite identification
- name: METLIN
  role: Curated public spectral database used as reference for spectrum comparison
    during metabolite identification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:b6e4ed2a065593f1d42a9594c38e9880d9f87d0f4ddf256a6c39414cb3bd8f12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - HMDB
  - METLIN
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-library-matching

## Summary

Identifies metabolite peaks by comparing experimental MS/MS spectra against curated spectral libraries (HMDB, METLIN, or self-made MetaPro libraries), which is one of the three configurable peak identification strategies in MetaPro's eight-step quantification pipeline.

## When to use

Use when a semi-targeted LC-MS analysis requires assigning compound identities to detected peaks, especially when curated authentic or high-quality MS/MS spectra are available for the targets of interest.

## When NOT to use

- The data are untargeted feature tables without MS/MS spectra — MetaPro targets semi-targeted (DDA) analysis anchored on mz/rt target libraries
- No authentic or high-quality reference spectra exist for the metabolites of interest — identification accuracy is highly dependent on spectral library quality
- Input is vendor raw data not yet converted by AirdPro to the .aird/. computational format

## Inputs

- Converted LC-MS/MS data in Aird format (.aird plus . index), produced by AirdPro from vendor DDA files (TripleTOF, QE HF)
- Target analyte and internal standard library tables (.txt/.tsv with at least mz and rt columns)
- MS/MS reference spectra from curated libraries (HMDB, METLIN) or self-made MetaPro spectral libraries

## Outputs

- Peak identification annotations (compound assignments) attached to screened peaks per batch
- Per-overview targeted detection results exported as a named .xlsx quantification/identification report

## How to apply

After the upstream steps of MetaPro's quantification pipeline (EIC extraction from Aird-format data, smoothing, noise estimation, peak selection, integration, and screening), apply identification as the final algorithmic step. Load an analyte/internal standard library table (.txt/.tsv containing at least mz and rt columns) so that target m/z and retention time anchor candidate matches. Select the identification method among the three exposed peak identification strategies via the MetaPro Method parameter object, choosing spectral comparison against curated databases (HMDB, METLIN) or self-made libraries when high-quality spectra exist, since identification accuracy is highly dependent on spectral library quality. Curate homemade MS/MS spectral libraries within MetaPro to improve match reliability where public or commercial libraries lack spectra. Iterate with the batch QC inspection interface: adjust identification behavior per compound using 'Reset integration params', 'Reset TargetRT', and two-point 'Manual Integration', then apply batch-wide. Export the resulting per-batch quantification/identification results into a named .xlsx report.

## Related tools

- **MetaPro** (Web application whose Method object exposes three peak identification strategies, including spectral comparison, applied in the identification step of the eight-step quantification pipeline) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (GUI client that converts vendor DDA files to the Aird computational format (.aird/.) consumed by the quantification pipeline before identification) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **HMDB** (Curated public spectral database used as reference for spectrum comparison during metabolite identification)
- **METLIN** (Curated public spectral database used as reference for spectrum comparison during metabolite identification)

## Examples

```
bash MetaproStart.sh  # then in browser at localhost:8080: Create Project (load .aird), Create Library (mz/rt .tsv), Set Method identification strategy, Target Search
```

## Evaluation signals

- The pipeline runs end-to-end on the TripleTOF and QE HF tutorial datasets and emits a non-empty, correctly formatted .xlsx report
- The Method configuration exposes exactly three peak identification methods, selectable without code changes
- Identified compounds resolve against library mz and rt anchors within the analyte/internal standard tables
- Batch-level re-identification after 'Reset integration params' / 'Reset TargetRT' propagates consistently across all samples in the batch
- Identification quality improves (or is at least unchanged) when swapping public libraries for curated self-made MS/MS spectra, consistent with the stated accuracy dependence on library quality

## Limitations

- Metabolite identification accuracy is highly dependent on the quality of the spectral library; public or commercial libraries lack spectra of many metabolites
- Manual inspection remains indispensable for verifying peak quantification; the workflow combines artificial judgment with algorithm inference (semi-targeted)
- MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method only

## Evidence

- [abstract] improve identification accuracy using different peak identification strategies: "With curated authentic or high-quality spectra, it can improve identification accuracy using different peak identification strategies."
- [intro] the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"
- [intro] comparing a given spectrum with curated spectral databases such as HMDB (Wishart et al., 2022) and METLIN (Smith et al., 2005): "comparing a given spectrum with curated spectral databases such as HMDB (Wishart et al., 2022) and METLIN (Smith et al., 2005)"
- [methods] MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method.: "MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
- [abstract] A spectral curation function is provided to promote the prosperity of homemade MS/MS spectral libraries in the metabolomics community.: "A spectral curation function is provided to promote the prosperity of homemade MS/MS spectral libraries in the metabolomics community."
