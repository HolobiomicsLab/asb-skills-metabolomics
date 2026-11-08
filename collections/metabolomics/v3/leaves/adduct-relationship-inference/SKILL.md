---
name: adduct-relationship-inference
description: Use when a processed, annotated LC-MS feature table (tidymass massdataset)
  contains suspected in-source fragmentation products misannotated as genuine metabolites
  and no in-pipeline deconvolution exists.
when_to_use_negative:
- The input is still raw instrument data — peak picking and data cleaning must first
  be done by MetMiner's tidymass-driven upstream processing before annotation filtering
  is meaningful
- The annotation table already provides MS/MS-level evidence distinguishing fragments
  from genuine metabolites, making retention-time/neutral-loss heuristics unnecessary
- Features lack retention times or elution profiles, so co-elution clustering — the
  core grouping step — cannot be computed
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0092
tools:
- name: MetMiner
  role: Supplies the processed, annotated feature table and the filter_annotations_massdataset
    annotation-filtering function used to remove flagged redundant annotations
  repo: github.com/ShawnWx2019/MetMiner
- name: MDAtoolkits
  role: Companion R package providing the statistical/analysis toolset invoked alongside
    MetMiner when manipulating the annotation table
  repo: github.com/ShawnWx2019/MDAtoolkits
- name: tidymass
  role: Provides the massdataset object structure in which the annotated feature table
    is loaded and filtered
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- name: tidyverse
  role: Used for the R data-manipulation steps (clustering by retention-time overlap,
    pairwise m/z difference computation, flagging rows)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:20d9070815be515460d891d9a6efdb370577a67f833e5f140167e93bed2a291d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MDAtoolkits
  - MetMiner
  - tidymass
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# adduct-relationship-inference

## Summary

A reconstructed procedure (declared as future work in the MetMiner paper) for detecting in-source fragmentation products that MetMiner misannotates as genuine metabolites, by grouping co-eluting features and matching pairwise m/z differences against common neutral-loss and adduct mass relationships, then removing flagged redundant annotations.

## When to use

Use when a processed, annotated LC-MS feature table (tidymass massdataset) contains suspected in-source fragmentation products misannotated as genuine metabolites and no in-pipeline deconvolution exists.

## When NOT to use

- The input is still raw instrument data — peak picking and data cleaning must first be done by MetMiner's tidymass-driven upstream processing before annotation filtering is meaningful
- The annotation table already provides MS/MS-level evidence distinguishing fragments from genuine metabolites, making retention-time/neutral-loss heuristics unnecessary
- Features lack retention times or elution profiles, so co-elution clustering — the core grouping step — cannot be computed

## Inputs

- Processed, annotated LC-MS feature table from the MetMiner pipeline (tidymass massdataset / annotation table)
- MetMiner demo files (Google Drive or Baidu Netdisk) as a test input
- Reference list of common neutral-loss masses (H2O, CO2, NH3) and isotopic/adduct mass offsets

## Outputs

- One row per flagged redundant annotation: child/parent feature IDs, m/z, retention time, and inferred parent/loss relationship
- Filtered annotation table with flagged in-source fragmentation entries removed via filter_annotations_massdataset

## How to apply

Load MetMiner's processed, annotated feature table (demo files from Google Drive or Baidu Netdisk) into R using tidyverse/tidymass data structures. Group features into co-elution clusters by retention-time overlap so features sharing an elution profile are compared within the same chromatographic peak. Within each cluster, compute pairwise m/z differences and match them against common neutral-loss and adduct mass relationships (e.g. losses of H2O, CO2, NH3 and isotopic/adduct mass offsets) to nominate fragment–parent candidate pairs. Flag child features whose m/z is lower than a candidate parent, that co-elute with it, and that satisfy a neutral-loss/adduct relationship, writing one row per flagged redundant annotation with feature IDs, m/z, retention time, and the inferred parent/loss relationship. Finally apply MetMiner's annotation-filtering logic — the filter_annotations_massdataset(object, annotate_tbl, method) function — to remove the flagged entries from the annotation table.

## Related tools

- **MetMiner** (Supplies the processed, annotated feature table and the filter_annotations_massdataset annotation-filtering function used to remove flagged redundant annotations) — github.com/ShawnWx2019/MetMiner
- **MDAtoolkits** (Companion R package providing the statistical/analysis toolset invoked alongside MetMiner when manipulating the annotation table) — github.com/ShawnWx2019/MDAtoolkits
- **tidymass** (Provides the massdataset object structure in which the annotated feature table is loaded and filtered) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- **tidyverse** (Used for the R data-manipulation steps (clustering by retention-time overlap, pairwise m/z difference computation, flagging rows))

## Examples

```
library(tidyverse); library(tidymass); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Every flagged pair must satisfy all three criteria: child m/z lower than the candidate parent, co-elution within the same chromatographic peak, and a matching neutral-loss/adduct mass relationship
- Flags must be restricted to co-eluting, mass-related feature pairs — inspect a sample of flagged pairs to confirm no unrelated annotations are flagged
- Each output row contains the required fields: feature IDs, m/z, retention time, and the inferred parent/loss relationship
- Applying filter_annotations_massdataset to the flagged rows demonstrably removes them from the annotation table (row count decreases by exactly the number of unique flagged child features)
- Delta m/z values of flagged pairs fall at known neutral-loss offsets (H2O, CO2, NH3) or recognized isotopic/adduct mass differences

## Limitations

- MetMiner itself offers no satisfactory solution for this task — the paper explicitly defers detection/removal of in-source fragmentation annotations to the next version, so this procedure is a reconstruction, not a supported pipeline feature
- Neutral-loss/adduct matching is heuristic: genuine metabolites sharing loss relationships with co-eluting parents risk being falsely flagged
- Reliability depends on retention-time overlap quality; noisy or drifting RT information will weaken co-elution clustering

## Evidence

- [discussion] a common issue during metabolite annotation is that some suspected in-source fragmentation products would be annotated as genuine metabolites: [evidence span withheld — anchor status: not_found]
- [discussion] Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version of MetMiner.: "Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version of MetMiner."
- [methods] filter_annotations_massdataset(object, annotate_tbl, method): "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [readme] start metMiner shinyapp run_metMiner(maxRequestSize = 300): "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
