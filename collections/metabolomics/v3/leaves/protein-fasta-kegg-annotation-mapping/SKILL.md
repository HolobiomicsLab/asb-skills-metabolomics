---
name: protein-fasta-kegg-annotation-mapping
description: Use when starting from an annotated protein .fasta file for a bacterial
  strain with no existing reconstruction and you need a draft GENRE that can be loaded
  into COBRApy for downstream flux analysis.
when_to_use_negative:
- You already have BLASTp/DIAMOND tabular hits rather than a protein .fasta — use
  Type 2 input instead
- You already possess an SBML GENRE that only needs additional gap-filling — use Type
  3 input instead
- The target is not a bacterial genome-scale metabolic reconstruction or you cannot
  supply an annotated amino acid .fasta
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Reconstructor
  role: Automated COBRApy-compatible GENRE building tool from annotated amino acid
    .fasta files based on KEGG annotations; runs the reconstruction and parsimony-based
    gap-filling
  repo: https://github.com/emmamglass/reconstructor
- name: DIAMOND
  role: Sequence aligner used by Reconstructor (Type 1 input) to align protein sequences
    to KEGG annotations; binary and database auto-downloaded by the test suite unless
    --skip-diamond is used
  repo: https://github.com/bbuchfink/diamond
- name: COBRApy
  role: Loads the exported SBML GENRE as a Model object for downstream flux balance
    analysis and gene deletion screening
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:8094ccb379fedec1b383dfd1e2183c1f5195aa495249825105eeb58db2d82ef2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - DIAMOND
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# protein-fasta-kegg-annotation-mapping

## Summary

Map an annotated amino acid .fasta to KEGG annotations via DIAMOND alignment using Reconstructor (Type 1 input), producing a draft genome-scale metabolic network reconstruction (GENRE) with parsimony-based gap-filling exported as SBML, compatible with COBRApy.

## When to use

Use when starting from an annotated protein .fasta file for a bacterial strain with no existing reconstruction and you need a draft GENRE that can be loaded into COBRApy for downstream flux analysis.

## When NOT to use

- You already have BLASTp/DIAMOND tabular hits rather than a protein .fasta — use Type 2 input instead
- You already possess an SBML GENRE that only needs additional gap-filling — use Type 3 input instead
- The target is not a bacterial genome-scale metabolic reconstruction or you cannot supply an annotated amino acid .fasta

## Inputs

- Annotated amino acid .fasta file (Type 1 input) for the target organism, e.g. NCGM2.S1 aa .fasta
- Gram classification of the organism (positive or negative)
- Media condition (default 'rich', or ModelSEED-namespace metabolite list ending in _e)
- Optional: DIAMOND/BLASTp tabular hits output (.hits.out, Type 2 input)
- Optional: existing SBML GENRE (.sbml, Type 3 input) for further gap-filling

## Outputs

- Draft GENRE exported as an SBML file (e.g. NCGM2.S1.xml) with ModelSEED-consistent naming
- COBRApy-loadable Model object supporting flux balance analysis to a biomass objective

## How to apply

Install the Reconstructor Python package with pip (Python >= 3.9) and run 'python -m reconstructor --test yes' to download resource files, including the modified ModelSEED biochemistry universal database (universal.pickle) and a DIAMOND binary/database (skippable with --skip-diamond). Supply the annotated amino acid .fasta as Type 1 input (--file_type 1) so Reconstructor runs DIAMOND to align proteins to KEGG annotations and builds a draft GENRE gap-filled on the principles of parsimony. Specify the correct Gram classification (--gram negative or positive) and media condition (default 'rich'; custom media as comma-separated ModelSEED cpdXXXXX_e metabolite strings) since these constrain the reconstruction. Reconstructor exports the result as an SBML file with ModelSEED-consistent reaction and metabolite naming, which loads directly as a COBRApy Model object. Optional follow-up uses file_type 2 (BLASTp/DIAMOND output) or file_type 3 (additional gap-filling of an existing .sbml) if more gap-filling is needed.

## Related tools

- **Reconstructor** (Automated COBRApy-compatible GENRE building tool from annotated amino acid .fasta files based on KEGG annotations; runs the reconstruction and parsimony-based gap-filling) — https://github.com/emmamglass/reconstructor
- **DIAMOND** (Sequence aligner used by Reconstructor (Type 1 input) to align protein sequences to KEGG annotations; binary and database auto-downloaded by the test suite unless --skip-diamond is used) — https://github.com/bbuchfink/diamond
- **COBRApy** (Loads the exported SBML GENRE as a Model object for downstream flux balance analysis and gene deletion screening)

## Examples

```
python -m reconstructor --input_file NCGM2.S1.aa.fasta --file_type 1 --gram negative --media rich
```

## Evaluation signals

- The output SBML file (e.g. NCGM2.S1.xml) exists and is a valid SBML model loadable by COBRApy
- The loaded model supports nonzero flux to the biomass objective under flux balance analysis before any deletion screening
- Reaction and metabolite identifiers in the model follow ModelSEED biochemistry database naming conventions
- The installation check 'python -m reconstructor --test yes' completes, confirming Reconstructor is functional and resource files (universal.pickle, DIAMOND database/binary) downloaded

## Limitations

- The emmamglass/reconstructor repository is no longer updated; maintenance has moved to csbl/reconstructor
- The test suite/download step takes ~1 hour and auto-downloads a DIAMOND binary unless --skip-diamond is passed
- Gram classification must be supplied (default = positive) and an incorrect call will bias the reconstruction
- Documentation is still under development; users are directed to open GitHub issues for problems

## Evidence

- [abstract] Reconstructor generates SBML GENREs from three input types: "Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE that can be further gap-filled"
- [intro] KEGG annotation basis: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [methods] Installation and test suite with resource download: "Run the following test to ensure reconstruction was installed correctly and is
functional:

shell
python -m reconstructor --test yes
"
- [methods] ModelSEED universal database: "The universal reaction database used in `Reconstructor` is a modified version of
the [ModelSEED biochemistry database](https://github.com/ModelSEED/ModelSEEDDatabase)"
- [readme] Type 1 CLI usage example: [evidence span withheld — anchor status: external_doc]
