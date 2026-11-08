---
name: spectral-cosine-similarity-scoring
description: Use when benchmarking in silico MS/MS spectral predictions (e.g., from
  FIORA, CFM-ID v4, or ICEBERG) against reference spectra in mgf/msp libraries such
  as NIST'17/MS-DIAL, MSnLib, or CASMI 2016/2022 challenge sets.
when_to_use_negative:
- Input is a raw LC-MS/MS file needing peak extraction rather than already-predicted
  vs. reference spectrum pairs
- Comparing spectra without collision-energy annotation or with unclear formats —
  the paper explicitly excludes MS/MS spectra with missing collision energies
- Evaluating CFM-ID on datasets where METLIN17-trained compounds overlap the test
  set, since train/test separation is not guaranteed for CFM-ID
edam_operation: http://edamontology.org/operation_3212
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Source of the spectral_sqrt_cosine default metric and the fiora-eval CLI used
    to compute per-compound similarity scores
  repo: https://github.com/BAMeScience/fiora
- name: CFM-ID
  role: Baseline predictor whose 10/20/40 eV predictions are scored at the closest
    matching collision energy
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ICEBERG
  role: Baseline predictor (retrained from ms-pred with test compounds excluded) whose
    spectra are scored against the same references
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:c480e3c68700976be138c55155b9b731f58e1ed5e896e185d8f5b20c2b950ad9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  - fiora-predict
  edam_operation: http://edamontology.org/operation_3212
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-cosine-similarity-scoring

## Summary

Score predicted MS/MS spectra against reference spectra using FIORA's square-root-transformed cosine similarity (spectral_sqrt_cosine), including the precursor peak, to benchmark tools such as FIORA, CFM-ID, and ICEBERG on test splits and CASMI challenges.

## When to use

Use when benchmarking in silico MS/MS spectral predictions (e.g., from FIORA, CFM-ID v4, or ICEBERG) against reference spectra in mgf/msp libraries such as NIST'17/MS-DIAL, MSnLib, or CASMI 2016/2022 challenge sets.

## When NOT to use

- Input is a raw LC-MS/MS file needing peak extraction rather than already-predicted vs. reference spectrum pairs
- Comparing spectra without collision-energy annotation or with unclear formats — the paper explicitly excludes MS/MS spectra with missing collision energies
- Evaluating CFM-ID on datasets where METLIN17-trained compounds overlap the test set, since train/test separation is not guaranteed for CFM-ID

## Inputs

- Predicted spectra in mgf or msp format (e.g., from fiora-predict, CFM-ID v4 Docker, or retrained ICEBERG from ms-pred)
- Reference spectra for NIST'17+MS-DIAL test split, MSnLib test split, or CASMI 2016/2022 challenge sets
- Query CSVs with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type'
- A trained FIORA checkpoint (e.g., checkpoints/fiora.pt or the default packaged model under fiora/resources/models)

## Outputs

- Per-compound spectral_sqrt_cosine similarity scores (e.g., test_eval.csv, validation_eval.csv via fiora-eval --output-dir)
- Per-spectrum annotated peak-intensity coverage values used to validate the sqrt(coverage) >= cosine bound
- Aggregated median cosine similarity per model, test set, and ionization mode (e.g., table1_spectral_similarity.csv)

## How to apply

For each compound query (columns Name, SMILES, Precursor_type, CE, Instrument_type), align the predicted spectrum to its reference and compute the square-root cosine similarity on sqrt-transformed intensities, including the precursor peak (FIORA's spectral_sqrt_cosine metric, the default in fiora-eval). Apply the paper's quality filters before trusting scores: at least two peak matches, minimum peak intensity coverage of 50%, and maximum precursor intensity of 90%; spectra with missing collision energies or unclear formats are excluded. For CFM-ID v4 predictions, select the prediction at the closest matching collision energy among 10, 20, and 40 eV; for CASMI 16, merge FIORA predictions across three stepped collision energies into a single 'stepped' spectrum before scoring. A key invariant is that a per-spectrum cosine value can never exceed the square root of its annotated intensity coverage, so validate coverage alongside similarity. Finally, aggregate median similarity per model, test set, and ionization mode ([M+H]+ / [M-H]-) into a summary table mirroring the paper's Table 1.

## Related tools

- **FIORA** (Source of the spectral_sqrt_cosine default metric and the fiora-eval CLI used to compute per-compound similarity scores) — https://github.com/BAMeScience/fiora
- **CFM-ID** (Baseline predictor whose 10/20/40 eV predictions are scored at the closest matching collision energy) — https://hub.docker.com/r/wishartlab/cfmid
- **ICEBERG** (Baseline predictor (retrained from ms-pred with test compounds excluded) whose spectra are scored against the same references) — https://github.com/samgoldman97/ms-pred

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf && fiora-eval --model checkpoints/fiora.pt -i test_queries.csv --output-dir eval/
```

## Evaluation signals

- Every per-spectrum cosine similarity satisfies the invariant cosine <= sqrt(annotated intensity coverage)
- Filtered spectra meet the paper's thresholds: >= 2 peak matches, >= 50% intensity coverage, <= 90% precursor intensity
- Median similarity for FIORA on the NIST/MS-DIAL test split reproduces Table 1 and exceeds CFM-ID/ICEBERG by 10%–49%
- Per-bin checks reproduce reported medians, e.g., FIORA median cosine of 0.63 in the Tanimoto 0.2–0.3 bin and 0.63 for organohalogen superclasses
- Eval CSVs (e.g., test_eval.csv) are written by fiora-eval --output-dir and contain per-compound spectral_sqrt_cosine scores

## Limitations

- The similarity metric is bounded by annotated intensity coverage: CASMI 22 predictions score low partly because median coverage is only ~32% (vs ~80% for CASMI 16), reflecting single-step fragmentation limits
- Not all collision energies for all CASMI 22 challenge spectra were found during extraction with the 10 ppm / 5 s RT window
- CFM-ID was trained on METLIN17, so train/test compound separation is not guaranteed for CFM-ID comparisons

## Evidence

- [discussion] We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%.: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] MS/MS spectra with missing collision energies or unclear formats were excluded.: "MS/MS spectra with missing collision energies or unclear formats were excluded."
- [readme] An input csv file must be provided and an output file specified (`mgf` or `msp` format).: "An input csv file must be provided and an output file specified (`mgf` or `msp` format)."
- [discussion] In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them: [evidence span withheld — anchor status: not_found]
