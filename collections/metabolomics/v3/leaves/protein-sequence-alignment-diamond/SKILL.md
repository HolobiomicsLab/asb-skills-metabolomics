---
name: protein-sequence-alignment-diamond
description: Use when the input is an annotated amino acid .fasta file (Type 1 input)
  and KEGG-based gene annotations are needed to seed a draft genome-scale metabolic
  network reconstruction.
when_to_use_negative:
- Input is already DIAMOND BLASTp output — use Type 2 input (--file_type 2) instead
  of re-aligning
- You already have an existing SBML GENRE needing further gap-filling — use Type 3
  input rather than sequence alignment
- Only transcript or whole-genome nucleotide data without annotated protein sequences
  is available (Reconstructor expects annotated amino acid .fasta files)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0622
tools:
- name: DIAMOND
  role: Sequence aligner performing BLASTp of protein sequences against the KEGG reference
    database
  repo: https://github.com/bbuchfink/diamond
- name: Reconstructor
  role: Orchestrates the alignment and translates DIAMOND KEGG hits into ModelSEED
    reactions for the draft GENRE
  repo: http://github.com/emmamglass/reconstructor
- name: COBRApy
  role: Loads universal.pickle as a Model object into which gene-hit reactions are
    placed
- name: ModelSEED biochemistry database
  role: Source of the curated universal reaction database (universal.pickle) whose
    reaction IDs the KEGG hits map to
  repo: https://github.com/ModelSEED/ModelSEEDDatabase
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:2b4b0d0c42d501cfc821d5b14425c6f5ae419a3ca55c6ea5cbecba026c272772
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - COBRApy
  - DIAMOND
  - ModelSEED biochemistry database
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# protein-sequence-alignment-diamond

## Summary

Aligns an annotated amino-acid FASTA against a KEGG reference database with DIAMOND BLASTp to produce gene hits that anchor a draft GENRE in Reconstructor. DIAMOND provides BLAST-level sensitivity at 100x–10,000x speed for big protein sequence data.

## When to use

Use when the input is an annotated amino acid .fasta file (Type 1 input) and KEGG-based gene annotations are needed to seed a draft genome-scale metabolic network reconstruction.

## When NOT to use

- Input is already DIAMOND BLASTp output — use Type 2 input (--file_type 2) instead of re-aligning
- You already have an existing SBML GENRE needing further gap-filling — use Type 3 input rather than sequence alignment
- Only transcript or whole-genome nucleotide data without annotated protein sequences is available (Reconstructor expects annotated amino acid .fasta files)

## Inputs

- Annotated amino acid .fasta file (Type 1 input), e.g. from NCBI RefSeq
- KEGG reference DIAMOND database downloaded during Reconstructor installation
- DIAMOND binary (auto-downloaded by the test suite unless --skip-diamond is set)

## Outputs

- DIAMOND BLASTp KEGG gene hits (Type 2-style hits output)
- Draft GENRE of gene-associated ModelSEED reactions plus media-condition reactions, serialized via pickle/SBML
- Tab-separated provenance table of reaction-to-gene hits and DIAMOND alignments

## How to apply

Run Reconstructor's reconstruct function on the Type 1 FASTA so DIAMOND BLASTp aligns the protein sequences against the KEGG reference database; the test suite automatically downloads both a DIAMOND database and a DIAMOND binary, or a pre-installed binary can be used via --skip-diamond. Translate the resulting DIAMOND KEGG gene hits into ModelSEED reactions by loading universal.pickle (a curated, mass-balanced version of the ModelSEED biochemistry database) as a COBRApy Model object. Attach gene-reaction associations so each retained reaction is traceable to an annotated gene and record alignment provenance in a tab-separated table. Parallelize blasting with the --cpu threads option for strain-scale workflows (e.g., 20 reconstructions). Use the KEGG organism code via --org when available and specify --gram (positive or negative) since the Gram classification shapes the draft model.

## Related tools

- **DIAMOND** (Sequence aligner performing BLASTp of protein sequences against the KEGG reference database) — https://github.com/bbuchfink/diamond
- **Reconstructor** (Orchestrates the alignment and translates DIAMOND KEGG hits into ModelSEED reactions for the draft GENRE) — http://github.com/emmamglass/reconstructor
- **COBRApy** (Loads universal.pickle as a Model object into which gene-hit reactions are placed)
- **ModelSEED biochemistry database** (Source of the curated universal reaction database (universal.pickle) whose reaction IDs the KEGG hits map to) — https://github.com/ModelSEED/ModelSEEDDatabase

## Examples

```
python -m reconstructor --input_file Osplanchnicus.aa.fasta --file_type 1 --gram negative --media rich --cpu 4
```

## Evaluation signals

- The test suite python -m reconstructor --test yes passes, confirming the DIAMOND database/binary installation is functional
- The saved draft model loads back as a COBRApy Model object
- Every reaction in the draft model is either gene-associated (traceable to a DIAMOND hit) or a media-condition exchange/boundary reaction
- Provenance table maps each retained reaction to an originating gene hit and DIAMOND alignment record
- Downstream, gap-filled models reproduce strain/species/class-level clustering (e.g., NMDS of flux samples, Fig. 1C pattern)

## Limitations

- Installation test suite takes ~1 hour and auto-downloads a DIAMOND database and binary unless --skip-diamond is used
- Draft quality depends on KEGG annotation coverage; parsimony-based gap-filling (Type 3) may still be required to restore objective flux
- This repository (emmamglass/reconstructor) is no longer updated; maintenance moved to csbl/reconstructor

## Evidence

- [intro] Reconstructor is an automated GENRE building tool from annotated amino acid .fasta files based on KEGG annotations: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [abstract] Type 1 input FASTA aligned with DIAMOND BLASTp to KEGG: "Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE"
- [other] DIAMOND performance characteristics: [evidence span withheld — anchor status: external_doc]
- [methods] Universal reaction database derived from ModelSEED: "The universal reaction database used in `Reconstructor` is a modified version of the [ModelSEED biochemistry database](https://github.com/ModelSEED/ModelSEEDDatabase)"
- [readme] Test suite auto-downloads DIAMOND database and binary: [evidence span withheld — anchor status: not_found]
- [readme] Type 1 CLI invocation with DIAMOND-backed reconstruction: [evidence span withheld — anchor status: external_doc]
