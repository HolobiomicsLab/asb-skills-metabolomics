---
name: molecular-mass-computation
description: Use when performing library matching against an augmented NEIMS spectral
  library and you know the molecular mass of the query molecule with reasonable accuracy,
  so candidates can be restricted to spectra within a few Daltons of the query mass.
when_to_use_negative:
- Molecular mass of the sample molecule is unknown or inaccurate — the article states
  the filter applies only 'for experimental setups where it is possible to know the
  molecular mass of the sample with some accuracy'.
- Molecule candidates span isotopic species that shift nominal mass by more than the
  tolerance, since NEIMS currently does not model isotope intensity peaks in ion fragments.
- The query is already restricted to a library whose spectra are all mass-compatible,
  making filtering a no-op.
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_0091
tools:
- name: NEIMS
  role: Produces the predicted spectra in the augmented library that the mass filter
    then restricts to candidates within the 5 Da tolerance of the query mass
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates the molecular structure representations (fingerprints) from which
    molecular mass is derived for each query and library candidate
- name: Tensorflow
  role: Framework in which the NEIMS model is built and trained; its inference output
    feeds the augmented library that mass filtering operates on
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:43e1ab967b8ac84e5a0152c2f3c133d2cc2a951e472c50c1e99aa040bcd289b3
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# molecular-mass-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the molecular mass of each query molecule from its structure so that the spectral reference library can be filtered to candidates within a 5 Da tolerance before library matching. This mass filter is the NEIMS library-matching step that reduces candidate libraries to a median of 6696 spectra per query and improves recall-at-10.

## When to use

Use when performing library matching against an augmented NEIMS spectral library and you know the molecular mass of the query molecule with reasonable accuracy, so candidates can be restricted to spectra within a few Daltons of the query mass.

## When NOT to use

- Molecular mass of the sample molecule is unknown or inaccurate — the article states the filter applies only 'for experimental setups where it is possible to know the molecular mass of the sample with some accuracy'.
- Molecule candidates span isotopic species that shift nominal mass by more than the tolerance, since NEIMS currently does not model isotope intensity peaks in ion fragments.
- The query is already restricted to a library whose spectra are all mass-compatible, making filtering a no-op.

## Inputs

- Query spectra from the NIST 2017 replicates library (noisier spectra for molecules in the NIST main library)
- Augmented reference library combining NIST main library observed spectra with NEIMS-predicted spectra
- Molecular structures (SDF/SMILES/InChIKey) for query molecules to compute molecular mass
- Mass filter tolerance (e.g. 5 Da)

## Outputs

- Per-query filtered candidate library (median 6696 spectra per query molecule)
- Ranks of the correct spectrum per query in the mass-filtered library
- recall-at-10 statistics over the query set
- Per-query ranks and recall statistics saved to mass_filter_results.csv

## How to apply

First compute the molecular mass of each query molecule from its molecular structure (e.g. via RDKit from the SDF/SMILES representation). Then, during the library-match search, apply the NEIMS mass filtering option: restrict the augmented reference library (observed NIST spectra plus model-predicted spectra) to only those candidate molecules whose molecular mass differs by no more than a few Daltons from the query's mass. In the article's configuration, the tolerance is set to 5 Da of the query molecule's mass, which reduces the library to a median of 6696 spectra per query molecule. The rationale is that for experimental setups where the molecular mass of the sample can be determined with some accuracy, filtering removes mass-incompatible candidates before the expensive weighted-cosine similarity computation, improving recall-at-10 on the augmented library. After filtering, compute weighted cosine similarity between each query spectrum and the remaining candidates and record the rank of the correct spectrum for recall-at-10 evaluation.

## Related tools

- **NEIMS** (Produces the predicted spectra in the augmented library that the mass filter then restricts to candidates within the 5 Da tolerance of the query mass) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates the molecular structure representations (fingerprints) from which molecular mass is derived for each query and library candidate)
- **Tensorflow** (Framework in which the NEIMS model is built and trained; its inference output feeds the augmented library that mass filtering operates on)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- The filtered library size per query should be about a median of 6696 spectra when a 5 Da tolerance is used on the NIST 2017 augmented library.
- Recall-at-10 on the mass-filtered augmented library should be ≥ the unfiltered value (the article reports the unfiltered model at 91.8% recall-at-10).
- Check that every retained candidate's molecular mass is within the 5 Da tolerance of the query molecule's mass — the filter should never admit mass-incompatible candidates.
- Per-query ranks written to mass_filter_results.csv should match the ranks computed with the unfiltered pipeline restricted to the mass window (sanity diff).
- The filter should fail or warn if a query molecule lacks a computed molecular mass (e.g. malformed SMILES/SDF).

## Limitations

- NEIMS currently does not have a method to model intensity peaks corresponding to isotopes in ion fragments, which can affect mass-based candidate matching.
- The mass filter's benefit assumes the sample's molecular mass can be known with some accuracy; if mass estimation is noisy, filtering may exclude the true match.
- The 5 Da tolerance and resulting median library size (6696) were tuned on the NIST 2017 setup and may not transfer to other libraries.

## Evidence
