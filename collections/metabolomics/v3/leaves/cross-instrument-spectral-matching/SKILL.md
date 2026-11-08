---
name: cross-instrument-spectral-matching
description: Use when you need to measure how well MS2 reference spectra acquired
  on one instrument type identify compounds queried from another (e.g. QS=[QqQ] vs
  TS=[QqTOF]), and to evaluate whether merged ESI-MS2 target sets improve precision
  at a fixed similarity-score cutoff.
when_to_use_negative:
- Your queries and targets come from the same instrument and identical acquisition
  conditions — the unmerged cross-instrument baseline and merged-set evaluation do
  not apply.
- Your spectra are EI-MS or GC-MS volatility data; the weighted cosine exponents here
  are optimized for ESI-MS2 data only.
- You need a screening result rather than a precision/recall benchmark of a spectral-search
  reference library.
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MassBank3
  role: Current MassBank web app / REST API for serving and searching the spectral
    reference library
  repo: https://github.com/MassBank/MassBank3
- name: MassBank-cli-tools
  role: Java command line tools, including the validator, used to run/validate the
    spectral-search workflow
  repo: https://github.com/MassBank/MassBank-cli-tools
- name: MassBank-web
  role: Deprecated MassBank software; its MassBank Record Format documentation still
    defines the record files used as input
  repo: https://github.com/MassBank/MassBank-web
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:ab1cddfca49cfe3e2389651ee1fc7224977c02db77fdaff4dc892bc81cac2f0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank-cli-tools
  - MassBank-web
  - MassBank3
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-instrument-spectral-matching

## Summary

Cross-searches ESI-MS2 query spectra from one instrument (QqQ or QqTOF) against reference/target spectra from another using MassBank's weighted cosine correlation similarity score, optionally with merged collision-energy spectra, to quantify precision/recall/F-value of compound identification.

## When to use

Use when you need to measure how well MS2 reference spectra acquired on one instrument type identify compounds queried from another (e.g. QS=[QqQ] vs TS=[QqTOF]), and to evaluate whether merged ESI-MS2 target sets improve precision at a fixed similarity-score cutoff.

## When NOT to use

- Your queries and targets come from the same instrument and identical acquisition conditions — the unmerged cross-instrument baseline and merged-set evaluation do not apply.
- Your spectra are EI-MS or GC-MS volatility data; the weighted cosine exponents here are optimized for ESI-MS2 data only.
- You need a screening result rather than a precision/recall benchmark of a spectral-search reference library.

## Inputs

- ESI-MS2 peak lists for 856 compounds from QqQ and QqTOF instruments (4205 and 4431 spectra depending on direction)
- Merged ESI-MS2 MassBank record sets (superposition of spectra recorded at different collision energies) as target entries
- MassBank record files in MassBank Record Format (.txt, accession-ID filenames)

## Outputs

- Per-direction precision, recall, and F-value at similarity-score threshold 0.6 (e.g. similarity_search_metrics.csv with six metric values)
- Merged vs unmerged precision comparison quantifying the 21-23% precision gain

## How to apply

Load the query and target ESI-MS2 peak lists (856 compounds common to the QqQ and QqTOF subsets; 4205 spectra for QS=[QqQ]/TS=[QqTOF] and 4431 for the reverse direction) with pandas, or use MassBank records in MassBank record format. Score every query-target spectrum pair with the weighted cosine correlation similarity function in which weighting exponents on peak intensity and m/z are optimized to ESI-MS2 data, as implemented in the MassBank spectral search. Label a query a true match when the top-scoring target is the same compound and its similarity score is at least 0.6, otherwise a false match; compute precision, recall, and F-value per direction. Optionally repeat with merged target sets, in which ESI-MS2 spectra of identical compounds recorded at different collision-induced dissociation conditions are superposed into single target entries, to compare precision against the unmerged baseline. Published reference values at score 0.6: unmerged TS=[QqQ] [0.222, 0.327, 0.264] and TS=[QqTOF] [0.276, 0.292, 0.284]; merged TS=[Merged QqQ] [0.454, 0.307, 0.366] and TS=[Merged QqTOF] [0.490, 0.299, 0.371], i.e. 21-23% precision gain at unchanged recall.

## Related tools

- **MassBank3** (Current MassBank web app / REST API for serving and searching the spectral reference library) — https://github.com/MassBank/MassBank3
- **MassBank-cli-tools** (Java command line tools, including the validator, used to run/validate the spectral-search workflow) — https://github.com/MassBank/MassBank-cli-tools
- **MassBank-web** (Deprecated MassBank software; its MassBank Record Format documentation still defines the record files used as input) — https://github.com/MassBank/MassBank-web

## Examples

```
import pandas as pd; df = pd.read_csv('peaks.csv'); score = lambda q,t: cosine_weights(q,t); matches = df.groupby('query').apply(top_hit); metrics = precision_recall_f(qy, ts, cutoff=0.6)
```

## Evaluation signals

- Unmerged baseline at score 0.6 reproduces TS=[QqQ] precision/recall/F = [0.222, 0.327, 0.264] and TS=[QqTOF] = [0.276, 0.292, 0.284].
- Merged-search metrics at score 0.6 reproduce TS=[Merged QqQ] = [0.454, 0.307, 0.366] and TS=[Merged QqTOF] = [0.490, 0.299, 0.371].
- Precision gain over the corresponding unmerged search is 23% (QqQ targets) and 21% (QqTOF targets) while recall is essentially unchanged.
- Spectrum counts per direction match 4205 (QS=[QqQ]/TS=[QqTOF]) and 4431 (QS=[QqTOF]/TS=[QqQ]) over the 856 common compounds.
- True-match definition holds: top-scoring target compound equals query compound AND similarity score >= 0.6.

## Limitations

- Unmerged ESI-MS2 reference data perform poorly cross-instrument (precision 0.222-0.276), so naive matching should not be expected to work.
- The 0.6 similarity-score cutoff and the weighted cosine exponents are specific to this ESI-MS2 evaluation; other cutoffs or MS levels need re-optimization.
- The similarity score is compared against the NIST Mass Spectral Library approach, which also uses a weighted cosine correlation — results are score-implementation dependent.

## Evidence

- [abstract] Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6.: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [abstract] Weighted cosine similarity optimized to ESI-MS2: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [intro] CLI tools location: "The validator and other command line tools are maintained in https://github.com/MassBank/MassBank-cli-tools"
- [readme] Record format requirement: [evidence span withheld — anchor status: not_found]
