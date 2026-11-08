---
name: metabolic-model-memote-quality-assessment
description: Use when you have draft or gap-filled SBML metabolic models (e.g., COBRApy/Reconstructor
  output with ModelSEED-consistent naming) and need standardized quality benchmarking
  scores to judge reconstruction quality.
when_to_use_negative:
- The model is not an SBML metabolic reconstruction or does not use ModelSEED-consistent
  reaction/metabolite naming, making MEMOTE scoring not directly applicable
- You need functional/phenotypic validation (e.g., gene deletion or media growth simulations)
  rather than standardized structural quality scoring
- You only have raw BLASTp output or an ungap-filled draft and have not yet completed
  the reconstruction pipeline
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3174
tools:
- name: MEMOTE
  role: Benchmarking suite that scores each exported SBML model to produce its overall
    MEMOTE score
- name: Reconstructor
  role: Builds the COBRApy-compatible GENREs (via DIAMOND BLASTp against KEGG and
    the ModelSEED-based universal.pickle database) that are then scored with MEMOTE
  repo: https://github.com/csbl/reconstructor
- name: COBRApy
  role: Model framework in which Reconstructor reconstructions are built and exported
    as SBML for MEMOTE scoring
- name: DIAMOND
  role: Sequence aligner used during reconstruction to assign KEGG orthologs from
    annotated proteins before MEMOTE benchmarking
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:aa462bcda37c978dee2379e68dd829b6a3c61094e164ac30540915772a8bc1b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csbl/reconstructor
  tools:
  - COBRApy
  - DIAMOND
  - MEMOTE
  - Python
  - Reconstructor
  - pip
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# metabolic-model-memote-quality-assessment

## Summary

Benchmark genome-scale metabolic network reconstructions (GENREs) with the MEMOTE suite by scoring exported SBML models to obtain overall quality scores, as done for 10 Reconstructor-built bacterial models that scored 83%-85%.

## When to use

Use when you have draft or gap-filled SBML metabolic models (e.g., COBRApy/Reconstructor output with ModelSEED-consistent naming) and need standardized quality benchmarking scores to judge reconstruction quality.

## When NOT to use

- The model is not an SBML metabolic reconstruction or does not use ModelSEED-consistent reaction/metabolite naming, making MEMOTE scoring not directly applicable
- You need functional/phenotypic validation (e.g., gene deletion or media growth simulations) rather than standardized structural quality scoring
- You only have raw BLASTp output or an ungap-filled draft and have not yet completed the reconstruction pipeline

## Inputs

- Completed, gap-filled GENREs exported as SBML files with ModelSEED-consistent naming
- Annotated amino-acid .fasta files (Type 1 input) per organism for the reconstruction step
- Gram classification (positive or negative) and media condition (e.g., 'rich') for each reconstruction

## Outputs

- Overall MEMOTE score per model (expected 83%-85% for the 10 bacterial GENREs)
- Comparison table of per-species overall MEMOTE scores (Fig. 1D-equivalent)

## How to apply

First complete the reconstruction: install Reconstructor with pip (Python >= 3.9), run 'python -m reconstructor --test yes' to download the universal.pickle reaction database (a modified ModelSEED biochemistry database) and a DIAMOND binary, then reconstruct each organism from its annotated amino-acid .fasta (Type 1 input) with DIAMOND BLASTp against KEGG references and apply parsimony-based gap-filling. Export each completed GENRE as an SBML file with ModelSEED-consistent reaction and metabolite naming. Run the MEMOTE suite on each exported SBML model to obtain its overall MEMOTE score. Compile per-species scores into a comparison table (analogous to Fig. 1D); for 10 diverse bacterial strains, expect overall scores in the reported 83%-85% range, which indicates high quality. Pre-computed benchmarking scores for 10 representative Reconstructor models are also available in the repository's reference/MEMOTE.zip for direct comparison.

## Related tools

- **MEMOTE** (Benchmarking suite that scores each exported SBML model to produce its overall MEMOTE score)
- **Reconstructor** (Builds the COBRApy-compatible GENREs (via DIAMOND BLASTp against KEGG and the ModelSEED-based universal.pickle database) that are then scored with MEMOTE) — https://github.com/csbl/reconstructor
- **COBRApy** (Model framework in which Reconstructor reconstructions are built and exported as SBML for MEMOTE scoring)
- **DIAMOND** (Sequence aligner used during reconstruction to assign KEGG orthologs from annotated proteins before MEMOTE benchmarking) — https://github.com/bbuchfink/diamond

## Examples

```
python -m reconstructor --input_file Osplanchnicus.aa.fasta --file_type 1 --gram negative --media rich
```

## Evaluation signals

- Each exported SBML model produces a valid overall MEMOTE score when run through the MEMOTE suite
- Overall MEMOTE scores for well-built bacterial GENREs fall in the reported 83%-85% range
- Per-species scores are consistent with the pre-computed benchmarking scores in the repository's reference/MEMOTE.zip
- Exported models use reaction and metabolite naming conventions consistent with the ModelSEED biochemistry database
- Scores are comparable across both well-studied (e.g., Clostridium difficile) and lesser-known (e.g., Tropheryma whipplei) taxa

## Limitations

- Overall MEMOTE scores near 83%-85% reflect structural quality only; models may still require additional gap-filling (Reconstructor Type 3 input) if functional tests fail
- Score quality depends on the curated universal.pickle universal reaction database; the original emmamglass/reconstructor repository is no longer updated (maintenance moved to csbl/reconstructor)
- Installation test suite takes ~1 hour depending on the system, and DIAMOND binary download may need to be skipped with --skip-diamond

## Evidence

- [results] Overall MEMOTE scores 83%-85% across 10 bacterial GENREs: [evidence span withheld — anchor status: not_found]
- [abstract] Reconstructor produces ModelSEED-consistent high-quality drafts with parsimony gap-filling: "produces high-quality draft reconstructions with reaction and metabolite naming conventions that are consistent with the ModelSEED biochemistry database"
- [readme] Run installation test to download dependencies: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmarking scores available in repository: [evidence span withheld — anchor status: external_doc]
- [abstract] Reconstructor generates high-quality GENREs capturing taxonomic metabolic differences: "We demonstrate how Reconstructor readily generates high-quality GENRES that capture strain, species, and higher taxonomic differences in functional metabolism of bacteria"
