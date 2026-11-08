---
name: gene-protein-reaction-association-assembly
description: Use when building a draft genome-scale metabolic model from annotated
  amino-acid FASTA (Type 1) or DIAMOND BLASTp hits (Type 2) and reactions must carry
  gene-reaction provenance from KEGG annotation.
when_to_use_negative:
- Input is already an existing SBML GENRE (Type 3) needing only additional gap-filling
  — no new GPR assembly from gene hits is required
- No annotated protein sequences or BLASTp hits are available, so reactions cannot
  be traced to genes (draft would lack GPR provenance)
- The task is curating the universal database itself rather than assembling a draft
  model — use UniversalCreation.py / curateuniversal.py instead
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3941
tools:
- name: Reconstructor
  role: Runs the reconstruct function that translates DIAMOND KEGG hits into ModelSEED
    reactions and attaches gene-reaction associations
  repo: https://github.com/emmamglass/reconstructor
- name: DIAMOND
  role: Aligns the input amino-acid FASTA against the KEGG reference database to produce
    the gene hits feeding GPR assembly
  repo: https://github.com/bbuchfink/diamond
- name: COBRApy
  role: Loads universal.pickle as a Model object so reactions and their GPRs can be
    manipulated and serialized
- name: ModelSEEDDatabase
  role: Source of the reaction and metabolite conventions used by the universal reaction
    database
  repo: https://github.com/ModelSEED/ModelSEEDDatabase
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:54e5a50aa69c0990e50faec10e4298914f2830fc2b94c8d8811181ad27c8707a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - DIAMOND
  - ModelSEEDDatabase
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gene-protein-reaction-association-assembly

## Summary

Assembly of gene-protein-reaction (GPR) associations during draft GENRE construction in Reconstructor: each DIAMOND BLASTp KEGG gene hit is mapped to a ModelSEED reaction (loaded from universal.pickle as a COBRApy Model) and the annotated gene is attached so every retained reaction is traceable to a gene of origin.

## When to use

Use when building a draft genome-scale metabolic model from annotated amino-acid FASTA (Type 1) or DIAMOND BLASTp hits (Type 2) and reactions must carry gene-reaction provenance from KEGG annotation.

## When NOT to use

- Input is already an existing SBML GENRE (Type 3) needing only additional gap-filling — no new GPR assembly from gene hits is required
- No annotated protein sequences or BLASTp hits are available, so reactions cannot be traced to genes (draft would lack GPR provenance)
- The task is curating the universal database itself rather than assembling a draft model — use UniversalCreation.py / curateuniversal.py instead

## Inputs

- annotated amino-acid .fasta file (Type 1 input), e.g. Osplanchnicus.aa.fasta
- DIAMOND BLASTp output hits file (Type 2 input), e.g. Osplanchnicus.hits.out
- universal.pickle: curated universal reaction database (modified ModelSEED biochemistry) loadable as a COBRApy Model
- KEGG annotations and gene names from the input proteome

## Outputs

- draft GENRE (COBRApy Model, serialized with pickle) containing only gene-associated and media-condition reactions
- tab-separated provenance table mapping each reaction to its originating gene hit and DIAMOND alignment
- GPR-annotated reactions carrying gene-reaction associations traceable to the input FASTA

## How to apply

Run DIAMOND BLASTp of the input protein sequences against the KEGG reference database (installed automatically by the Reconstructor test/download step). Load the curated universal reaction database from universal.pickle as a COBRApy Model; this universal is a modified version of the ModelSEED biochemistry database with poorly defined metabolite formulas corrected and mass-imbalanced reactions removed via curateuniversal.py. Translate each DIAMOND KEGG gene hit into its corresponding ModelSEED reaction, and attach gene-reaction associations so every retained reaction is traceable to an annotated gene from the FASTA. Keep only gene-associated reactions, then add medium-condition exchange and boundary reactions (e.g., 'rich' media or user-specified ModelSEED metabolite strings ending in _e). Serialize the draft with pickle.dump and record each reaction's originating gene hit and DIAMOND alignment in a tab-separated provenance table.

## Related tools

- **Reconstructor** (Runs the reconstruct function that translates DIAMOND KEGG hits into ModelSEED reactions and attaches gene-reaction associations) — https://github.com/emmamglass/reconstructor
- **DIAMOND** (Aligns the input amino-acid FASTA against the KEGG reference database to produce the gene hits feeding GPR assembly) — https://github.com/bbuchfink/diamond
- **COBRApy** (Loads universal.pickle as a Model object so reactions and their GPRs can be manipulated and serialized)
- **ModelSEEDDatabase** (Source of the reaction and metabolite conventions used by the universal reaction database) — https://github.com/ModelSEED/ModelSEEDDatabase

## Examples

```
python -m reconstructor --input_file Osplanchnicus.aa.fasta --file_type 1 --gram negative --media rich
```

## Evaluation signals

- The saved draft model loads back successfully as a COBRApy Model after pickle.dump
- Every reaction in the draft is either gene-associated (traceable to a DIAMOND KEGG hit) or media-condition (exchange/boundary)
- The tab-separated provenance table records each reaction's originating gene hit and DIAMOND alignment
- Reaction and metabolite identifiers follow ModelSEED biochemistry namespace conventions (e.g., media compounds end in _e)

## Limitations

- The draft is built solely from gene-associated reactions plus media-condition reactions, so it may need parsimonious flux-based gap-filling (Type 3, controlled by --min_frac 0.01 / --max_frac 0.5) to satisfy objective thresholds
- Gene hits depend entirely on KEGG annotation quality via DIAMOND alignment; genes without KEGG hits contribute no reactions
- Installation/test downloads the DIAMOND binary and database automatically unless --skip-diamond is used; full test suite takes ~1hr
- Upstream maintenance has moved to the csbl/reconstructor repository; emmamglass/reconstructor is no longer updated

## Evidence

- [intro] KEGG hits translated to ModelSEED reactions with GPRs: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [methods] Universal database provenance: "The universal reaction database used in `Reconstructor` is a modified version of the [ModelSEED biochemistry database](https://github.com/ModelSEED/ModelSEEDDatabase)"
- [methods] Universal curation filter steps: "We corrected poorly defined metabolite formulas and removed mass imbalanced reactions using the [curateuniversal.py](src/curateuniversal.py) script"
- [methods] Serialize universal database: "with open("path/to/universal.pickle", "wb") as f:
    pickle.dump(universal, f)"
- [readme] CLI usage from README: [evidence span withheld — anchor status: external_doc]
- [readme] Test suite downloads DIAMOND: [evidence span withheld — anchor status: not_found]
