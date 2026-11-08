---
name: threshold-based-hit-evaluation
description: Use when query spectra (e.g. ESI-MS2 peak lists from QqQ or QqTOF instruments)
  are searched against a reference spectral library and you need precision/recall/F-value
  metrics at a specific similarity-score cutoff.
when_to_use_negative:
- Reference spectra are merged across collision-induced dissociation conditions —
  the article shows merging improves precision by 21-23% at score 0.6, so the unmerged
  baseline metrics will not apply
- The goal is spectral similarity scoring itself rather than classification-level
  evaluation of top-hit correctness
- Spectra are EI-MS or volatile-compound data rather than the ESI-MS2 data for which
  the weighting exponents were optimized
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassBank-cli-tools
  role: Command-line tools referenced as the validator/tooling context for the MassBank
    spectral search and record validation workflow
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:1cef7cda0975fdcc0c7b0d740b9c829406dbf646b70b25c60e6a55679e88920f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank-cli-tools
  tools:
  - MassBank-cli-tools
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# threshold-based-hit-evaluation

## Summary

Evaluates spectral-search compound identification quality by labeling top-scoring hits as true/false matches at a fixed similarity-score threshold (e.g. 0.6) and computing precision, recall, and F-value per search direction.

## When to use

Use when query spectra (e.g. ESI-MS2 peak lists from QqQ or QqTOF instruments) are searched against a reference spectral library and you need precision/recall/F-value metrics at a specific similarity-score cutoff.

## When NOT to use

- Reference spectra are merged across collision-induced dissociation conditions — the article shows merging improves precision by 21-23% at score 0.6, so the unmerged baseline metrics will not apply
- The goal is spectral similarity scoring itself rather than classification-level evaluation of top-hit correctness
- Spectra are EI-MS or volatile-compound data rather than the ESI-MS2 data for which the weighting exponents were optimized

## Inputs

- Query ESI-MS2 peak lists (e.g. 4205 spectra for QS=[QqQ], 4431 for QS=[QqTOF])
- Target/reference ESI-MS2 peak lists from the cross-instrument subset (QqQ and QqTOF, 856 compounds common to both)
- Similarity-score threshold (0.6)

## Outputs

- Per-direction precision, recall, and F-value (six metrics, e.g. TS=[QqQ]: 0.222/0.327/0.264; TS=[QqTOF]: 0.276/0.292/0.284)
- similarity_search_metrics.csv containing the computed metric values

## How to apply

Load query and target peak lists (e.g. 4205 spectra for QS=[QqQ]/TS=[QqTOF] and 4431 for QS=[QqTOF]/TS=[QqQ] over 856 common compounds) with pandas. Score every query-target spectrum pair using the weighted cosine correlation similarity function, with weighting exponents on peak intensity and m/z as optimized for ESI-MS2 data in MassBank. For each search direction, label a query a true match if its top-scoring target is the same compound and the similarity score is at least the threshold (0.6 in the source task), otherwise a false match. Compute precision, recall and F-value from the true/false counts per target set, write the six metric values to a CSV (e.g. similarity_search_metrics.csv), and validate against reported values (TS=[QqQ]: 0.222/0.327/0.264; TS=[QqTOF]: 0.276/0.292/0.284). The rationale is that threshold-based hit evaluation quantifies how well unmerged reference spectra identify compounds, providing a baseline against which data-merging improvements (21-23% precision gain at score 0.6) can be measured.

## Related tools

- **MassBank-cli-tools** (Command-line tools referenced as the validator/tooling context for the MassBank spectral search and record validation workflow) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
python evaluate_hits.py --query QS_QqQ --target TS_QqTOF --threshold 0.6 --out similarity_search_metrics.csv
```

## Evaluation signals

- TS=[QqQ] direction reproduces precision 0.222, recall 0.327, F-value 0.264 at similarity-score threshold 0.6
- TS=[QqTOF] direction reproduces precision 0.276, recall 0.292, F-value 0.284 at similarity-score threshold 0.6
- Spectrum-pair counts match the source: 4205 pairs for QS=[QqQ]/TS=[QqTOF] and 4431 for QS=[QqTOF]/TS=[QqQ] over 856 compounds
- All similarity scores are cosine-correlation values in [0,1]; a hit counts as a true match only if score >= 0.6 AND the top target is the same compound
- Output CSV similarity_search_metrics.csv contains exactly six numeric metric values

## Limitations

- Unmerged ESI-MS2 data perform poorly as reference spectra (precision ~0.22-0.28 at threshold 0.6 in the reported cross-searches)
- Results are specific to the QqQ/QqTOF cross-instrument setting and the MassBank weighted cosine correlation scoring; other instruments or scoring functions may not reproduce these values
- Distributed nature of MassBank means users must access the correct subset of data by experimental conditions, which can affect reproducibility of the search set

## Evidence

- [abstract] Merging ESI-MS2 data of identical compounds under different collision-induced dissociation conditions improved the precision of compound identification by 21-23% at a similarity score of 0.6.: [evidence span withheld — anchor status: not_found]
- [abstract] Spectral search similarity scores are calculated by a weighted cosine correlation with weighting exponents on peak intensity and m/z optimized to ESI-MS2 data.: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [abstract] MassBank is a distributed database with each research group providing data from its own servers, and users can access all or a subset of data by experimental conditions.: "MassBank is a distributed database. Each research group provides data from its own MassBank data servers distributed on the Internet."
- [readme] The validator and other command line tools are maintained in https://github.com/MassBank/MassBank-cli-tools: "The validator and other command line tools are maintained in https://github.com/MassBank/MassBank-cli-tools"
