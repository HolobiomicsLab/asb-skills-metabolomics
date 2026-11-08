---
name: blastp-protein-sequence-alignment-against-kegg
description: Use when starting from annotated amino-acid .fasta files (Type 1 input)
  and KEGG ortholog assignments are needed as the basis for COBRApy-compatible draft
  GENRE building, or when pre-computed DIAMOND BLASTp output (Type 2 input) must be
  supplied instead.
when_to_use_negative:
- Input is an existing SBML GENRE that only needs additional gap-filling — use Reconstructor's
  Type 3 input instead of re-aligning sequences
- Input is an unannotated genome or nucleotide FASTA without protein annotations —
  gene calling/annotation must precede BLASTp alignment
- The target organism's metabolism cannot be captured by KEGG ortholog assignments,
  e.g. when taxonomy-specific biochemistry is absent from KEGG and the ModelSEED universal
  database
edam_operation: http://edamontology.org/operation_0371
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Reconstructor
  role: Wraps DIAMOND BLASTp alignment against KEGG references as part of automated
    GENRE building from annotated amino-acid .fasta files
  repo: https://github.com/csbl/reconstructor
- name: DIAMOND
  role: Sequence aligner performing BLASTp of annotated proteins against KEGG references
    to derive KEGG ortholog assignments
  repo: https://github.com/bbuchfink/diamond
- name: COBRApy
  role: Compatibility framework into which the alignment-derived draft metabolic model
    is built and analyzed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:eb87ef6ab72720259da301302dabe93c95afdf7fcf07e2f47219c61bbf0c5a4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csbl/reconstructor
  tools:
  - COBRApy
  - DIAMOND
  - Python
  - Reconstructor
  - pip
  edam_operation: http://edamontology.org/operation_0371
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# blastp-protein-sequence-alignment-against-kegg

## Summary

Aligns annotated amino-acid FASTA sequences against KEGG references using the DIAMOND sequence aligner to derive KEGG ortholog assignments, the annotation layer that drives Reconstructor's automated genome-scale metabolic network reconstruction (GENRE) pipeline.

## When to use

Use when starting from annotated amino-acid .fasta files (Type 1 input) and KEGG ortholog assignments are needed as the basis for COBRApy-compatible draft GENRE building, or when pre-computed DIAMOND BLASTp output (Type 2 input) must be supplied instead.

## When NOT to use

- Input is an existing SBML GENRE that only needs additional gap-filling — use Reconstructor's Type 3 input instead of re-aligning sequences
- Input is an unannotated genome or nucleotide FASTA without protein annotations — gene calling/annotation must precede BLASTp alignment
- The target organism's metabolism cannot be captured by KEGG ortholog assignments, e.g. when taxonomy-specific biochemistry is absent from KEGG and the ModelSEED universal database

## Inputs

- Annotated amino-acid FASTA file (.fasta/.fa, Type 1 input)
- DIAMOND BLASTp tabular hits output (Type 2 input, e.g. .hits.out)
- KEGG reference database and DIAMOND binary (downloaded during Reconstructor test suite)
- Gram classification (positive or negative) and media condition (e.g. 'rich', ModelSEED-namespace metabolites ending in _e)

## Outputs

- KEGG ortholog assignments per annotated protein (DIAMOND BLASTp hits)
- Draft COBRApy-compatible GENRE built from matched orthologs against the ModelSEED-based universal.pickle database
- Gap-filled, SBML-exported GENRE with ModelSEED-consistent reaction and metabolite naming

## How to apply

Install the Reconstructor Python package with pip (Python >= 3.9), then run `python -m reconstructor --test yes` so the DIAMOND database and DIAMOND binary are downloaded automatically (use `--skip-diamond` to supply your own binary). Pass an annotated amino-acid .fasta file with `--file_type 1`; Reconstructor internally runs DIAMOND BLASTp to align the annotated proteins against KEGG references and derive KEGG ortholog assignments. Specify the organism's Gram classification with `--gram positive|negative` and the medium with `--media` (default 'rich'). Matched KEGG orthologs are mapped onto the curated ModelSEED-based universal reaction database (universal.pickle) to draft a COBRApy-compatible model. If BLASTp hits were computed separately, supply the DIAMOND tabular output as Type 2 input (`--file_type 2`) to skip the alignment step.

## Related tools

- **Reconstructor** (Wraps DIAMOND BLASTp alignment against KEGG references as part of automated GENRE building from annotated amino-acid .fasta files) — https://github.com/csbl/reconstructor
- **DIAMOND** (Sequence aligner performing BLASTp of annotated proteins against KEGG references to derive KEGG ortholog assignments) — https://github.com/bbuchfink/diamond
- **COBRApy** (Compatibility framework into which the alignment-derived draft metabolic model is built and analyzed)

## Examples

```
python -m reconstructor --input_file Osplanchnicus.aa.fasta --file_type 1 --gram negative --media rich
```

## Evaluation signals

- Running `python -m reconstructor --test yes` completes successfully, confirming the DIAMOND database and binary were downloaded and alignment functionality works (~1hr runtime)
- BLASTp output is in DIAMOND tabular format accepted as Type 2 input (`--file_type 2`), e.g. `Osplanchnicus.hits.out`
- The resulting draft model builds against the ModelSEED universal.pickle database and uses ModelSEED metabolite namespace (e.g. 'cpd00001_e' for exchange metabolites)
- Gap-filling respects parsimony bounds with min_frac default 0.01 and max_frac default 0.5 of the objective
- Benchmarked 10-species models reach overall MEMOTE scores in the reported 83%-85% range (see reference/MEMOTE.zip)

## Limitations

- Maintenance has moved to the csbl/reconstructor repository; emmamglass/reconstructor is no longer updated
- The DIAMOND binary download during the test suite is automatic but optional — `--skip-diamond` requires manual DIAMOND installation
- Test suite/dependency download can take about 1 hour depending on the system
- Orthology is based solely on KEGG annotations, so pathway coverage depends on KEGG reference representation for the organism's taxonomy

## Evidence

- [intro] Reconstructor is a COBRApy compatible, automated GENRE building tool from annotated amino acid .fasta files based on KEGG annotations: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [abstract] Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE: "Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE that can be further gap-filled"
- [methods] The universal reaction database used in Reconstructor is a modified version of the ModelSEED biochemistry database: "The universal reaction database used in `Reconstructor` is a modified version of
the [ModelSEED biochemistry database](https://github.com/ModelSEED/ModelSEEDDatabase)"
- [readme] Running the test suite will automatically download a DIAMOND database and a DIAMOND binary for blasting: "Running the test suite will automatically download a DIAMOND database and a
> DIAMOND binary for blasting."
- [readme] Type 2 input builds GENRE from BLASTp hits via the reconstructor command line: [evidence span withheld — anchor status: external_doc]
