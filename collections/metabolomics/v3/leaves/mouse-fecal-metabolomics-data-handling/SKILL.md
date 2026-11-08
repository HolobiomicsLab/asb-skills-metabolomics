---
name: mouse-fecal-metabolomics-data-handling
description: Use when a metabolomics analysis needs a ready-to-use mouse fecal sample
  count dataset with treatment-factor metadata for testing omu workflows or reproducing
  the paper's figures.
when_to_use_negative:
- When the user already possesses their own metabolomics count table and metadata
  — this skill is only for the packaged example dataset.
- When the analysis targets non-fecal or non-mouse samples with different metadata
  structure, since the example files are specifically fecal samples from a two-factor
  mouse experiment.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: omu
  role: R package that ships the example mouse fecal metabolomics dataset and provides
    read.metabo, assign_hierarchy, and omu_summary for handling it
  repo: github.com/connor-reid-tiffany/Omu
- name: R
  role: runtime environment in which omu's data-loading and metadata-assignment functions
    are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:1e225ff9c4f0dd96e566b36e4dab2e671a08d0c437c11359844f0197be4de6f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ggplot2
  - omu
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mouse-fecal-metabolomics-data-handling

## Summary

Loads the omu package's example mouse fecal metabolomics count table and metadata (c57_nos2KO_mouse_countDF / c57_nos2KO_mouse_metadata), assigns KEGG hierarchy metadata, and prepares the data frame for statistical testing and plotting with omu's functions.

## When to use

Use when a metabolomics analysis needs a ready-to-use mouse fecal sample count dataset with treatment-factor metadata for testing omu workflows or reproducing the paper's figures.

## When NOT to use

- When the user already possesses their own metabolomics count table and metadata — this skill is only for the packaged example dataset.
- When the analysis targets non-fecal or non-mouse samples with different metadata structure, since the example files are specifically fecal samples from a two-factor mouse experiment.

## Inputs

- omu example metabolomics count data frame c57_nos2KO_mouse_countDF (metabolite counts from fecal samples of a two-factor experiment)
- omu example metadata file c57_nos2KO_mouse_metadata (sample descriptors including a Treatment factor)

## Outputs

- data frame with KEGG-assigned hierarchical compound class metadata per metabolite
- input table ready for omu_summary / omu_anova statistical testing and ggplot2-based plotting

## How to apply

Load the packaged example data frame c57_nos2KO_mouse_countDF and its metadata file c57_nos2KO_mouse_metadata with omu's read.metabo function. Then apply assign_hierarchy with identifier = "KEGG" (optionally keep_unknowns = TRUE to retain compounds lacking KEGG numbers) to attach hierarchical compound class metadata to each metabolite. The resulting data frame carries a Treatment factor (e.g. Strep versus Mock) that can be passed to omu_summary (with log_transform = TRUE, p_adjust = "BH", test_type = "welch") or count_fold_changes for downstream statistics. Rationale: omu ships this two-factor fecal metabolomics dataset specifically so users can practice the assign_hierarchy → omu_summary → visualization chain without external data.

## Related tools

- **omu** (R package that ships the example mouse fecal metabolomics dataset and provides read.metabo, assign_hierarchy, and omu_summary for handling it) — github.com/connor-reid-tiffany/Omu
- **R** (runtime environment in which omu's data-loading and metadata-assignment functions are executed)

## Examples

```
library(omu); data(c57_nos2KO_mouse_countDF); data(c57_nos2KO_mouse_metadata); df <- assign_hierarchy(c57_nos2KO_mouse_countDF, identifier = "KEGG", keep_unknowns = TRUE)
```

## Evaluation signals

- c57_nos2KO_mouse_countDF and c57_nos2KO_mouse_metadata both load successfully from the omu package without errors
- assign_hierarchy adds KEGG-derived compound class columns; compounds without KEGG numbers remain only when keep_unknowns = TRUE
- the metadata contains a Treatment factor with at least two levels (e.g. Strep and Mock) usable as omu_summary's numerator/denominator
- the resulting data frame is accepted by omu_summary with log_transform = TRUE and p_adjust = "BH" without subsetting or reshaping

## Limitations

- The dataset is a packaged example from fecal samples collected in a two-factor experiment; it does not generalize to other sample types or designs.
- Compounds lacking KEGG identifiers are dropped unless keep_unknowns = TRUE is explicitly set during assign_hierarchy.

## Evidence

- [methods] packaged example metabolomics dataset of fecal samples: "Included with Omu is an example metabolomics dataset of data from fecal samples collected from a two factor experiment"
- [abstract] assign hierarchical metadata via KEGG: [evidence span withheld — anchor status: not_found]
- [methods] read.metabo loading recommendation: "it is recommended to use the ```read.metabo``` function to load it into R"
- [methods] keep_unknowns handling of missing KEGG ids: "```keep_unknowns = TRUE``` keeps compounds without KEGG numbers"
