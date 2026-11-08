---
name: insilico-spectral-library-construction
description: Use when you have a set of predicted or known metabolite structures plus
  a corpus of experimental MS/MS spectra and need to measure annotation gains from
  expanding the reference library.
when_to_use_negative:
- When the goal is to discover and structurally confirm new metabolites (synthesis
  and targeted MS/MS acquisition), not to annotate existing spectra against a fixed
  reference library.
- When experimental spectra must be annotated with retention time or collision energy
  metadata beyond MS/MS peak matching — predicting MS/MS spectra or retention times
  from structures remains an open problem per the article.
- When the organism is evolutionarily distant from the model's training set (e.g.
  plant or bacterial metabolomes) and a bespoke DeepMet model has not been trained,
  since the reference model was trained on human metabolites.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DeepMet
  role: Generates the candidate metabolite structures (2.4 million anticipated metabolites)
    that expand the in silico MS/MS library beyond known entries.
  repo: https://github.com/skinniderlab/clm
- name: Python 3.10
  role: Runtime environment for implementing the spectral matching, library construction,
    and fold-gain computation.
  repo: https://github.com/skinniderlab/CLM
- name: Snakemake
  role: Orchestrates the end-to-end workflow including preprocessing, model training,
    and downstream structure generation used to build the library.
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Converts candidate SMILES to molecule objects and removes light fragments,
    neutralizes charges, and filters invalid elements before library construction.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:142fa431f7712e36fbc31b16c5f54606b71a6d774aeea0241a14f005539686b6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/clm
  tools:
  - DeepMet
  - Python
  - Python 3.10
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# insilico-spectral-library-construction

## Summary

Constructs an in silico MS/MS spectral library from candidate metabolite structures and searches experimental spectra against it via cosine similarity, quantifying how many spectra can be assigned a chemical structure.

## When to use

Use when you have a set of predicted or known metabolite structures plus a corpus of experimental MS/MS spectra and need to measure annotation gains from expanding the reference library.

## When NOT to use

- When the goal is to discover and structurally confirm new metabolites (synthesis and targeted MS/MS acquisition), not to annotate existing spectra against a fixed reference library.
- When experimental spectra must be annotated with retention time or collision energy metadata beyond MS/MS peak matching — predicting MS/MS spectra or retention times from structures remains an open problem per the article.
- When the organism is evolutionarily distant from the model's training set (e.g. plant or bacterial metabolomes) and a bespoke DeepMet model has not been trained, since the reference model was trained on human metabolites.

## Inputs

- Corpus of experimental MS/MS spectra (e.g. 29.1 million spectra from 4,510 MetaboLights analyses in .mzML/.mzMLb or equivalent peak-list form)
- Reference structures of known metabolites (e.g. HMDB SMILES)
- Predicted candidate structures with sampling frequencies (e.g. 2.4 million DeepMet-generated metabolites)
- Optional experimental m/z values used for mass-to-charge-based prioritization of the candidate pool

## Outputs

- Two in silico MS/MS spectral libraries (known-only and combined known+predicted)
- Per-spectrum match lists with cosine similarity scores for each library search
- Match counts at the cosine similarity >= 0.8 threshold and the fold-gain statistic
- msms_annotation_comparison.csv containing match counts and fold-gain comparison

## How to apply

Build two in silico MS/MS libraries from the candidate structures: one restricted to known metabolites (e.g. HMDB structures) and one combining known metabolites with language-model-predicted structures (e.g. 2.4 million DeepMet metabolites). Search the experimental spectra corpus — here 29.1 million MS/MS spectra compiled from 4,510 human blood analyses deposited on MetaboLights — against each library using cosine similarity spectral matching, scoring every spectrum independently per library. Count spectra matched at cosine similarity >= 0.8 for each library and compute the fold-change in matched spectra between the combined and known-only searches; in the article's benchmark the combined library approximately tripled the number of matched experimental spectra. The candidate structure pool can be restricted a priori to the experimental m/z (a mass-to-charge-based prioritization step) to focus the search on plausible annotations, and predicted structures should be de-duplicated against known entries to avoid double counting. Match counts are written to a comparison table (msms_annotation_comparison.csv) for downstream reporting.

## Related tools

- **DeepMet** (Generates the candidate metabolite structures (2.4 million anticipated metabolites) that expand the in silico MS/MS library beyond known entries.) — https://github.com/skinniderlab/clm
- **Python 3.10** (Runtime environment for implementing the spectral matching, library construction, and fold-gain computation.) — https://github.com/skinniderlab/CLM
- **Snakemake** (Orchestrates the end-to-end workflow including preprocessing, model training, and downstream structure generation used to build the library.) — https://github.com/skinniderlab/CLM
- **RDKit** (Converts candidate SMILES to molecule objects and removes light fragments, neutralizes charges, and filters invalid elements before library construction.)

## Examples

```
python -c "import pandas as pd; exp = pd.read_csv('spectra.csv'); lib = pd.read_csv('combined_library.csv'); from scipy.spatial.distance import cosine; matches = exp.apply(lambda r: 1 - cosine(r.mz_peaks, lib[lib.mz.isclose(r.mz, atol=0.01)].mz_peaks.mean(axis=0)) if not lib.empty else 0, axis=1); matches.to_csv('msms_annotation_comparison.csv')"
```

## Evaluation signals

- Match counts at cosine similarity >= 0.8 should be reproducible across identical spectral searches; the combined-library count should exceed the known-only count by a fold-gain (approximately 3x in the article's human blood benchmark).
- Every experimental spectrum must be scored against both libraries independently to isolate the effect of adding predicted structures — differing spectrum sets invalidates the comparison.
- De-duplication check: predicted structures that coincide with known HMDB entries should be removed from the combined library so the fold-gain reflects genuinely novel candidates.
- Structure validity: all candidate SMILES entering the library must survive RDKit canonicalization, charge neutralization, and element filtering without errors.
- Output schema: msms_annotation_comparison.csv must contain at minimum the per-library matched-spectrum counts and the computed fold-change value.

## Limitations

- Predicting MS/MS spectra or retention times from chemical structures remains an open problem, so the in silico libraries depend on approximate spectral prediction quality.
- DeepMet was trained and evaluated only on human metabolite structures; applying the same library to plant or bacterial metabolomes will likely require a bespoke model.
- The language model has inherent limitations in anticipating metabolites that originate from divergent, undiscovered biosynthetic routes.
- Meta-learning-based filtering improves annotation accuracy and calibration but adds a model-integration step beyond simple cosine matching.

## Evidence

- [readme] CLM repository README describes the project scope and links to full documentation: [evidence span withheld — anchor status: external_doc]
