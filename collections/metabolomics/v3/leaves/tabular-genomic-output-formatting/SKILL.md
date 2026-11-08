---
name: tabular-genomic-output-formatting
description: Use when writing or validating the tab-delimited output of a gene/operon
  prediction step (e.g. MetaRon's Operon_File.tab) so records conform to the scaftig
  and gene prediction file format required by the 'op' process.
when_to_use_negative:
- Input is already a finalized operon table that only needs downstream antiSMASH/BOWTIE
  analysis rather than re-formatting
- The goal is functional or pathway annotation of operons — the article notes secondary
  metabolite prediction and pathway annotation are not integrated into the pipeline
  and were performed separately
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaRon
  role: Produces the tab-delimited Operon_File.tab operon predictions via 'metaron
    --process op'; format conventions are documented in its README
  repo: https://github.com/zaidissa/MetaRon
- name: prodigal
  role: Generates the gene prediction .tab file whose format the operon output must
    mirror (--tool 2)
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:f9e45fa370cce2dd1a3a7a0e3d08607788e577a812a451033096d30e77579da0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - MetaGeneMark
  - MetaRon
  - NNPP
  - Prodigal
  - prodigal
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tabular-genomic-output-formatting

## Summary

Emit and validate gene/operon predictions as tab-delimited genomic records, following the scaftig and gene prediction file conventions that MetaRon requires for Operon_File.tab. This ensures predicted operons (built from co-directional gene pairs with IGD <= 600 bp) are consumable by downstream tools like antiSMASH and BOWTIE/FMAP.

## When to use

Use when writing or validating the tab-delimited output of a gene/operon prediction step (e.g. MetaRon's Operon_File.tab) so records conform to the scaftig and gene prediction file format required by the 'op' process.

## When NOT to use

- Input is already a finalized operon table that only needs downstream antiSMASH/BOWTIE analysis rather than re-formatting
- The goal is functional or pathway annotation of operons — the article notes secondary metabolite prediction and pathway annotation are not integrated into the pipeline and were performed separately

## Inputs

- Gene prediction .tab file from MetaGeneMark or Prodigal (passed via --igp)
- Scaftigs FASTA file (passed via --isc)
- Predicted operon records from co-directional proximal gene clusters (IGD < 601 bp)

## Outputs

- Tab-delimited Operon_File.tab listing predicted metagenomic operons in the scaftig/gene-prediction file format

## How to apply

Run the operon prediction process ('op') so predictions are written to a tab-delimited Operon_File.tab in the designated output folder; the file carries the operonic information determined by co-directionality, IGD < 601 bp thresholding, and NNPP promoter evidence. Each record must list gene clusters restricted to adjacent same-strand pairs satisfying IGD <= 600 bp, mirroring the reference benchmark of 822 co-directional proximal clusters covering 2955 genes. Before accepting the output, verify the file exists, confirm it is genuinely tab-delimited (not space- or comma-separated), and check record structure against the scaftig and gene prediction file format MetaRon documents for the 'op' process. Because 'op' consumes gene prediction .tab files generated via MetaGeneMark (--tool 1) or Prodigal (--tool 2), the output format must stay round-trip compatible with those inputs.

## Related tools

- **MetaRon** (Produces the tab-delimited Operon_File.tab operon predictions via 'metaron --process op'; format conventions are documented in its README) — https://github.com/zaidissa/MetaRon
- **prodigal** (Generates the gene prediction .tab file whose format the operon output must mirror (--tool 2))

## Examples

```
metaron --sample ERR022075 --process op --igp ERR022075 --isc ERR022075_scaf.fa --tool 2 --output ~/path/to/output/directory/
```

## Evaluation signals

- Operon_File.tab exists in the specified output destination folder after the run
- The file parses as tab-delimited (fields separated by tabs, not spaces or commas)
- Every listed cluster contains only adjacent same-strand gene pairs with IGD <= 600 bp
- Record counts are consistent with the MetaRon benchmark (822 co-directional proximal clusters, 2955 genes on the reference dataset)
- Output records conform to the scaftig and gene prediction file format documented for the 'op' process

## Limitations

- Pipeline restricted to Linux and academic use only
- MetaRon requires Python (2.7 per README; article cites Python > 3.0), so environment mismatch can affect output generation
- NNPP's inability to identify a promoter can affect which genes pass into the final operon records

## Evidence

- [readme] Operon_File.tab output saved by MetaRon: "This will save metagenomic operon predictions ``Operon_File.tab``.  The prediction file will report the operonic information based on the above mentioned parameters."
- [readme] op process file format requirement: "1- If the selected --process is  'op', then please refer to the provided scaftig and gene prediction file format"
- [results] proximon clustering benchmark underlying the output: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
