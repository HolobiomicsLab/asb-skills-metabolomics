---
name: genome-scale-metabolic-network-reconstruction
description: Use when annotated protein FASTA, BLASTp/DIAMOND output, or a draft SBML
  model is available for a bacterial genome and a COBRApy-compatible genome-scale
  metabolic model (SBML GENRE) is needed for FBA, gene essentiality, or flux sampling
  analyses.
when_to_use_negative:
- Input is raw (unannotated) nucleotide or protein sequence — the tool requires annotated
  amino acid .fasta, BLASTp hits, or an existing SBML GENRE, not raw assemblies.
- You need a model with non-ModelSEED reaction/metabolite naming conventions or a
  hand-curated reconstruction — Reconstructor exports ModelSEED-consistent SBML and
  is automated, not manual-curation software.
- The organism is non-bacterial with unknown Gram status and the Gram-classification
  argument cannot be justified.
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0780
tools:
- name: Reconstructor
  role: 'Primary tool: builds the draft GENRE from KEGG annotations and performs parsimony-based
    (pFBA) gap-filling against universal.pickle'
  repo: https://github.com/emmamglass/reconstructor
- name: COBRApy
  role: Loads the resulting SBML model for FBA, gap-filling configuration, and downstream
    analyses (gene deletion, flux sampling)
- name: DIAMOND
  role: Sequence aligner performing DIAMOND BLASTp of annotated proteins against KEGG
    references to derive ortholog assignments
  repo: https://github.com/bbuchfink/diamond
- name: ModelSEEDDatabase
  role: Source of the modified ModelSEED biochemistry database used as the universal
    reaction database for gap-filling
  repo: https://github.com/ModelSEED/ModelSEEDDatabase
- name: Python
  role: Runtime environment (>= 3.9) for the Reconstructor package and COBRApy integration
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:961d7dbb44cb3b599b0c10ade16c76e16bfff3ce271eb6795d96614dd9b10904
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
  - pip
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# genome-scale-metabolic-network-reconstruction

## Summary

Automated reconstruction of genome-scale metabolic network reconstructions (GENREs) from annotated amino acid .fasta files using the Reconstructor Python package, which applies DIAMOND-based KEGG annotation matching against a ModelSEED-derived universal.pickle reaction database and completes drafts with parsimony-based (pFBA) gap-filling.

## When to use

Use when annotated protein FASTA, BLASTp/DIAMOND output, or a draft SBML model is available for a bacterial genome and a COBRApy-compatible genome-scale metabolic model (SBML GENRE) is needed for FBA, gene essentiality, or flux sampling analyses.

## When NOT to use

- Input is raw (unannotated) nucleotide or protein sequence — the tool requires annotated amino acid .fasta, BLASTp hits, or an existing SBML GENRE, not raw assemblies.
- You need a model with non-ModelSEED reaction/metabolite naming conventions or a hand-curated reconstruction — Reconstructor exports ModelSEED-consistent SBML and is automated, not manual-curation software.
- The organism is non-bacterial with unknown Gram status and the Gram-classification argument cannot be justified.

## Inputs

- annotated amino acid .fasta file (Type 1 input)
- DIAMOND/BLASTp hits output file (Type 2 input)
- existing SBML GENRE for additional gap-filling (Type 3 input)
- Gram classification (positive or negative)
- media condition (default 'rich' or comma-separated ModelSEED cpd IDs ending in _e)
- universal.pickle modified ModelSEED reaction database

## Outputs

- gap-filled genome-scale metabolic network reconstruction in SBML format (.sbml/.xml)
- COBRApy-compatible Model object
- gap-filling statistics (count of reactions added and biomass objective flux)

## How to apply

Install Reconstructor via pip (Python >= 3.9) and run 'python -m reconstructor --test yes' to download the universal.pickle ModelSEED-derived reaction database and a DIAMOND binary. For Type 1 input, run Reconstructor on the annotated amino acid .fasta so DIAMOND BLASTp aligns proteins to KEGG references and a draft COBRApy-compatible GENRE is built; specify the Gram classification (--gram negative/positive), media condition (default 'rich'), and optionally --cpu for parallelized blasting. Apply Reconstructor's parsimony-based gap-filling: it runs pFBA to select the optimal reaction set from the universal database by minimizing flux through both gene-associated and non-gene-associated reactions (not minimizing reaction count) while restoring biomass production under the user-defined media, with objective fractions controlled by --min_frac (default 0.01) and --max_frac (default 0.5). Export the gap-filled model as SBML with ModelSEED-consistent reaction and metabolite naming; the resulting model loads directly into COBRApy for downstream FBA, single-gene deletion, or flux sampling analyses.

## Related tools

- **Reconstructor** (Primary tool: builds the draft GENRE from KEGG annotations and performs parsimony-based (pFBA) gap-filling against universal.pickle) — https://github.com/emmamglass/reconstructor
- **COBRApy** (Loads the resulting SBML model for FBA, gap-filling configuration, and downstream analyses (gene deletion, flux sampling))
- **DIAMOND** (Sequence aligner performing DIAMOND BLASTp of annotated proteins against KEGG references to derive ortholog assignments) — https://github.com/bbuchfink/diamond
- **ModelSEEDDatabase** (Source of the modified ModelSEED biochemistry database used as the universal reaction database for gap-filling) — https://github.com/ModelSEED/ModelSEEDDatabase
- **Python** (Runtime environment (>= 3.9) for the Reconstructor package and COBRApy integration)

## Examples

```
python -m reconstructor --input_file Osplanchnicus.aa.fasta --file_type 1 --gram negative --media rich
```

## Evaluation signals

- The exported SBML model loads in COBRApy and carries positive flux to the biomass objective under the specified media conditions.
- Reaction and metabolite identifiers in the SBML use ModelSEED-consistent naming (e.g., cpd/RXN namespace).
- Gap-fill statistics report a consistent count of reactions added and match the model contents in the exported SBML.
- MEMOTE benchmarking of exported models falls in the reported 83%–85% overall-score range.
- Reconstructions of distinct bacterial strains yield distinguishable steady-state flux distributions (strain/species/class clustering in NMDS of FBA samples).

## Limitations

- The universal database required curation — poorly defined metabolite formulas were corrected and mass-imbalanced reactions removed via curateuniversal.py — so raw database versions may introduce balance issues.
- Installation test suite takes ~1 hour and downloads a DIAMOND database and binary; use --skip-diamond to avoid the binary download.
- The emmamglass/reconstructor repository is no longer updated; maintenance has moved to csbl/reconstructor.
- Documentation is still under development.

## Evidence

- [abstract] Reconstructor generates SBML GENREs from three input types: "Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE that can be further gap-filled"
- [abstract] gap-filling based on parsimony: "produces high-quality draft reconstructions with reaction and metabolite naming conventions that are consistent with the ModelSEED biochemistry database and includes a gap-filling technique based on"
- [methods] universal database is modified ModelSEED: "The universal reaction database used in `Reconstructor` is a modified version of the [ModelSEED biochemistry database](https://github.com/ModelSEED/ModelSEEDDatabase)"
- [readme] test suite installs dependencies: [evidence span withheld — anchor status: not_found]
- [methods] curated universal db: "We corrected poorly defined metabolite formulas and removed mass imbalanced reactions using the [curateuniversal.py](src/curateuniversal.py) script"
