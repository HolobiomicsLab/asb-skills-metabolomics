---
name: fragment-subformula-limit-screening
description: Use when preparing large SMILES corpora for synthetic-spectrum generation
  with a model that has a hard limit on the number of possible fragment subformulae,
  and the corpus must be reduced to processable compounds.
when_to_use_negative:
- The downstream spectra generator has no constraint on the number of possible fragment
  subformulae, so the 4096-fragment-formulae ceiling does not apply.
- Input compounds contain elements outside the 8 most common elements supported by
  the downstream model — those compounds are discarded by design, not repaired.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: RASSP
  role: Source of the element whitelist (8 most common elements) and the 4096-fragment-formulae
    ceiling that define which compounds survive filtering
- name: Jupyter Notebook
  role: Used for data preprocessing and filtering steps in the SpecTUS workflow
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Environment setup for running the preprocessing notebooks (trainSpectus_build.yaml)
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:7ada257225740e4565d2cf53c647c016d38b1b16d59b043b51eee0bb016b6148
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - RASSP
  - conda
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fragment-subformula-limit-screening

## Summary

Screens large SMILES corpora down to compounds a spectra-prediction model (RASSP) can process, by restricting to the 8 most common elements and compounds whose possible fragment subformulae fit within the model's 4096-fragment-formula limit.

## When to use

Use when preparing large SMILES corpora for synthetic-spectrum generation with a model that has a hard limit on the number of possible fragment subformulae, and the corpus must be reduced to processable compounds.

## When NOT to use

- The downstream spectra generator has no constraint on the number of possible fragment subformulae, so the 4096-fragment-formulae ceiling does not apply.
- Input compounds contain elements outside the 8 most common elements supported by the downstream model — those compounds are discarded by design, not repaired.

## Inputs

- Large SMILES list scraped from ZINC20 (~30 million compounds per corpus, obtained with the 2D-standard-annotated-druglike query), one per target corpus (synth1, synth2)
- Target experimental spectral library compound list (NIST 20) for overlap/duplicate removal

## Outputs

- Filtered compound set per corpus (synth1 ~4.7 million, synth2 ~4.8 million compounds) suitable for downstream NEIMS/RASSP synthetic-spectrum generation
- Removed/leaked compound list (compounds matching NIST 20 or failing element/subformulae constraints)

## How to apply

Starting from a large scraped SMILES set (e.g. the ~30-million-compound ZINC20 '2D-standard-annotated-druglike' lists used for the synth1/synth2 pretraining corpora), parse each SMILES string and apply two structural constraints derived from the downstream spectra generator: retain only compounds composed exclusively of the 8 most common elements supported by RASSP, and discard compounds whose number of possible fragment subformulae exceeds the published RASSP model's limit of 4096 fragment formulae. Also remove any compound that appears in the target experimental spectral library (NIST 20 in this workflow) to prevent data leakage when the pretrained model is later finetuned on that library's spectra. The expected outcome is a reduced, processable compound set — approximately 4.7 million compounds for synth1 and 4.8 million for synth2 from the original 30-million sets. Decision point: if a different spectra generator is substituted, re-derive both the element whitelist and the subformulae ceiling from that generator's published model specifications rather than assuming RASSP's limits carry over.

## Related tools

- **RASSP** (Source of the element whitelist (8 most common elements) and the 4096-fragment-formulae ceiling that define which compounds survive filtering)
- **Jupyter Notebook** (Used for data preprocessing and filtering steps in the SpecTUS workflow) — https://github.com/hejjack/SpecTUS
- **conda** (Environment setup for running the preprocessing notebooks (trainSpectus_build.yaml)) — https://github.com/hejjack/SpecTUS

## Evaluation signals

- Filtered synth1 set size ≈ 4.7 million compounds; filtered synth2 set size ≈ 4.8 million compounds.
- No compound in the output set contains an element outside the 8-element whitelist supported by RASSP.
- No compound in the output set exceeds the 4096 possible-fragment-subformulae limit.
- No compound in the output set overlaps the NIST 20 library compounds used for downstream finetuning.
- Downstream synthetic-spectrum generation with RASSP proceeds without compound-rejection errors on the filtered set.

## Limitations

- The element whitelist and subformulae ceiling are model-specific (RASSP's published model); other spectra generators may impose different or no such constraints.
- This skill filters by feasibility of the spectra generator, not by chemical plausibility — compounds outside the whitelist are simply dropped, which is intentional in the source workflow.
- NIST 20 compound removal is required to prevent data leakage when SpecTUS is finetuned on NIST 20 spectra; omitting it invalidates the test set.

## Evidence

- [methods] SpecTUS finetuning on NIST 20 spectra: [evidence span withheld — anchor status: not_found]
