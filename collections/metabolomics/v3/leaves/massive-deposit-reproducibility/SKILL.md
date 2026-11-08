---
name: massive-deposit-reproducibility
description: Use when reproducing published SIRIUS/ZODIAC molecular formula annotation
  benchmarks (e.g. dendroides, NIST1950) requires fetching the original deposits and
  ground-truth files instead of regenerating them.
when_to_use_negative:
- The analysis goal is de novo processing of newly acquired metabolomics data, not
  reproducing the ZODIAC paper's published benchmark numbers
- Preprocessed spectra and candidate lists are already available locally, so deposit
  retrieval would only duplicate existing files
- The dataset has no published MassIVE accession or author-provided ground truth to
  validate against
tools:
- name: SIRIUS
  role: Command-line host of the ZODIAC Gibbs sampling reranker; the retrieved candidate
    lists are executed against it for the known-adducts ablation
  repo: https://github.com/sirius-ms/sirius
- name: OpenMS
  role: Modified OpenMS 2.4.0 branch plus patch file used to reproduce the preprocessing
    step when the deposit's processed spectra are not used as-is
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:ef54cb34da82499929e3899508b5b04aa8c2a748325d0920300ffb7efb050d8e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# massive-deposit-reproducibility

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves the original data deposits (raw mzML/mzXML, SIRIUS candidate lists, ground-truth adduct annotations) and the authors' virtual machine from MassIVE and bio.informatik.uni-jena.de/data so that the ZODIAC paper's benchmark accuracies can be independently reproduced rather than re-estimated from scratch.

## When to use

Use when reproducing published SIRIUS/ZODIAC molecular formula annotation benchmarks (e.g. dendroides, NIST1950) requires fetching the original deposits and ground-truth files instead of regenerating them.

## When NOT to use

- The analysis goal is de novo processing of newly acquired metabolomics data, not reproducing the ZODIAC paper's published benchmark numbers
- Preprocessed spectra and candidate lists are already available locally, so deposit retrieval would only duplicate existing files
- The dataset has no published MassIVE accession or author-provided ground truth to validate against

## Inputs

- MassIVE deposit MSV000080502 (dendroides dataset): raw/processed LC-MS/MS spectra
- MassIVE deposit MSV000081364 (NIST1950 dataset)
- groundtruth_dendroides.csv with per-compound ground-truth ionization adducts
- SIRIUS top-50 molecular formula candidate lists per compound
- Authors' virtual machine image from https://bio.informatik.uni-jena.de/data/
- Patch file with OpenMS 2.4.0 modifications for the ZODIAC workflow

## Outputs

- Reproduced SIRIUS and ZODIAC top-ranked molecular formulas per dendroides compound
- Accuracy table (e.g. dendroides_known_adducts_accuracy.csv) reporting SIRIUS and ZODIAC annotation accuracies under the known-adducts condition

## How to apply

Resolve the paper's MassIVE accessions — dendroides is MSV000080502 and NIST1950 is MSV000081364 — and download the associated raw spectra and processed candidate lists rather than re-running peak picking, unless the full preprocessing chain is the target. If preprocessing must be rerun, install the modified OpenMS 2.4.0 branch (release_2.4.0_modifications_ZODIAC_workflow) plus the patch file from https://bio.informatik.uni-jena.de/data/, because stock OpenMS does not support the ZODIAC workflow. Load groundtruth_dendroides.csv to obtain each compound's ground-truth ionization adduct, since adducts cannot be reliably deduced from isotope pattern and MS/MS data alone. Feed the retrieved top-50 SIRIUS candidate lists into ZODIAC's Gibbs sampling reranker under the known-adducts constraint, then compare predicted formulas against the ground truth established by spectral library search and manual validation. Verify reproduction by confirming the reported accuracies: SIRIUS 67.17% and ZODIAC 99.50% on dendroides under the known-adducts condition, a 66-fold error rate decrease. Pin tool versions from the published releases (e.g. sirius-ms/sirius releases) so the reranking environment matches the paper's.

## Related tools

- **SIRIUS** (Command-line host of the ZODIAC Gibbs sampling reranker; the retrieved candidate lists are executed against it for the known-adducts ablation) — https://github.com/sirius-ms/sirius
- **OpenMS** (Modified OpenMS 2.4.0 branch plus patch file used to reproduce the preprocessing step when the deposit's processed spectra are not used as-is) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Evaluation signals

- MassIVE accession MSV000080502 resolves and the file listing corresponds to the dendroides dataset described in the paper
- groundtruth_dendroides.csv covers the 135 dendroides compounds with ground-truth adducts established by spectral library search and manual validation
- Recomputed accuracies match the published values: SIRIUS 67.17% (135 compounds correct) and ZODIAC 99.50% under the known-adducts condition
- Implied error-rate ratio (SIRIUS vs ZODIAC) is approximately 66-fold, matching the reported 66-fold error rate decrease
- Output CSV contains paired SIRIUS and ZODIAC accuracy entries rather than a single-tool result

## Limitations

- Reproducibility requires a modified version of OpenMS 2.4.0 plus a separate patch file rather than stock OpenMS
- Adduct information cannot be reliably deduced from isotope pattern and MS/MS data; the ground-truth adduct file is mandatory for the known-adducts condition
- SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist) are restricted to academic use under the publisher's terms of service

## Evidence

- [discussion] The modified version of OpenMS 2.4.0 is available on GitHub: [evidence span withheld — anchor status: not_found]
