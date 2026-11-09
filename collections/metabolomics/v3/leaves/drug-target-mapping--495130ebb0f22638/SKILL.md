---
name: drug-target-mapping
description: Use when you have an essential-gene list from single-gene deletion analysis
  of a metabolic model and want to enumerate approved/known drugs targeting those
  genes and flag which drug-gene hypotheses are novel.
when_to_use_negative:
- You need de novo drug discovery or compound scoring rather than repurposing existing
  DrugBank target annotations against a fixed essential-gene set
- Your gene list comes from expression or fitness data rather than model-based single-gene
  deletion, so the essentiality semantics differ from the Fig. 1E classification
- DrugBank access is unavailable and no equivalent curated drug-target association
  source can be reconciled to the model's gene identifiers
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_3407
tools:
- name: Reconstructor
  role: Automated COBRApy-compatible reconstruction tool whose gap-filled model supplies
    the essential gene list via single-gene deletion analysis
  repo: https://github.com/emmamglass/reconstructor
- name: COBRApy
  role: Framework used to run the in silico single-gene deletion analysis that identifies
    essential genes for drug mapping
  repo: https://github.com/opencobra/cobrapy
- name: Python
  role: Scripting environment for intersecting DrugBank targets with the essential
    gene set and writing drug_gene_mappings.tsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:060e364ff6e0a3bde29278698713d29f067026f934e4dac8efbf3957fa942b06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - Python
  - Reconstructor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# drug-target-mapping

## Summary

Cross-references essential genes identified by in silico single-gene deletion of a genome-scale metabolic reconstruction (e.g. Pseudomonas aeruginosa NCGM2.S1 built with Reconstructor/COBRApy) against DrugBank drug-target associations, then classifies each drug-gene pair as a known microbial growth inhibitor or a previously untested hypothesis.

## When to use

Use when you have an essential-gene list from single-gene deletion analysis of a metabolic model and want to enumerate approved/known drugs targeting those genes and flag which drug-gene hypotheses are novel.

## When NOT to use

- You need de novo drug discovery or compound scoring rather than repurposing existing DrugBank target annotations against a fixed essential-gene set
- Your gene list comes from expression or fitness data rather than model-based single-gene deletion, so the essentiality semantics differ from the Fig. 1E classification
- DrugBank access is unavailable and no equivalent curated drug-target association source can be reconciled to the model's gene identifiers

## Inputs

- Essential gene list from in silico single-gene deletion analysis of the NCGM2.S1 reconstruction (COBRApy)
- Curated drug-target protein associations from the DrugBank database
- Gene/protein identifier crosswalk reconciling model gene names with DrugBank targets

## Outputs

- Annotated drug-gene mapping table (drug name, target essential gene, label) written to drug_gene_mappings.tsv
- Class counts: 7 known microbial growth inhibitors and 13 previously untested drug hypotheses (reproducing Fig. 1E)

## How to apply

Start from the essential gene list produced by in silico single-gene deletion of the reconstructed model (NCGM2.S1 in COBRApy) and pull curated drug-target protein associations from DrugBank. Reconcile gene and protein identifier conventions between the model and DrugBank before intersecting, since mismatched identifiers will silently drop matches. Intersect DrugBank target genes with the essential gene set so each drug inherits the essential gene(s) it targets, then label each drug-gene pair as 'known inhibitor' (microbial growth inhibitor) or 'untested hypothesis' per the Fig. 1E classification. Validate the mapping by confirming the counts reproduce the published 7 known growth inhibitors and 13 previously untested drug hypotheses, and write the annotated table to drug_gene_mappings.tsv.

## Related tools

- **Reconstructor** (Automated COBRApy-compatible reconstruction tool whose gap-filled model supplies the essential gene list via single-gene deletion analysis) — https://github.com/emmamglass/reconstructor
- **COBRApy** (Framework used to run the in silico single-gene deletion analysis that identifies essential genes for drug mapping) — https://github.com/opencobra/cobrapy
- **Python** (Scripting environment for intersecting DrugBank targets with the essential gene set and writing drug_gene_mappings.tsv)

## Examples

```
python map_drug_targets.py --essential_genes ncgm2s1_essential_genes.txt --drugbank drugbank_targets.csv --id_crosswalk gene_protein_map.tsv --output drug_gene_mappings.tsv
```

## Evaluation signals

- The intersection of DrugBank target genes with the NCGM2.S1 essential gene list reproduces exactly 7 drugs classified as known inhibitors of microbial growth
- The classification reproduces exactly 13 drugs marked as previously untested hypotheses (matching Fig. 1E)
- drug_gene_mappings.tsv contains the three required columns (drug name, target essential gene, label) with no unlabeled drug-gene pairs
- Identifier reconciliation succeeded, i.e. no essential genes are lost solely due to gene/protein naming convention mismatches between the model and DrugBank

## Limitations

- Drug hypotheses are limited to targets present in DrugBank; drugs targeting essential genes absent from DrugBank will not surface
- The known-inhibitor vs untested labels reflect the classification reported for Fig. 1E of the source study and must be re-curated for a new organism or gene set
- The underlying reconstruction pipeline (Reconstructor) and its maintenance have moved; the emmamglass/reconstructor repository is no longer updated and users should reference csbl/reconstructor

## Evidence

- [other] Essential genes derived from model deletion analysis are mapped to DrugBank targets: [evidence span withheld — anchor status: not_found]
- [other] Expected output counts validate the mapping: [evidence span withheld — anchor status: not_found]
- [other] Identifier reconciliation is a required cross-referencing step: [evidence span withheld — anchor status: not_found]
- [readme] Model provenance: COBRApy-compatible reconstruction tool: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [readme] Reconstructor maintenance relocated; original repo stale: "Maintenance and updates for this project have moved to the [csbl/reconstructor](https://github.com/csbl/reconstructor) repository. Consequently this repository is no longer being updated."
