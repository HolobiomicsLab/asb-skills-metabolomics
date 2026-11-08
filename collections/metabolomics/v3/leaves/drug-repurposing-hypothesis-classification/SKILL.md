---
name: drug-repurposing-hypothesis-classification
description: Use when you have a validated list of essential genes from constraint-based
  deletion analysis of a metabolic model and a curated drug-target association table,
  and need to enumerate and label candidate drug-repurposing hypotheses.
when_to_use_negative:
- You have no experimentally or computationally derived essential-gene set for the
  target organism's metabolic model — the classification depends entirely on intersecting
  DrugBank targets with essential genes.
- The model has not undergone single-gene deletion analysis and gap-filling via Reconstructor/COBRApy,
  so its essential-gene predictions would not be reliable.
- You need quantitative drug-efficacy predictions (e.g., flux-inhibition modeling)
  rather than binary known-inhibitor/untested-hypothesis labeling.
tools:
- name: COBRApy
  role: Runs the in silico single-gene deletion analysis that yields the essential
    gene list used for the DrugBank intersection
  repo: https://github.com/emmamglass/reconstructor
- name: Reconstructor
  role: Produces the COBRApy-compatible genome-scale reconstruction (e.g., of P. aeruginosa
    NCGM2.S1) whose essential genes feed the drug-target mapping
  repo: https://github.com/emmamglass/reconstructor
- name: Python
  role: Scripting environment for reconciling identifiers, intersecting gene sets,
    labeling drug-gene pairs, and writing drug_gene_mappings.tsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:b6af1fd21ed465a9cfd0283518da0f9665aa76c746648281d427e5932546f047
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - Python
  - Reconstructor
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# drug-repurposing-hypothesis-classification

## Summary

Classifies drugs from curated drug-target databases (e.g., DrugBank) by intersecting their protein targets with essential genes identified via in silico single-gene deletion of a genome-scale metabolic model (e.g., the P. aeruginosa NCGM2.S1 reconstruction), separating known microbial growth inhibitors from previously untested repurposing hypotheses.

## When to use

Use when you have a validated list of essential genes from constraint-based deletion analysis of a metabolic model and a curated drug-target association table, and need to enumerate and label candidate drug-repurposing hypotheses.

## When NOT to use

- You have no experimentally or computationally derived essential-gene set for the target organism's metabolic model — the classification depends entirely on intersecting DrugBank targets with essential genes.
- The model has not undergone single-gene deletion analysis and gap-filling via Reconstructor/COBRApy, so its essential-gene predictions would not be reliable.
- You need quantitative drug-efficacy predictions (e.g., flux-inhibition modeling) rather than binary known-inhibitor/untested-hypothesis labeling.

## Inputs

- List of essential genes from single-gene deletion analysis of the NCGM2.S1 metabolic model (COBRApy output)
- Curated drug-target protein association table from the DrugBank database
- Identifier mapping between model gene loci and DrugBank protein targets

## Outputs

- drug_gene_mappings.tsv with columns: drug name, target essential gene, label ('known inhibitor' or 'untested hypothesis')
- Reproduced classification counts (7 known growth inhibitors, 13 previously untested hypotheses)

## How to apply

Load the essential-gene list produced by in silico single-gene deletion analysis of the reconstructed model in COBRApy, and retrieve curated drug-target protein associations from DrugBank. Reconcile gene and protein identifier conventions between the model (e.g., locus tags or ModelSEED-style IDs) and DrugBank target accessions so the cross-reference is unambiguous. Intersect the DrugBank target gene set with the essential gene list to assign each drug the essential gene(s) it targets. Label each drug-gene pair as 'known inhibitor' (microbial growth inhibitor) or 'untested hypothesis' according to the classification scheme used for Fig. 1E, then verify the classification reproduces the reported counts of 7 known growth inhibitors and 13 previously untested drug hypotheses before exporting the annotated table to drug_gene_mappings.tsv.

## Related tools

- **COBRApy** (Runs the in silico single-gene deletion analysis that yields the essential gene list used for the DrugBank intersection) — https://github.com/emmamglass/reconstructor
- **Reconstructor** (Produces the COBRApy-compatible genome-scale reconstruction (e.g., of P. aeruginosa NCGM2.S1) whose essential genes feed the drug-target mapping) — https://github.com/emmamglass/reconstructor
- **Python** (Scripting environment for reconciling identifiers, intersecting gene sets, labeling drug-gene pairs, and writing drug_gene_mappings.tsv)

## Examples

```
from cobra.flux_analysis import single_gene_deletion; ess = single_gene_deletion(model)
```

## Evaluation signals

- The number of drugs labeled 'known inhibitor' equals 7 and the number labeled 'untested hypothesis' equals 13, matching the Fig. 1E counts.
- Every drug-gene row in drug_gene_mappings.tsv has a non-empty target essential gene that appears in the single-gene deletion essential set.
- Each drug-gene pair carries exactly one of the two labels; no unlabeled or ambiguous rows remain.
- Gene/protein identifier reconciliation is lossless: every DrugBank target that maps to a model gene did so through an explicit identifier mapping, not string coincidence.
- Re-running the intersect-and-label script from the same inputs reproduces the identical mapping table (deterministic output).

## Limitations

- Counts (7 known inhibitors, 13 untested) are specific to the P. aeruginosa NCGM2.S1 reconstruction and DrugBank version used; they must be re-derived for other models or database releases.
- Classification is binary and does not predict efficacy, pharmacokinetics, or whether a DrugBank association reflects direct enzyme inhibition relevant to microbial viability.
- Identifier reconciliation between DrugBank protein targets and model gene loci is error-prone and may miss drug-gene pairs if mappings are incomplete.

## Evidence

- [results] Finding reproduced in Fig. 1E: [evidence span withheld — anchor status: not_found]
- [methods] Core intersection step: [evidence span withheld — anchor status: not_found]
- [methods] Identifier reconciliation requirement: [evidence span withheld — anchor status: not_found]
- [readme] Provenance of the essential gene list: "Reconstructor is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
