---
name: gene-essentiality-annotation
description: Use when you have a list of essential genes from single-gene deletion
  analysis of a metabolic reconstruction and want to map approved drugs onto those
  targets to prioritize repurposing hypotheses.
when_to_use_negative:
- The model has not been gap-filled/reconstructed and no validated essential-gene
  list exists from single-gene deletion analysis
- The drug source is not a curated target database (e.g. DrugBank) with protein-level
  associations, so gene-target intersection is not possible
- The goal is de novo drug discovery rather than repurposing drugs already cataloged
  against the organism's essential genes
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: COBRApy
  role: Performs in silico single-gene deletion on the reconstructed model to derive
    the essential gene list used for DrugBank intersection
- name: Reconstructor
  role: Produces the COBRApy-compatible genome-scale reconstruction (NCGM2.S1) whose
    essential genes feed the drug-target mapping
  repo: https://github.com/emmamglass/reconstructor
- name: Python
  role: Implements the DrugBank cross-reference, identifier reconciliation, intersection,
    labeling, and TSV export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:88cec61d5901719ee3c18730f24758ceaa6998368b2f154d476dd39241a303fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-essentiality-annotation

## Summary

Cross-references a model's in silico essential genes (from COBRApy single-gene deletion of a genome-scale metabolic network such as P. aeruginosa NCGM2.S1) against DrugBank drug-target associations to classify each drug as a known microbial growth inhibitor or a previously untested therapeutic hypothesis.

## When to use

Use when you have a list of essential genes from single-gene deletion analysis of a metabolic reconstruction and want to map approved drugs onto those targets to prioritize repurposing hypotheses.

## When NOT to use

- The model has not been gap-filled/reconstructed and no validated essential-gene list exists from single-gene deletion analysis
- The drug source is not a curated target database (e.g. DrugBank) with protein-level associations, so gene-target intersection is not possible
- The goal is de novo drug discovery rather than repurposing drugs already cataloged against the organism's essential genes

## Inputs

- Essential gene list from in silico single-gene deletion of the NCGM2.S1 COBRApy model
- Curated drug-target protein associations from the DrugBank database
- Identifier mapping between DrugBank protein targets and model gene names

## Outputs

- drug_gene_mappings.tsv with drug name, target essential gene, and label ('known inhibitor' vs 'untested hypothesis') columns
- Classified drug counts (7 known growth inhibitors, 13 previously untested hypotheses, per Fig. 1E)

## How to apply

Load the essential-gene list produced by in silico single-gene deletion analysis of the reconstruction (built with Reconstructor/COBRApy) and retrieve curated drug-target protein associations from DrugBank. Reconcile gene and protein identifier conventions between DrugBank targets and the model's genes before intersecting, since the two sources use different identifier namespaces. Assign each intersecting drug its essential target gene(s), then label each drug-gene pair as 'known inhibitor' (microbial growth inhibitor) or 'untested hypothesis' per the Fig. 1E classification. Correct application reproduces the reported counts of 7 known growth inhibitors and 13 previously untested drug hypotheses, and the annotated mapping is written as drug_gene_mappings.tsv with drug name, target essential gene, and label columns.

## Related tools

- **COBRApy** (Performs in silico single-gene deletion on the reconstructed model to derive the essential gene list used for DrugBank intersection)
- **Reconstructor** (Produces the COBRApy-compatible genome-scale reconstruction (NCGM2.S1) whose essential genes feed the drug-target mapping) — https://github.com/emmamglass/reconstructor
- **Python** (Implements the DrugBank cross-reference, identifier reconciliation, intersection, labeling, and TSV export)

## Examples

```
from cobra.flux_analysis import single_gene_deletion; essential = single_gene_deletion(model)
```

## Evaluation signals

- The drug-gene classification reproduces the reported counts of 7 known inhibitors and 13 previously untested hypotheses (Fig. 1E)
- drug_gene_mappings.tsv contains exactly the three required columns: drug name, target essential gene, and label
- Every drug-gene pair labeled in the table traces to a gene present in the COBRApy essential gene set (no orphan targets)
- DrugBank identifiers were reconciled to model gene identifiers before intersection, so no pairs are lost to namespace mismatches
- Every essential gene with a DrugBank association is represented by at least one drug-gene pair in the output

## Limitations

- Classification depends on DrugBank's curated associations; drugs lacking protein-target annotations in DrugBank cannot be surfaced by this mapping
- Gene/protein identifier conventions differ between DrugBank and the reconstruction, so reconciliation errors silently drop true drug-gene pairs
- The 7/13 split reflects the classification reported for Fig. 1E; 'untested hypotheses' are computational predictions, not validated growth-inhibition results

## Evidence

- [results] Of the drugs identified by mapping essential genes of the P. aeruginosa NCGM2.S1 reconstruction to DrugBank targets, 7 are known inhibitors of microbial growth and 13 had not been tested previously (Fig. 1E): [evidence span withheld — anchor status: not_found]
- [methods] Intersect DrugBank target genes with the NCGM2.S1 essential gene list to assign each drug the essential gene(s) it targets.: [evidence span withheld — anchor status: not_found]
- [methods] reconciling gene and protein identifier conventions: [evidence span withheld — anchor status: not_found]
- [methods] COBRApy single-gene deletion: [evidence span withheld — anchor status: not_found]
- [readme] Reconstructor produces the model: "Reconstructor` is a COBRApy compatible, automated GENRE building tool
from annotated aminoa acid .fasta files based on KEGG annotations."
