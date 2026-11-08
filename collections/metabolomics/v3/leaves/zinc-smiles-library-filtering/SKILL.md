---
name: zinc-smiles-library-filtering
description: Use when preparing a pretraining corpus for a spectra-to-structure model
  whose synthetic-spectrum generator (e.g. RASSP, limited to 4096 fragment subformulae)
  imposes hard element-count and formula-complexity constraints on candidate molecules.
when_to_use_negative:
- The generator is not formula-limited (e.g. NEIMS-only pretraining, which achieved
  2% better Acc1 than RASSP-only but does not impose a subformula cap) — the RASSP-specific
  element/subformula filters would needlessly discard valid compounds.
- The downstream model is not finetuned on NIST 20 spectra, so the NIST-20 de-duplication
  step serves no leakage-prevention purpose.
- The input is already a spectra file (e.g. an .msp of experimental EI-MS peaks) rather
  than a raw SMILES library — structural filtering cannot be applied to spectra.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RASSP
  role: Provides the processing constraints (8 most common elements, ≤4096 possible
    fragment subformulae) that define which ZINC compounds survive filtering; RASSP
    later generates synthetic EI-MS spectra from the retained molecules.
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Hosts the data-preprocessing notebooks in the SpecTUS repository that implement
    the filtering of scraped ZINC SMILES into the synth1/synth2 corpora.
  repo: https://github.com/hejjack/SpecTUS
- name: conda
  role: Sets up the Python environment (trainSpectus_build.yaml) needed to run the
    SMILES parsing and filtering scripts.
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:1396b29c6b9e18d8559be03c73db695eba26a92f55a106cf39982120d7232cf1
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
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# zinc-smiles-library-filtering

## Summary

Reduces a large scraped ZINC20 SMILES library (~30M compounds) to the subset processable by RASSP for synthetic-spectrum generation — the step that produced the synth1 (≈4.7M) and synth2 (≈4.8M) pretraining corpora for SpecTUS.

## When to use

Use when preparing a pretraining corpus for a spectra-to-structure model whose synthetic-spectrum generator (e.g. RASSP, limited to 4096 fragment subformulae) imposes hard element-count and formula-complexity constraints on candidate molecules.

## When NOT to use

- The generator is not formula-limited (e.g. NEIMS-only pretraining, which achieved 2% better Acc1 than RASSP-only but does not impose a subformula cap) — the RASSP-specific element/subformula filters would needlessly discard valid compounds.
- The downstream model is not finetuned on NIST 20 spectra, so the NIST-20 de-duplication step serves no leakage-prevention purpose.
- The input is already a spectra file (e.g. an .msp of experimental EI-MS peaks) rather than a raw SMILES library — structural filtering cannot be applied to spectra.

## Inputs

- ~30-million-compound ZINC20 SMILES list per corpus (scraped with the 2D-standard-annotated-druglike query), one list each for synth1 and synth2
- RASSP model constraints: the 8 supported elements and the maximum of 4096 possible fragment subformulae
- NIST 20 compound list (for de-duplication against the finetuning set)

## Outputs

- rassp_filtered_compounds.csv — rows of SMILES tagged with source corpus (synth1 or synth2)
- Filtered corpora of ≈4.7M (synth1) and ≈4.8M (synth2) RASSP-processable compounds
- NIST-20-depleted pretraining compound set (no leakage into the finetuning spectra)

## How to apply

Start from the two scraped ZINC20 SMILES lists (~30 million compounds each, obtained with the 2D-standard-annotated-druglike query) that underlie the synth1 and synth2 corpora. Parse each SMILES string and retain only compounds composed exclusively of the 8 most common elements supported by RASSP. Next enforce RASSP's constraint on the number of possible fragment subformulae — the published model is limited to 4096 fragment formulae — discarding compounds outside this limit; this step alone collapses each set to roughly 4.7 million (synth1) and 4.8 million (synth2) compounds. Finally, remove any compound that also appears in the NIST 20 library to prevent data leakage, since SpecTUS is subsequently finetuned on NIST 20 experimental spectra. Tag each surviving row with its source corpus (synth1 or synth2) so the two halves can later be mixed 1:1 with NEIMS-generated spectra. Validation: the filtered sets should contain approximately 4.7 million compounds for synth1 and 4.8 million for synth2, matching the sizes reported in the paper.

## Related tools

- **RASSP** (Provides the processing constraints (8 most common elements, ≤4096 possible fragment subformulae) that define which ZINC compounds survive filtering; RASSP later generates synthetic EI-MS spectra from the retained molecules.) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Hosts the data-preprocessing notebooks in the SpecTUS repository that implement the filtering of scraped ZINC SMILES into the synth1/synth2 corpora.) — https://github.com/hejjack/SpecTUS
- **conda** (Sets up the Python environment (trainSpectus_build.yaml) needed to run the SMILES parsing and filtering scripts.) — https://github.com/hejjack/SpecTUS

## Examples

```
python notebooks/filter_zinc_for_rassp.py --input zinc_synth1_smiles.csv --output rassp_filtered_compounds.csv --tag synth1
```

## Evaluation signals

- Count of surviving compounds matches the reported values: ≈4.7 million for synth1 and ≈4.8 million for synth2 (down from ~30 million each).
- No SMILES in the output contains an element outside RASSP's 8-element set, and no output compound exceeds 4096 possible fragment subformulae.
- Intersection of the filtered sets with the NIST 20 compound list is empty (zero data leakage into the finetuning split).
- Every output row carries a corpus tag (synth1 or synth2) so the two halves can be recombined 1:1 with NEIMS-generated spectra during pretraining.
- Round-trip check: the filtered SMILES list, when fed through RASSP, produces synthetic spectra for essentially all retained molecules (no silent failures at spectrum-generation time).

## Limitations

- The filter is generator-specific: it keeps only the 8 most common elements and compounds within RASSP's 4096-fragment-subformula limit, so compounds valid for other generators are discarded.
- Roughly 80% of the scraped ZINC library is dropped by this filtering alone; any downstream claim of library coverage must account for this reduction.
- The published RASSP model's capacity (4096 fragment formulae) caps which molecules can ever be converted to synthetic spectra, so filtering decisions bind the pretraining set permanently.

## Evidence
