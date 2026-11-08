---
name: metabolomics-workbench-and-metabolights-data-retrieval
description: Use when a reproducibility, benchmarking, or reanalysis task cites public
  study accessions (ST* from Metabolomics Workbench, MTBLS* from MetaboLights) or
  Zenodo deposits and requires the raw MS1 LC/HRMS files, parameter spreadsheets,
  IPDBs, or MSI 1-3 annotation lists behind them.
when_to_use_negative:
- The dataset is proprietary, under embargo, or lacks a resolvable public accession
  — nothing can be retrieved from the repositories
- Raw files and IPDBs are already staged locally and the task is annotation computation
  (IDSL.UFA/IPA), not data acquisition
- The study accession format does not match repository conventions (ST* = Metabolomics
  Workbench, MTBLS* = MetaboLights) and no deposit URL is given
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Consumes the retrieved raw mzXML/mzML/netCDF MS1 files downstream for peak
    extraction; retrieval must supply files in these formats
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: Consumes retrieved raw MS1 data plus downloaded IPDB and UFA parameter spreadsheet
    to annotate formulas on the retrieved studies
  repo: https://cran.r-project.org/package=IDSL.UFA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:db6770cf447d6529307fc891176151e417a01b6f7990646946314cee7a9746ce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-workbench-and-metabolights-data-retrieval

## Summary

Locates and downloads public untargeted LC/HRMS study datasets and their companion parameter/IPDB deposits from Metabolomics Workbench, MetaboLights, and Zenodo so that formula-annotation workflows (e.g., IDSL.UFA benchmarking) can be reproduced on identical inputs.

## When to use

Use when a reproducibility, benchmarking, or reanalysis task cites public study accessions (ST* from Metabolomics Workbench, MTBLS* from MetaboLights) or Zenodo deposits and requires the raw MS1 LC/HRMS files, parameter spreadsheets, IPDBs, or MSI 1-3 annotation lists behind them.

## When NOT to use

- The dataset is proprietary, under embargo, or lacks a resolvable public accession — nothing can be retrieved from the repositories
- Raw files and IPDBs are already staged locally and the task is annotation computation (IDSL.UFA/IPA), not data acquisition
- The study accession format does not match repository conventions (ST* = Metabolomics Workbench, MTBLS* = MetaboLights) and no deposit URL is given

## Inputs

- Public study accession IDs (e.g., ST001154, ST001683, ST002263; MTBLS1684, MTBLS2542)
- Zenodo deposit URLs for parameter spreadsheets, IPDBs, and annotation tables (e.g., zenodo.org/record/5838709)
- Repository API/page endpoints (Metabolomics Workbench DRCCMetadata study pages, MetaboLights study pages)

## Outputs

- Raw MS1 LC/HRMS data files (mzXML, mzML, netCDF) organized per study
- Deposited IDSL.UFA/IPA parameter spreadsheets and IPDB .Rdata files (e.g., IDSL.ExposomeDB IPDB)
- Reference MSI 1-3 annotation tables used as ground truth for validation

## How to apply

Resolve each accession to its repository: ST-prefixed IDs (e.g., ST001154, ST001683, ST002263) live at metabolomicsworkbench.org and MTBLS-prefixed IDs (e.g., MTBLS1684, MTBLS2542) at MetaboLights; fetch the raw MS1 files (mzXML, mzML, netCDF are the formats IDSL.IPA consumes). In parallel, retrieve the companion Zenodo deposits that hold the deposited IDSL.UFA/IPA parameter spreadsheets, the IPDB (the IDSL.ExposomeDB IPDB contains 209,592 positive-mode and 129,122 negative-mode ion formulas), and the MSI 1-3 confidence-level annotations (Zenodo record 5838709 for MTBLS2542 annotations). Confirm each accession resolves and the deposit contains the claimed file types before running downstream annotation, so the reproduction matches the published inputs exactly.

## Related tools

- **IDSL.IPA** (Consumes the retrieved raw mzXML/mzML/netCDF MS1 files downstream for peak extraction; retrieval must supply files in these formats) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFA** (Consumes retrieved raw MS1 data plus downloaded IPDB and UFA parameter spreadsheet to annotate formulas on the retrieved studies) — https://cran.r-project.org/package=IDSL.UFA

## Examples

```
curl -L -o ST001154.html "https://www.metabolomicsworkbench.org/data/DRCCMetadata.php?Mode=Study&StudyID=ST001154&DataMode=AllData&ResultType=1"; curl -L -O "https://zenodo.org/record/5838709"
```

## Evaluation signals

- Each accession URL resolves and returns study metadata (e.g., Metabolomics Workbench DRCCMetadata.php?StudyID=ST001154 loads the correct study)
- Retrieved raw files are MS1-level LC/HRMS in mzXML/mzML/netCDF format compatible with the IDSL.IPA workflow
- Zenodo deposits contain the documented content, e.g., the IDSL.ExposomeDB IPDB with 209,592 and 129,122 ion formulas in positive and negative modes
- Ground-truth annotation table corresponds to MSI 1-3 confidence levels (Zenodo record 5838709) for MTBLS2542 validation
- Downstream reproduction lands in the published windows: top-hit 54.31-85.51% and top-5-hit 90.58-100% across the four studies

## Limitations

- Retrieval success depends on third-party repository availability and stable Zenodo record identifiers; deposits are the reproducibility anchor for parameter spreadsheets and IPDBs
- Accession naming is inconsistent across repositories (ST* vs MTBLS*) and article text may merge accession numbers with citation numerals (e.g., 'ST00115427'), requiring careful ID parsing
- The article does not state IDSL.UFA or IDSL.IPA software versions, so retrieved inputs alone cannot guarantee bit-identical reprocessing

## Evidence

- [readme] README study accession URL pattern and file formats: [evidence span withheld — anchor status: not_found]
