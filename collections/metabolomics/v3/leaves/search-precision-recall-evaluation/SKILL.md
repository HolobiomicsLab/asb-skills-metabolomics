---
name: search-precision-recall-evaluation
description: Use when you need to quantify the identification accuracy of a spectral
  search against a reference library — e.g., comparing merged vs. unmerged ESI-MS2
  target sets, tuning a similarity-score threshold, or cross-instrument (QqQ vs. QqTOF)
  query-target evaluation.
when_to_use_negative:
- You need de novo structure elucidation rather than library-based identification
  scoring — this skill only evaluates search performance against a reference set
- Your data are EI-MS or FAB-MS rather than ESI-MS2, since the weighted cosine correlation
  exponents were optimized specifically to ESI-MS2 data
- The reference library has no possibility of merging records under different collision
  energies, so the merged-vs-unmerged precision comparison is not applicable
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassBank-web
  role: Contains the MassBank software; deprecated in favor of MassBank3 for the web
    app, but documents the MassBank record format used by the search targets
  repo: https://github.com/MassBank/MassBank-web
- name: MassBank3
  role: Current MassBank web app and REST API serving the spectral reference library
    searched against, deployable via Docker Compose with MassBank-data imported into
    /MassBank3/data/MassBank-data
  repo: https://github.com/MassBank/MassBank3
- name: MassBank-cli-tools
  role: Java command line tools including the validator for MassBank records used
    as search targets
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:e1c97c13b7bcf23464cb0b1c7eba3473dcf77eaf1b97f7f7ef9ca7a17ccaaa1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank-web
  tools:
  - MassBank-cli-tools
  - MassBank-web
  - MassBank3
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# search-precision-recall-evaluation

## Summary

Evaluates spectral-search compound identification by computing precision, recall, and F-value at a fixed similarity-score cutoff, testing whether merged ESI-MS2 target spectra (superposition of spectra at different collision energies) improve precision without reducing recall in MassBank searches.

## When to use

Use when you need to quantify the identification accuracy of a spectral search against a reference library — e.g., comparing merged vs. unmerged ESI-MS2 target sets, tuning a similarity-score threshold, or cross-instrument (QqQ vs. QqTOF) query-target evaluation.

## When NOT to use

- You need de novo structure elucidation rather than library-based identification scoring — this skill only evaluates search performance against a reference set
- Your data are EI-MS or FAB-MS rather than ESI-MS2, since the weighted cosine correlation exponents were optimized specifically to ESI-MS2 data
- The reference library has no possibility of merging records under different collision energies, so the merged-vs-unmerged precision comparison is not applicable

## Inputs

- Merged ESI-MS2 MassBank spectral record sets (targets) in MassBank record file format (.txt files with accession ID and peaks)
- Query ESI-MS2 spectra acquired on QqQ and QqTOF instruments
- Ground-truth compound annotations for computing true/false positives

## Outputs

- Precision, recall, and F-value at similarity-score cutoff 0.6 per query-target pairing
- Relative precision improvement of merged vs. unmerged target sets (expected ~21-23%)
- Identification calls (matched compound candidates above cutoff) per query spectrum

## How to apply

Load target spectra in MassBank record format, merging ESI-MS2 records of identical compounds acquired under different collision-induced dissociation conditions into single target entries. Run the MassBank spectral search using the weighted cosine correlation similarity score, with weighting exponents on peak intensity and m/z optimized for ESI-MS2 data, pairing QqQ queries against QqTOF targets and vice versa. Apply a similarity-score cutoff of 0.6 to call compound identifications, then compute precision, recall, and F-measure against ground truth. Compare merged-target precision against the corresponding unmerged baseline at the same recall; in the source study this yielded precision/recall/F of [0.454, 0.307, 0.366] for TS=[Merged QqQ] and [0.490, 0.299, 0.371] for TS=[Merged QqTOF], a 21-23% precision improvement. Vary the cutoff as a decision point if the precision/recall trade-off needs exploring, but report values at a fixed threshold for comparability.

## Related tools

- **MassBank-web** (Contains the MassBank software; deprecated in favor of MassBank3 for the web app, but documents the MassBank record format used by the search targets) — https://github.com/MassBank/MassBank-web
- **MassBank3** (Current MassBank web app and REST API serving the spectral reference library searched against, deployable via Docker Compose with MassBank-data imported into /MassBank3/data/MassBank-data) — https://github.com/MassBank/MassBank3
- **MassBank-cli-tools** (Java command line tools including the validator for MassBank records used as search targets) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
cd MassBank3/compose && cp env.dist .env && docker compose up -d && curl http://localhost:8081/MassBank-api/records/count
```

## Evaluation signals

- Precision/recall/F at cutoff 0.6 reproduce the reported merged-search values: TS=[Merged QqQ] = [0.454, 0.307, 0.366] and TS=[Merged QqTOF] = [0.490, 0.299, 0.371]
- Precision gain of 21% (QqTOF) and 23% (QqQ) over the unmerged-baseline searches is reproduced at unchanged recall
- Every query-target identification call is supported by a weighted cosine correlation similarity score >= 0.6
- All target records used pass the MassBank-cli-tools validator and conform to the MassBank record format with matching accession IDs and filenames
- Recall for merged target sets is not significantly lower than the unmerged baseline (no recall drop)

## Limitations

- The 21-23% precision improvement was measured at a similarity score of 0.6; the advantage may differ at other cutoffs
- Merging requires multiple ESI-MS2 records of the same compound acquired under different collision-induced dissociation conditions; compounds with single-condition records cannot benefit
- The similarity score's weighting exponents on peak intensity and m/z are optimized to ESI-MS2 data, so transfer to other ionization modes is unvalidated
- The MassBank-web repo is deprecated; users should use MassBank3 for the web app and MassBank-cli-tools for the validator and command line tools

## Evidence

- [abstract] Merging ESI-MS2 data of identical compounds under different collision-induced dissociation conditions improved the precision of compound identification by 21-23% at a similarity score of 0.6.: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [abstract] Weighted cosine similarity score optimized for ESI-MS2.: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [readme] MassBank record format requirement for target files.: [evidence span withheld — anchor status: not_found]
- [readme] MassBank-web deprecation notice directing to maintained repos.: "This repo is deprecated. Please do not use it! The new MassBank web app is maintained in repo https://github.com/MassBank/MassBank3."
