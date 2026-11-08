---
name: kegg-gene-to-modelseed-reaction-mapping
description: Use when the input is an annotated amino acid FASTA (Type 1) and the
  goal is a KEGG-annotation-based draft GENRE whose reactions are traceable to genes
  via ModelSEED reaction IDs.
when_to_use_negative:
- The input is already a BLASTp output (Type 2) or an existing SBML GENRE (Type 3)
  — use the corresponding Reconstructor input path instead of re-running DIAMOND on
  a FASTA
- The input contains no gene annotations (e.g., unannotated nucleotide sequence) and
  must first be functionally annotated before KEGG-based reaction mapping
- A highly curated, manually reconstructed model is needed rather than an automated
  KEGG/ModelSEED-derived draft
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0602
tools:
- name: Reconstructor
  role: Automated COBRApy-compatible GENRE building tool whose reconstruct function
    maps DIAMOND KEGG gene hits to ModelSEED reactions and assembles the draft model
  repo: https://github.com/emmamglass/reconstructor
- name: DIAMOND
  role: Sequence aligner that BLASTp-aligns the input amino-acid FASTA against the
    KEGG reference database to produce gene hits
  repo: https://github.com/bbuchfink/diamond
- name: COBRApy
  role: Loads the universal.pickle reaction database as a Model object and validates
    that the draft model reloads as a COBRApy Model
- name: ModelSEED biochemistry database
  role: Source of reaction and metabolite identifiers used to translate KEGG gene
    hits into reactions in the universal database
  repo: https://github.com/ModelSEED/ModelSEEDDatabase
- name: Python
  role: Runtime environment (>= Python 3.9) for running Reconstructor and pickle serialization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:087511a26807c86fe2daa329128e483262abec7651b86d37f246f5d4e6c98c54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - DIAMOND
  - ModelSEED biochemistry database
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kegg-gene-to-modelseed-reaction-mapping

## Summary

Translates DIAMOND BLASTp KEGG gene hits into ModelSEED reactions to build a draft genome-scale metabolic network reconstruction (GENRE) with Reconstructor, using the curated universal.pickle reaction database and COBRApy.

## When to use

Use when the input is an annotated amino acid FASTA (Type 1) and the goal is a KEGG-annotation-based draft GENRE whose reactions are traceable to genes via ModelSEED reaction IDs.

## When NOT to use

- The input is already a BLASTp output (Type 2) or an existing SBML GENRE (Type 3) — use the corresponding Reconstructor input path instead of re-running DIAMOND on a FASTA
- The input contains no gene annotations (e.g., unannotated nucleotide sequence) and must first be functionally annotated before KEGG-based reaction mapping
- A highly curated, manually reconstructed model is needed rather than an automated KEGG/ModelSEED-derived draft

## Inputs

- Annotated amino acid .fasta file (Type 1 input)
- Gram classification (positive or negative)
- Media condition ('rich' or comma-separated ModelSEED-namespace metabolite list ending in _e)
- universal.pickle curated universal reaction database (modified ModelSEED biochemistry database)
- KEGG reference database (DIAMOND database downloaded during installation)

## Outputs

- Draft GENRE (SBML/pickled COBRApy Model) containing only gene-associated and media-condition reactions
- Tab-separated provenance table of each reaction's originating gene hit and DIAMOND alignment

## How to apply

Run Reconstructor's reconstruct function on the annotated amino-acid FASTA, which aligns protein sequences to the KEGG reference database with DIAMOND BLASTp. Load the curated universal reaction database (universal.pickle, a modified version of the ModelSEED biochemistry database) as a COBRApy Model object, and translate each DIAMOND KEGG gene hit into its corresponding ModelSEED reactions. Attach gene-reaction associations so every retained reaction is traceable to an annotated gene. Add medium-condition exchange and boundary reactions (e.g., rich media or a comma-separated list of ModelSEED-namespace metabolites ending in _e such as 'cpd00001_e') on top of the gene-associated reactions, then serialize the draft GENRE with pickle.dump and record a tab-separated provenance table of gene hits and DIAMOND alignments.

## Related tools

- **Reconstructor** (Automated COBRApy-compatible GENRE building tool whose reconstruct function maps DIAMOND KEGG gene hits to ModelSEED reactions and assembles the draft model) — https://github.com/emmamglass/reconstructor
- **DIAMOND** (Sequence aligner that BLASTp-aligns the input amino-acid FASTA against the KEGG reference database to produce gene hits) — https://github.com/bbuchfink/diamond
- **COBRApy** (Loads the universal.pickle reaction database as a Model object and validates that the draft model reloads as a COBRApy Model)
- **ModelSEED biochemistry database** (Source of reaction and metabolite identifiers used to translate KEGG gene hits into reactions in the universal database) — https://github.com/ModelSEED/ModelSEEDDatabase
- **Python** (Runtime environment (>= Python 3.9) for running Reconstructor and pickle serialization)

## Examples

```
python -m reconstructor --input_file Osplanchnicus.aa.fasta --file_type 1 --gram negative --media rich
```

## Evaluation signals

- The saved draft model loads back as a COBRApy Model object
- Every reaction in the draft model is either gene-associated (traceable to a DIAMOND KEGG hit) or a media-condition exchange/boundary reaction
- The tab-separated provenance table accounts for each retained reaction with its originating gene hit and DIAMOND alignment record
- Reaction and metabolite identifiers follow ModelSEED biochemistry namespace conventions
- Installation and pipeline functionality pass via python -m reconstructor --test yes

## Limitations

- Draft GENREs built solely from gene-associated reactions plus media reactions can have gaps and may require additional parsimony-based gap-filling (e.g., via --file_type 3 with --media)
- universal.pickle is a modified, curated subset of ModelSEED (mass-imbalanced reactions removed, poorly defined formulas corrected), so reactions outside it cannot be mapped
- DIAMOND installation is optional; if the binary is not downloaded during the test suite (--skip-diamond), Type 1 FASTA processing must handle that dependency
- The documentation is still under development (per the README warning)

## Evidence

- [methods] DIAMOND translates KEGG hits into ModelSEED reactions: [evidence span withheld — anchor status: not_found]
- [methods] Universal database from ModelSEED: "The universal reaction database used in `Reconstructor` is a modified version of the [ModelSEED biochemistry database]"
- [methods] Locate universal.pickle: "locate the universal.pickle file that was downloaded during the `Reconstructor` installation phase on your local computer"
- [readme] Type 1 CLI invocation: [evidence span withheld — anchor status: external_doc]
- [intro] Reconstructor is a KEGG-based automated GENRE builder: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
