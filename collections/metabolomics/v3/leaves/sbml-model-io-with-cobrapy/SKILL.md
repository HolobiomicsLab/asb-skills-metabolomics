---
name: sbml-model-io-with-cobrapy
description: Use when you need to turn an annotated amino acid .fasta (or BLASTp output)
  into an SBML metabolic model and immediately interrogate it with COBRApy tools such
  as FBA or single_gene_deletion.
when_to_use_negative:
- The input is already a curated, ready-to-analyze model in COBRApy's native format
  and needs no reconstruction or further gap-filling
- You need reconstruction from nucleotide sequences or raw reads rather than annotated
  amino acid .fasta or BLASTp output, since Reconstructor only accepts these input
  types
- The target organism requires annotations from databases other than KEGG, since Reconstructor
  builds from KEGG annotations and the ModelSEED biochemistry namespace
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Reconstructor
  role: Automated GENRE builder that produces SBML models compatible with COBRApy,
    using parsimony-based gap-filling
  repo: https://github.com/emmamglass/reconstructor
- name: COBRApy
  role: Loads the exported SBML model as a Model object and runs FBA and single-gene
    deletion analysis on it
- name: DIAMOND
  role: Sequence aligner used to align annotated .fasta proteins to KEGG annotations
    for Type 1 reconstructions
  repo: https://github.com/bbuchfink/diamond
- name: Python
  role: Runtime (>= 3.9) in which Reconstructor and COBRApy are installed and integrated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:3b2d979642f62ac7ce0836f0c7d7ca12b98355e1d3a727f11ab033ed9f22cd0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - DIAMOND
  - Python
  - Reconstructor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# sbml-model-io-with-cobrapy

## Summary

Generates an SBML genome-scale metabolic network reconstruction (GENRE) with Reconstructor and loads it into COBRApy as a Model object so downstream flux balance and gene deletion analyses can run directly in Python.

## When to use

Use when you need to turn an annotated amino acid .fasta (or BLASTp output) into an SBML metabolic model and immediately interrogate it with COBRApy tools such as FBA or single_gene_deletion.

## When NOT to use

- The input is already a curated, ready-to-analyze model in COBRApy's native format and needs no reconstruction or further gap-filling
- You need reconstruction from nucleotide sequences or raw reads rather than annotated amino acid .fasta or BLASTp output, since Reconstructor only accepts these input types
- The target organism requires annotations from databases other than KEGG, since Reconstructor builds from KEGG annotations and the ModelSEED biochemistry namespace

## Inputs

- annotated amino acid .fasta file (Type 1 input)
- DIAMOND BLASTp output file (Type 2 input)
- existing SBML GENRE (.sbml) for additional gap-filling (Type 3 input)
- Gram classification (positive or negative)
- media condition (default 'rich', ModelSEED cpdXXXXX_e metabolite strings)

## Outputs

- SBML GENRE file (e.g., NCGM2.S1.xml)
- COBRApy Model object loaded from the SBML file
- FBA biomass flux confirmation
- essential/non-essential gene classification table (e.g., NCGM2.S1_essential_genes.csv)

## How to apply

Install Reconstructor via pip (Python >= 3.9) and run its test suite (python -m reconstructor --test yes) so the ModelSEED-derived universal.pickle reaction database and DIAMOND resources are downloaded. Choose the input type: annotated protein .fasta (file_type 1) triggers DIAMOND alignment to KEGG annotations, BLASTp hits (file_type 2), or an existing .sbml for further gap-filling (file_type 3), and set --gram negative/positive and --media rich as appropriate. Reconstructor applies parsimony-based gap-filling and exports the model as an SBML file. Load the resulting .sbml into COBRApy with cobra.io.read_sbml_model, confirm the model carries nonzero flux to the biomass objective via FBA, then run analyses such as cobra.flux_analysis.single_gene_deletion to label genes essential or non-essential.

## Related tools

- **Reconstructor** (Automated GENRE builder that produces SBML models compatible with COBRApy, using parsimony-based gap-filling) — https://github.com/emmamglass/reconstructor
- **COBRApy** (Loads the exported SBML model as a Model object and runs FBA and single-gene deletion analysis on it)
- **DIAMOND** (Sequence aligner used to align annotated .fasta proteins to KEGG annotations for Type 1 reconstructions) — https://github.com/bbuchfink/diamond
- **Python** (Runtime (>= 3.9) in which Reconstructor and COBRApy are installed and integrated)

## Examples

```
python -m reconstructor --input_file NCGM2.S1.aa.fasta --file_type 1 --gram negative --media rich
```

## Evaluation signals

- The output SBML file (e.g., NCGM2.S1.xml) exists and is a valid SBML model loadable by COBRApy
- FBA on the loaded model yields nonzero flux to the biomass objective before deletion screening
- Reaction and metabolite naming conventions in the model are consistent with the ModelSEED biochemistry database namespace (cpd/rx identifiers ending in _e for extracellular)
- The single-gene deletion output table covers every gene in the model with an essential or non-essential label
- The installation test suite (python -m reconstructor --test yes) passes, confirming the reconstruction environment is functional

## Limitations

- ModelSEED media conditions must be specified as comma-separated ModelSEED-namespace metabolites ending with _e; arbitrary media definitions are not accepted
- Gap-filling quality depends on the --min_frac/--max_frac objective fraction bounds (default 0.01 and 0.5); poorly chosen bounds can alter the draft model
- The Reconstructor documentation is 'still very much under development', and the original emmamglass/reconstructor repository is no longer updated (maintenance moved to csbl/reconstructor)
- Running the installation test suite downloads a DIAMOND database and binary (~1 hr) unless --skip-diamond is used

## Evidence

- [readme] Reconstructor is COBRApy compatible GENRE builder from README: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [readme] Python integration for COBRApy analysis: "Reconstructor can be imported and used directly in Python for easy integration
with COBRApy analysis tools:"
- [abstract] Three SBML input types: "Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE"
- [abstract] Parsimony gap-filling: "includes a gap-filling technique based on
the principles of parsimony"
- [methods] Python version requirement: "You must be running >= Python 3.9"
- [readme] CLI Type 1 invocation from README: [evidence span withheld — anchor status: external_doc]
