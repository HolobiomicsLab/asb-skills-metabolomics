---
name: metabolomics-statistics-subsetting
description: Use when a metabolomics data frame with KEGG compound IDs has been statistically
  tested (e.g. omu_summary student's t-test with BH adjustment) and only metabolites
  of interest (e.g. adjusted p <= 0.05) should be carried into KEGG enzyme/gene annotation.
when_to_use_negative:
- The input is a raw untested metabolomics count table that has not yet been analyzed
  with omu_summary or omu_anova — run the statistical tests first.
- Compounds lacking KEGG numbers are the target of interest; keep_unknowns/KEGG annotation
  paths handle those separately and subsetting on adjusted p values will not add identifiers.
- The goal is compound hierarchy classification alone without enzyme/gene annotation
  — assign_hierarchy with a KEGG identifier suffices without the KEGG_gather subsetting
  pipeline.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: omu
  role: R package providing read.metabo, omu_summary, KEGG_gather and assign_hierarchy
    used to load, statistically test, subset, and annotate the metabolomics data
  repo: github.com/connor-reid-tiffany/Omu
- name: KEGGREST
  role: provides keggGet, called by KEGG_gather against the KEGG API to retrieve orthology
    and gene metadata
- name: read.metabo
  role: loads the metabolite count data frame into R so the KEGG column has the proper
    class before subsetting
  repo: github.com/connor-reid-tiffany/Omu
- name: assign_hierarchy
  role: attaches enzyme metadata via the 'KO_Number' identifier and organism metadata
    to the subsetted annotation table
  repo: github.com/connor-reid-tiffany/Omu
- name: R
  role: language in which the entire load-test-subset-annotate workflow is executed
  repo: github.com/connor-reid-tiffany/Omu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:3ec6e9fc85f94c659a94f1be01e2bac558f424af72eb16b595eca62a7bb77d4d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - R
  - assign_hierarchy
  - omu
  - read.metabo
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolomics-statistics-subsetting

## Summary

A workflow for reducing an omu metabolomics count data frame (with KEGG compound identifiers) to statistically significant metabolites of interest, so that downstream KEGG orthology and gene annotation via KEGG_gather/assign_hierarchy is performed only on biologically relevant compounds.

## When to use

Use when a metabolomics data frame with KEGG compound IDs has been statistically tested (e.g. omu_summary student's t-test with BH adjustment) and only metabolites of interest (e.g. adjusted p <= 0.05) should be carried into KEGG enzyme/gene annotation.

## When NOT to use

- The input is a raw untested metabolomics count table that has not yet been analyzed with omu_summary or omu_anova — run the statistical tests first.
- Compounds lacking KEGG numbers are the target of interest; keep_unknowns/KEGG annotation paths handle those separately and subsetting on adjusted p values will not add identifiers.
- The goal is compound hierarchy classification alone without enzyme/gene annotation — assign_hierarchy with a KEGG identifier suffices without the KEGG_gather subsetting pipeline.

## Inputs

- metabolomics count data frame with a KEGG compound identifier column, loaded via omu read.metabo
- metadata file for the two-factor experiment design (e.g. c57_nos2KO_mouse_metadata)
- omu_summary output containing per-compound statistics and BH-adjusted p values

## Outputs

- subset data frame restricted to metabolites of interest (e.g. adjusted p <= 0.05)
- data frame with appended functional orthology and KO_numbers columns from KEGG_gather
- combined orthology/gene annotation table saved as kegg_orthology_gene_annotations.csv

## How to apply

First load the metabolite statistics data frame with omu's read.metabo so the KEGG column has the proper class, and generate statistics with omu_summary (e.g. student's t-test with p_adjust = 'BH'). Then subset the data frame to metabolites of interest — for example compounds with adjusted p values <= 0.05 — because compounds can have multiple functional orthologies associated with them and running KEGG_gather on all compounds inflates API calls and output size. On the subset, run KEGG_gather to append functional orthology and KO_numbers columns, run it again for gene metadata, and finish with assign_hierarchy using the 'KO_Number' identifier to attach enzyme and organism metadata. The result is saved as a combined orthology/gene annotation table (e.g. kegg_orthology_gene_annotations.csv).

## Related tools

- **omu** (R package providing read.metabo, omu_summary, KEGG_gather and assign_hierarchy used to load, statistically test, subset, and annotate the metabolomics data) — github.com/connor-reid-tiffany/Omu
- **KEGGREST** (provides keggGet, called by KEGG_gather against the KEGG API to retrieve orthology and gene metadata)
- **read.metabo** (loads the metabolite count data frame into R so the KEGG column has the proper class before subsetting) — github.com/connor-reid-tiffany/Omu
- **assign_hierarchy** (attaches enzyme metadata via the 'KO_Number' identifier and organism metadata to the subsetted annotation table) — github.com/connor-reid-tiffany/Omu
- **R** (language in which the entire load-test-subset-annotate workflow is executed) — github.com/connor-reid-tiffany/Omu

## Examples

```
sig <- omu_summary(count_data = countDF, metadata = meta, column = "treatment", test_type = "students", p_adjust = "BH"); sig_sub <- sig[sig$adjusted_p <= 0.05, ]
```

## Evaluation signals

- The output data frame has functional orthologies and KO_numbers columns added, as the methods state it should after KEGG_gather.
- All rows in the subset satisfy the significance criterion (e.g. adjusted p <= 0.05) computed with p_adjust = 'BH'.
- Every retained compound row carries a KEGG compound identifier of the proper class established by read.metabo.
- The final table (kegg_orthology_gene_annotations.csv) contains both enzyme metadata (from assign_hierarchy with 'KO_Number') and organism/gene metadata for the organism of interest (e.g. Pseudomonas spp.).

## Limitations

- Compounds can have multiple functional orthologies associated with them, so subsetting before KEGG_gather is needed to keep the annotation table manageable.
- Compounds without KEGG numbers cannot be annotated unless keep_unknowns = TRUE is used, and even then they carry no orthology information.
- p value adjustment method must be chosen explicitly (e.g. Benjamini & Hochberg with p_adjust = 'BH'); a different choice changes which metabolites of interest are retained.

## Evidence

- [abstract] The KEGG_gather function can get all known enzyme orthology data and gene data associated with each metabolite from the KEGG database: "The KEGG_gather function can get all known enzyme orthology data and gene data associated with each metabolite from the KEGG database"
- [methods] read.metabo load recommendation: "it is recommended to use the ```read.metabo``` function to load it into R"
- [methods] BH adjustment parameter: "a p value adjustment method of Benjamini & Hochberg with the argument ```p_adjust = "BH"```"
- [other] rationale for subsetting before KEGG_gather: [evidence span withheld — anchor status: not_found]
