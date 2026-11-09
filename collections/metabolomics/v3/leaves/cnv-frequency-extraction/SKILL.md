---
name: cnv-frequency-extraction
description: Decision support for quantifying gene-level CNV deletion frequency in
  GutUDB colorectal cancer data and cross-referencing it against the RNA expression
  annotation to assess whether copy-number state and transcript level are coupled.
when_to_use_negative:
- The task concerns SNP or structural-variation (SV) calls rather than CNV deletion
  frequency — GutUDB tracks 72,248 SNPs and 1097 SVs separately from its 54,131 CNVs.
- A quantitative correlation coefficient or statistical test between CNV and RNA expression
  across a gene panel is required; this skill only captures GutUDB's annotated frequency
  value and the article's qualitative weak-correlation statement.
- The target disease is outside the 56 intestinal diseases catalogued in GutUDB or
  outside the colorectal cancer subset.
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GutUDB
  role: Source database queried via Browse/Query to retrieve gene-level CNV deletion
    frequency and RNA expression annotations for colorectal cancer
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:d8852c56c59df00817fd95b59b1e115ce739bcc5fb96b0496a3aa912c952c3d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cnv-frequency-extraction

## Summary

Extracts gene-specific copy number variation (CNV) deletion frequency values and paired RNA expression annotations from the GutUDB colorectal cancer genomics and transcriptomics layers, as demonstrated for METTL3 where a CNV deletion frequency of 0.04 co-occurs with significantly high RNA expression, supporting a weak CNV–RNA correlation conclusion.

## When to use

Decision support for quantifying gene-level CNV deletion frequency in GutUDB colorectal cancer data and cross-referencing it against the RNA expression annotation to assess whether copy-number state and transcript level are coupled.

## When NOT to use

- The task concerns SNP or structural-variation (SV) calls rather than CNV deletion frequency — GutUDB tracks 72,248 SNPs and 1097 SVs separately from its 54,131 CNVs.
- A quantitative correlation coefficient or statistical test between CNV and RNA expression across a gene panel is required; this skill only captures GutUDB's annotated frequency value and the article's qualitative weak-correlation statement.
- The target disease is outside the 56 intestinal diseases catalogued in GutUDB or outside the colorectal cancer subset.

## Inputs

- GutUDB colorectal cancer CNV frequency annotation for a target gene (e.g. METTL3, value 0.04)
- GutUDB transcriptomics RNA expression status annotation for the same gene (e.g. 'significantly high RNA expression')
- Article-reported CNV–RNA relationship statement for the gene (weak correlation)

## Outputs

- Structured JSON record (e.g. mettl3_cnv_rna_observations.json) containing the three gene-level observations: CNV deletion frequency, RNA expression annotation, and CNV–RNA correlation statement

## How to apply

Query the GutUDB repository (Ansanqi/GutUDB) using its Browse/Query functionality to locate the target gene — e.g. METTL3 — within the colorectal cancer subset of its genomics content, which aggregates 54,131 CNVs alongside 72,248 SNPs and 1097 structural variations. Retrieve the annotated CNV deletion frequency value (for METTL3 this is 0.04) and the paired transcriptomics annotation ('significantly high RNA expression') drawn from the 137 RNA-seq data sets / 7.9 million transcriptional profiles in GutUDB's transcriptomics layer. Record the qualitative CNV–RNA relationship the article reports (weak correlation for METTL3), then assemble all three observations into a single structured JSON record (e.g. mettl3_cnv_rna_observations.json) and verify that the file contains exactly those three elements with values matching the source article.

## Related tools

- **GutUDB** (Source database queried via Browse/Query to retrieve gene-level CNV deletion frequency and RNA expression annotations for colorectal cancer) — https://github.com/Ansanqi/GutUDB

## Examples

```
Browse GutUDB colorectal cancer CNV layer → locate METTL3 → record deletion frequency 0.04; then emit mettl3_cnv_rna_observations.json with {cnv_del_freq: 0.04, rna_expr: 'significantly high', correlation: 'weak'}
```

## Evaluation signals

- The output JSON contains exactly three records for METTL3: CNV deletion frequency 0.04, the 'significantly high RNA expression' annotation, and the weak CNV–RNA correlation statement.
- The CNV deletion frequency value in the JSON matches the value reported in the source article.
- The RNA expression annotation string in the JSON matches the GutUDB transcriptomics wording verbatim.
- The correlation statement reflects the article's qualitative conclusion, not a computed statistic.
- The JSON file parses without error and all required keys are present.

## Limitations

- GutUDB provides a frequency value and an expression label, not a raw statistical correlation measure; the 'weak correlation' conclusion is the article's qualitative interpretation.
- CNV extraction is limited to the disease subsets GutUDB covers (56 intestinal diseases across six species); other cancer types are out of scope.
- No changelog is available for the GutUDB repository, so version drift in annotation values cannot be tracked.

## Evidence

- [abstract] GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes: "GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes"
- [abstract] four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [abstract] 137 RNA‐seq data sets related to 7.9 million transcriptional profiles: "137 RNA‐seq data sets related to 7.9 million transcriptional profiles"
