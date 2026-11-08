---
name: adduct-constrained-msms-annotation
description: Use when evaluating molecular formula annotation accuracy on LC-MS/MS
  data with a known ground-truth adduct per compound, or when quantifying how adduct
  misassignment contributes to SIRIUS/ZODIAC errors.
when_to_use_negative:
- Adducts are unknown and cannot be constrained — the article notes adduct information
  cannot be reliably deduced from isotope pattern and MS/MS data and must come from
  high-quality preprocessing
- You lack a validated ground truth; the paper's evaluation relies on ground truth
  established by spectral library search and manual validation
- The goal is de novo formula ranking performance without adduct priors (the standard
  ZODIAC benchmark rather than the known-adducts ablation)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS
  role: Computes molecular formula candidate lists (top 50 per compound) and fragmentation
    trees; ZODIAC is implemented inside the SIRIUS command-line tool
  repo: https://github.com/sirius-ms/sirius
- name: ZODIAC
  role: Reranks the SIRIUS molecular formula candidates using Bayesian statistics
    via Gibbs sampling under the known-adduct constraint
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS
  role: Modified OpenMS 2.4.0 used to preprocess input mzML/mzXML files and discard
    low-quality MS/MS spectra
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:db766d2dae4a37197755c983965d1a672e482cd37fae397705ac60cf69fe1d11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - OpenMS
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# adduct-constrained-msms-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces ZODIAC's known-adducts ablation on the dendroides dataset: each compound's ionization adduct is constrained to the ground-truth adduct before SIRIUS/ZODIAC formula annotation, quantifying how strongly wrong adduct assumptions drive errors (SIRIUS 67.17% vs ZODIAC 99.50% correct on dendroides).

## When to use

Use when evaluating molecular formula annotation accuracy on LC-MS/MS data with a known ground-truth adduct per compound, or when quantifying how adduct misassignment contributes to SIRIUS/ZODIAC errors.

## When NOT to use

- Adducts are unknown and cannot be constrained — the article notes adduct information cannot be reliably deduced from isotope pattern and MS/MS data and must come from high-quality preprocessing
- You lack a validated ground truth; the paper's evaluation relies on ground truth established by spectral library search and manual validation
- The goal is de novo formula ranking performance without adduct priors (the standard ZODIAC benchmark rather than the known-adducts ablation)

## Inputs

- dendroides SIRIUS candidate lists (top 50 molecular formula candidates per compound)
- groundtruth_dendroides.csv mapping each compound to its ground-truth ionization adduct
- dendroides LC-MS/MS data (MassIVE accession MSV000080502)

## Outputs

- dendroides_known_adducts_accuracy.csv with SIRIUS and ZODIAC annotation accuracies under the known-adducts condition
- top-ranked molecular formula per compound from SIRIUS and from ZODIAC output

## How to apply

Obtain the dendroides SIRIUS candidate lists (top 50 molecular formula candidates per compound) and groundtruth_dendroides.csv from the data deposit at https://bio.informatik.uni-jena.de/data/ or the MassIVE accession MSV000080502. For each compound, read its ground-truth ionization adduct from groundtruth_dendroides.csv and set the adduct constraint accordingly — adducts cannot be reliably deduced from isotope pattern and MS/MS data, so they must be supplied from preprocessing/ground truth. Run ZODIAC's Gibbs sampling-based reranking via the SIRIUS command-line tool on the candidate lists under the known-adducts condition, then extract the top-ranked formula per compound from both SIRIUS and ZODIAC output. Compare predicted formulas against the ground truth established by spectral library search and manual validation for the 135 dendroides compounds, and compute annotation accuracies for both tools, writing dendroides_known_adducts_accuracy.csv.

## Related tools

- **SIRIUS** (Computes molecular formula candidate lists (top 50 per compound) and fragmentation trees; ZODIAC is implemented inside the SIRIUS command-line tool) — https://github.com/sirius-ms/sirius
- **ZODIAC** (Reranks the SIRIUS molecular formula candidates using Bayesian statistics via Gibbs sampling under the known-adduct constraint) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS** (Modified OpenMS 2.4.0 used to preprocess input mzML/mzXML files and discard low-quality MS/MS spectra) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
sirius --input dendroides.mzml --candidates top50 --adduct $(cat groundtruth_dendroides.csv) zodiac  # Gibbs reranking under known adducts via the SIRIUS CLI
```

## Evaluation signals

- Under the known-adducts condition, ZODIAC accuracy on dendroides should be 99.50% (200 compounds) and SIRIUS 67.17% (135 compounds), a 66-fold error rate decrease
- Output CSV dendroides_known_adducts_accuracy.csv must contain accuracy values for both SIRIUS and ZODIAC over the 135 dendroides compounds
- Every compound's SIRIUS run must use the adduct read from groundtruth_dendroides.csv, not an adduct inferred from isotope pattern or MS/MS data
- ZODIAC input candidate lists must contain the top 50 SIRIUS molecular formula candidates per compound

## Limitations

- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data, so results depend entirely on high-quality adduct annotations established during preprocessing
- The ablation is specific to the 135-compound dendroides ground truth and does not transfer to datasets without validated adducts
- Reproducing the full workflow requires a modified version of OpenMS 2.4.0 plus a separate patch file, not stock OpenMS

## Evidence

- [results] known-adduct accuracy finding: "We evaluated SIRIUS and ZODIAC against a ground truth which was established by spectral library search and manual validation"
- [discussion] adducts must come from preprocessing: [evidence span withheld — anchor status: not_found]
- [results] candidate count and reranking method: "ZODIAC builds upon SIRIUS and uses, say, the top 50 molecular formula annotations from SIRIUS as candidates for one compound"
- [discussion] data deposit and virtual machine availability: "SIRIUS and ZODIAC results and a virtual machine to reproduce the data are available from https://bio.informatik.uni-jena.de/data/."
- [discussion] OpenMS preprocessing requirement: [evidence span withheld — anchor status: not_found]
