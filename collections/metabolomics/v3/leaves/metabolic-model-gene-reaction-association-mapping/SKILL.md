---
name: metabolic-model-gene-reaction-association-mapping
description: Use when you have an essential-gene list from single-gene deletion of
  a COBRApy/Reconstructor model and need to intersect it with DrugBank targets to
  enumerate drug-hypothesis counts (e.g. 7 known inhibitors, 13 untested).
when_to_use_negative:
- The metabolic model has not been gap-filled or essential genes have not yet been
  determined by single-gene deletion, so no essential-gene input exists
- The goal is metabolic reconstruction itself (building the GENRE from annotated amino
  acid .fasta files) rather than downstream drug-target mapping
- Drug candidates come from a source other than curated DrugBank target associations,
  so identifier reconciliation against DrugBank conventions would not apply
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0154
tools:
- name: COBRApy
  role: Runs the in silico single-gene deletion analysis on the reconstructed NCGM2.S1
    model to produce the essential-gene input
- name: Reconstructor
  role: COBRApy-compatible tool that produced the draft reconstruction with ModelSEED-consistent
    naming and parsimonious flux-based gap-filling underlying the essential-gene analysis
  repo: https://github.com/emmamglass/reconstructor
- name: Python
  role: Scripting environment for the DrugBank cross-reference, identifier reconciliation,
    intersection, labeling, and TSV export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:7e7d7e87b683bbac92e5dddf48e0b5296bee08a5345233892ac35220fde67d29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# metabolic-model-gene-reaction-association-mapping

## Summary

Maps essential genes from an in silico single-gene deletion analysis of a genome-scale metabolic reconstruction (e.g. P. aeruginosa NCGM2.S1 built with Reconstructor/COBRApy) onto curated DrugBank drug-target associations, classifying each drug as a known microbial growth inhibitor or a previously untested hypothesis (7 known vs 13 new in Fig. 1E).

## When to use

Use when you have an essential-gene list from single-gene deletion of a COBRApy/Reconstructor model and need to intersect it with DrugBank targets to enumerate drug-hypothesis counts (e.g. 7 known inhibitors, 13 untested).

## When NOT to use

- The metabolic model has not been gap-filled or essential genes have not yet been determined by single-gene deletion, so no essential-gene input exists
- The goal is metabolic reconstruction itself (building the GENRE from annotated amino acid .fasta files) rather than downstream drug-target mapping
- Drug candidates come from a source other than curated DrugBank target associations, so identifier reconciliation against DrugBank conventions would not apply

## Inputs

- List of essential genes from in silico single-gene deletion analysis of the NCGM2.S1 reconstruction (COBRApy)
- Curated drug-target protein association table from the DrugBank database
- Gene/protein identifier mapping for reconciling DrugBank target IDs with model gene IDs

## Outputs

- drug_gene_mappings.tsv: annotated table with drug name, target essential gene, and label columns
- Reproduced classification counts: 7 known microbial growth inhibitors and 13 previously untested drug hypotheses (Fig. 1E)

## How to apply

First obtain the essential gene list from in silico single-gene deletion analysis of the reconstruction (NCGM2.S1) run in COBRApy after Reconstructor-based gap-filling. Retrieve curated drug-target protein associations from DrugBank and reconcile gene and protein identifier conventions against the model's gene IDs before intersecting, since mismatches here are the main correctness risk. Intersect DrugBank target genes with the essential gene set so each drug is assigned the essential gene(s) it targets. Label each drug-gene pair as 'known inhibitor' (microbial growth inhibitor) or 'untested hypothesis' following the Fig. 1E classification, then verify the classification reproduces the reported 7 known growth inhibitors and 13 previously untested hypotheses before writing drug_gene_mappings.tsv with drug name, target essential gene, and label columns.

## Related tools

- **COBRApy** (Runs the in silico single-gene deletion analysis on the reconstructed NCGM2.S1 model to produce the essential-gene input)
- **Reconstructor** (COBRApy-compatible tool that produced the draft reconstruction with ModelSEED-consistent naming and parsimonious flux-based gap-filling underlying the essential-gene analysis) — https://github.com/emmamglass/reconstructor
- **Python** (Scripting environment for the DrugBank cross-reference, identifier reconciliation, intersection, labeling, and TSV export)

## Examples

```
from reconstructor import reconstruct; model = reconstruct('NCGM2.S1.fa', file_type=1, gram='negative'); # then COBRApy single_gene_deletion + intersect with DrugBank targets -> drug_gene_mappings.tsv
```

## Evaluation signals

- Classification counts match the reported Fig. 1E values exactly: 7 drugs labeled as known microbial growth inhibitors and 13 labeled as previously untested hypotheses
- Every drug-gene row in drug_gene_mappings.tsv targets a gene present in the NCGM2.S1 essential-gene list
- drug_gene_mappings.tsv contains the required columns (drug name, target essential gene, label) with no missing labels
- Identifier reconciliation is complete: no DrugBank target proteins are dropped solely because of gene/protein ID convention mismatch
- Total number of mapped drugs equals 7 + 13 = 20, matching the reported mapping

## Limitations

- Maintenance of the Reconstructor repository has moved to csbl/reconstructor; the emmamglass/reconstructor repository is no longer being updated
- Labeling depends on the 'known inhibitor' vs 'untested hypothesis' classification reported for Fig. 1E; new DrugBank versions may shift counts away from 7/13
- The Reconstructor documentation 'is still very much under development' per the README warning

## Evidence

- [methods] Drug-gene classification workflow: [evidence span withheld — anchor status: not_found]
- [results] Reported counts: [evidence span withheld — anchor status: not_found]
- [intro] Reconstructor role: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [readme] Repository maintenance moved: [evidence span withheld — anchor status: not_found]
