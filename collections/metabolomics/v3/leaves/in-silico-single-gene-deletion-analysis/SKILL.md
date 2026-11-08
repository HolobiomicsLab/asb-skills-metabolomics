---
name: in-silico-single-gene-deletion-analysis
description: Use when you have a metabolic model (e.g., a Reconstructor-generated
  SBML GENRE like NCGM2.S1.xml) and need to identify which genes are essential versus
  non-essential for biomass production.
when_to_use_negative:
- Input is an unannotated genome or raw sequence data rather than a reconstructed,
  gap-filled metabolic model in SBML format
- The model does not yet support nonzero biomass flux, since deletion screening on
  a non-producing model yields meaningless essentiality calls (perform additional
  Type 3 gap-filling first)
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: COBRApy
  role: Loads the SBML GENRE as a Model object, runs flux balance analysis to verify
    biomass production, and executes single-gene deletion analysis to call essentiality
- name: Reconstructor
  role: Produces the COBRApy-compatible SBML GENRE from the annotated amino acid .fasta
    input, enabling direct downstream deletion analysis
  repo: https://github.com/emmamglass/reconstructor
- name: DIAMOND
  role: Aligns the annotated proteins to KEGG annotations during the upstream reconstruction
    step that supplies the model
  repo: https://github.com/bbuchfink/diamond
- name: Python
  role: Runtime environment (>= 3.9) in which Reconstructor and COBRApy are imported
    and the deletion workflow is scripted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:040cb29796638ab67a4a09edc0ef88751c5948fa740b4680a24d41f2fc18427d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - DIAMOND
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# in-silico single-gene deletion analysis

## Summary

Generates testable gene essentiality hypotheses by running COBRApy flux balance analysis and single-gene deletion screening on a genome-scale metabolic network reconstruction (GENRE), classifying each gene as essential or non-essential based on loss of biomass production.

## When to use

Use when you have a metabolic model (e.g., a Reconstructor-generated SBML GENRE like NCGM2.S1.xml) and need to identify which genes are essential versus non-essential for biomass production.

## When NOT to use

- Input is an unannotated genome or raw sequence data rather than a reconstructed, gap-filled metabolic model in SBML format
- The model does not yet support nonzero biomass flux, since deletion screening on a non-producing model yields meaningless essentiality calls (perform additional Type 3 gap-filling first)

## Inputs

- SBML genome-scale metabolic network reconstruction file (e.g., NCGM2.S1.xml) loadable by COBRApy
- Reconstructor draft GENRE built from an annotated amino acid .fasta file (Type 1 input) with parsimony-based gap-filling

## Outputs

- Essential-genes table (e.g., NCGM2.S1_essential_genes.csv) with gene identifiers and essential/non-essential labels
- Confirmation of nonzero biomass flux from FBA on the intact model

## How to apply

Load the SBML model (e.g., NCGM2.S1.xml) into COBRApy as a Model object, then perform flux balance analysis to confirm the model carries nonzero flux to the biomass objective before any deletion screening. Next, run cobra.flux_analysis.single_gene_deletion on the model, which knocks out each gene in silico and re-solves FBA to detect loss of biomass production. Classify each gene as essential if deletion abolishes biomass flux and non-essential otherwise. Finally, save the deletion results as a table (e.g., NCGM2.S1_essential_genes.csv) listing gene identifiers with their essentiality calls. The key rationale is that COBRApy compatibility of the Reconstructor pipeline means the draft GENRE can be analyzed directly without format conversion, enabling rapid hypothesis generation for previously unreconstructed strains.

## Related tools

- **COBRApy** (Loads the SBML GENRE as a Model object, runs flux balance analysis to verify biomass production, and executes single-gene deletion analysis to call essentiality)
- **Reconstructor** (Produces the COBRApy-compatible SBML GENRE from the annotated amino acid .fasta input, enabling direct downstream deletion analysis) — https://github.com/emmamglass/reconstructor
- **DIAMOND** (Aligns the annotated proteins to KEGG annotations during the upstream reconstruction step that supplies the model) — https://github.com/bbuchfink/diamond
- **Python** (Runtime environment (>= 3.9) in which Reconstructor and COBRApy are imported and the deletion workflow is scripted)

## Examples

```
python -m reconstructor --input_file NCGM2.S1.aa.fasta --file_type 1 --gram negative --media rich; then in Python: from cobra.io import read_sbml_model; from cobra.flux_analysis import single_gene_deletion; model = read_sbml_model('NCGM2.S1.xml')
```

## Evaluation signals

- NCGM2.S1.xml (or equivalent SBML output) exists and loads as a valid COBRApy Model object without errors
- Flux balance analysis on the intact model yields nonzero flux through the biomass objective
- The essential-genes CSV covers all genes in the model, with an essential or non-essential label for each gene
- Essential calls are consistent with the FBA criterion: essential genes abolish biomass production upon deletion, non-essential genes do not

## Limitations

- The test suite/installation step downloads resource files and can take ~1 hour depending on the system; the DIAMOND binary can be skipped with --skip-diamond
- The Reconstructor emmamglass repository is no longer updated; maintenance has moved to csbl/reconstructor
- If gap-filling is insufficient (e.g., minimum objective fraction --min_frac default 0.01 not met), the model may not carry biomass flux and deletion results would be invalid

## Evidence

- [abstract] COBRApy compatibility enabled gene essentiality analysis: [evidence span withheld — anchor status: not_found]
- [intro] Reconstructor is a COBRApy compatible automated GENRE building tool: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [methods] Run the installation test suite: "Run the following test to ensure reconstruction was installed correctly and is
functional:

shell
python -m reconstructor --test yes"
- [readme] Python import for COBRApy integration: "Reconstructor can be imported and used directly in Python for easy integration
with COBRApy analysis tools"
- [methods] Python version requirement: "You must be running >= Python 3.9"
