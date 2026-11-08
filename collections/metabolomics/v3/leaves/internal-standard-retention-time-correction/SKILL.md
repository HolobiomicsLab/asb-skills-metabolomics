---
name: internal-standard-retention-time-correction
description: Use when running MetaPro Target Search on Aird-converted DDA LC-MS batches
  (TripleTOF 6600 .wiff or Q Exactive HF .
when_to_use_negative:
- Input files are still vendor format (.raw/.wiff) and have not been converted to
  Aird format with AirdPro — MetaPro cannot scan or analyze them directly
- The compound library contains no usable 'rt' column values, since the RT-distribution
  pattern recognition relies on RTs of adjacent library compounds
- The goal is untargeted feature detection or de novo metabolite identification rather
  than semi-targeted quantification anchored to a library
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaPro
  role: Web application whose Target Search result-optimization module performs multi-file
    EIC alignment and RT-distribution pattern recognition to refine peak selection;
    provides Reset TargetRT, Reset integration params, and Manual Integration for
    correction of drifted compounds
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: GUI client that converts vendor .raw/.wiff DDA files into Aird format (.aird
    + .) so MetaPro can load them for RT-corrected semi-targeted analysis
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Backing service started by MetaPro's launcher, required for in-memory caching
    and analysis of MS data during Target Search
- name: MongoDB
  role: Backing service started by MetaPro's launcher, required alongside Redis for
    the analysis data directory and task tracking
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:dd4a82d63881fce0790967879b41e35c4fa3f07e2eb0ee4a8bbb5e7c38c9b55f
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Internal-standard retention time correction

## Summary

A MetaPro semi-targeted analysis technique in which internal standard library entries (.txt/.tsv tables with 'mz' and 'rt' columns) and the RT distributions of adjacent library compounds are exploited by MetaPro's RT-based pattern recognition algorithm to refine peak selection and retention-time anchoring before manual batch QC inspection.

## When to use

Use when running MetaPro Target Search on Aird-converted DDA LC-MS batches (TripleTOF 6600 .wiff or Q Exactive HF .raw) and the analyte library's RT values need reinforcement or correction using internal standards and neighboring compound RT distributions to keep auto-selected peaks consistent across batch files.

## When NOT to use

- Input files are still vendor format (.raw/.wiff) and have not been converted to Aird format with AirdPro — MetaPro cannot scan or analyze them directly
- The compound library contains no usable 'rt' column values, since the RT-distribution pattern recognition relies on RTs of adjacent library compounds
- The goal is untargeted feature detection or de novo metabolite identification rather than semi-targeted quantification anchored to a library

## Inputs

- Aird-converted LC-MS data (.aird data file + . index file) from DDA acquisition, organized under project/batch folders
- Analyte library as .txt/.tsv table with at least 'mz' and 'rt' columns (Library Type = ANALYTES)
- Internal standard library as .txt/.tsv table with at least 'mz' and 'rt' columns (Library Type = IS)
- MetaPro Method with preset analysis parameters per instrument platform (TripleTOF 6600 or Q Exactive HF)

## Outputs

- Per-compound auto-selected peaks with consistent RT regions and integration bounds across all files in each batch
- Batch QC overview page showing per-compound Success/Failed/Unknow peak-selection consistency status
- Exported quantification overview (.xlsx via Export Report) whose peak areas can be used for blank/analyte (SB/SA) fold-change computation

## How to apply

Convert vendor files to Aird format (.aird + . index) with AirdPro v1.1 in DDA mode and load them into MetaPro v1.1.8 via Scan Files (Redis and MongoDB running as backing services). Upload two libraries on the Library page — Library Type ANALYTES and Library Type IS — as .txt/.tsv tables whose first line contains at least 'mz' and 'rt'; the IS entries supply RT anchors while adjacent library compounds supply RT distributions. Create a Method per instrument platform (AB SCIEX TripleTOF 6600, Thermo Q Exactive HF) with the preset recommended parameters, assign libraries and method to each batch, then click `Target Search`. The result-optimization module performs compound-by-compound multi-file EIC alignment for peak-selection consistency across all experiments in a batch, and RT-distribution pattern recognition over adjacent library compounds to improve auto peak selection; if a compound's RT drifts, use `Reset integration params` and `Reset TargetRT` or `Manual Integration` (two data points) on the inspection page.

## Related tools

- **MetaPro** (Web application whose Target Search result-optimization module performs multi-file EIC alignment and RT-distribution pattern recognition to refine peak selection; provides Reset TargetRT, Reset integration params, and Manual Integration for correction of drifted compounds) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (GUI client that converts vendor .raw/.wiff DDA files into Aird format (.aird + .) so MetaPro can load them for RT-corrected semi-targeted analysis) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Backing service started by MetaPro's launcher, required for in-memory caching and analysis of MS data during Target Search)
- **MongoDB** (Backing service started by MetaPro's launcher, required alongside Redis for the analysis data directory and task tracking)

## Evaluation signals

- On the batch QC page, every compound whose auto-selected peaks share the same RT region and integration bounds across all batch files must be recorded with a Success check status
- Any compound flagged Failed or Unknow must be traceable to its specific inconsistent files, matching the Success/Failed/Unknow status scheme of the MetaPro batch QC interface
- IS and ANALYTES libraries were both uploaded with Library Type set correctly and contain 'mz' and 'rt' in the first line of the .txt/.tsv file
- Log-transformed SB/SA fold changes computed from exported peak areas remain stable when comparing results before and after manual inspection (the paper's run-1 accuracy/stability criterion)

## Limitations

- Peak identification accuracy is highly dependent on spectral library quality; public or commercial libraries lack spectra of many metabolites, so RT correction is bounded by library completeness
- RT-based pattern recognition improves but does not guarantee peak selection: compounds flagged Failed/Unknow still require manual inspection (Reset TargetRT, Manual Integration with two data points)
- MetaPro supports only DDA-acquired MS data; DIA/SWATH or MRM acquisitions fall outside this workflow's assumptions

## Evidence

- [abstract] Semi-targeted workflow combining algorithms with manual correction: "A semi-targeted analysis workflow is designed based on the concept of combining artificial judgment and algorithm inference."
- [intro] Library quality governs identification accuracy: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"
- [methods] Library upload procedure in MetaPro: "#### 4.2.2 Load library Enter Library page, click `Create Library` at the top."
- [methods] Manual RT correction controls on the inspection page: "users can `Reset integration params` and `Reset TargetRT` of a compound."
- [methods] Target Search launches the semi-targeted analysis on selected batches: "Click `Target Search` to start a targeted analysis. You can select multiple batches and click `Target Search`"
- [readme] MetaPro installation requires Redis and MongoDB backing services: "********* Start Redis *********"
