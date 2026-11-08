---
name: network-coverage-scoring
description: 'Use when annotating metabolites from filtered INADEQUATE peak networks:
  after clustering and network finding, each candidate network must be scored against
  simulated database entries via hit and coverage scores before annotation.'
when_to_use_negative:
- Input is a raw or unfiltered INADEQUATE spectrum — peaks must first be picked, clustered,
  and formed into networks by the upstream PyINETA steps.
- The analysis goal is 1D or JRES-only annotation without INADEQUATE network connectivity
  — this skill relies on double-quantum connectivity between carbon pairs.
- Comparing sample networks against an arbitrary spectral library rather than a simulated
  INADEQUATE database of metabolites.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0593
tools:
- name: PyINETA
  role: Python package that filters picked INADEQUATE peaks into networks and matches
    them to a simulated INADEQUATE database of metabolites, assigning hit and coverage
    scores to each match
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime for PyINETA; run_pyineta.py executes the match step (e.g. -s match)
    that produces the scored network matches
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Public hosting environment where PyINETA (including the matching/scoring module)
    is installed and available
  repo: https://nmrbox.org
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:ab6941c2d7f2e880d23ce340fafc94c3b5e4f72e63b5dc7aa2cfd114125feb5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Network coverage scoring

## Summary

Score each peak network matched by PyINETA's INADEQUATE database-search step using a hit score (how well the network matches a database compound) and a coverage score (fraction of the database network represented in the sample), enabling compound annotation from 13C INADEQUATE spectra.

## When to use

Use when annotating metabolites from filtered INADEQUATE peak networks: after clustering and network finding, each candidate network must be scored against simulated database entries via hit and coverage scores before annotation.

## When NOT to use

- Input is a raw or unfiltered INADEQUATE spectrum — peaks must first be picked, clustered, and formed into networks by the upstream PyINETA steps.
- The analysis goal is 1D or JRES-only annotation without INADEQUATE network connectivity — this skill relies on double-quantum connectivity between carbon pairs.
- Comparing sample networks against an arbitrary spectral library rather than a simulated INADEQUATE database of metabolites.

## Inputs

- Filtered peak networks from PyINETA's clustering/finding steps (ideally peaks from the same compound)
- Simulated INADEQUATE database of metabolite networks
- Match thresholds CSMT (carbon shift, ppm), NCMT (network connectivity, Hz), and DQMT (double-quantum shift, ppm)

## Outputs

- Per-network match results table listing each matched database network with its hit score and coverage score
- Annotated list of metabolites present in the query INADEQUATE spectra

## How to apply

Take the filtered peak networks produced by PyINETA's clustering and finding steps as input, together with a simulated INADEQUATE database of metabolite networks. For each sample network, compare it against every database entry by testing whether each peak matches within the CSMT carbon-shift tolerance (ppm), whether network connectivity matches within the NCMT tolerance (Hz), and whether double-quantum shift differences fall below the DQMT tolerance (ppm). For networks passing these match criteria, compute two complementary scores: a hit score reflecting how well the network matches the database compound, and a coverage score reflecting what fraction of the database network's peaks are represented in the sample. Write the per-network results, including hit and coverage scores for every matched network, to the matching module's output table so downstream annotation can be judged on both match quality and database completeness.

## Related tools

- **PyINETA** (Python package that filters picked INADEQUATE peaks into networks and matches them to a simulated INADEQUATE database of metabolites, assigning hit and coverage scores to each match) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime for PyINETA; run_pyineta.py executes the match step (e.g. -s match) that produces the scored network matches) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Public hosting environment where PyINETA (including the matching/scoring module) is installed and available) — https://nmrbox.org

## Examples

```
python run_pyineta.py -c config.ini -s match -o example1_output
```

## Evaluation signals

- Every matched network in the output table has both a hit score and a coverage score populated; absent or null scores indicate the match step did not complete.
- Networks exceeding the CSMT carbon-shift tolerance (ppm) or the NCMT connectivity tolerance (Hz) are absent from the matched-results table — unmatched networks must not appear.
- Double-quantum shift differences for each matched pair must fall below the DQMT tolerance (ppm); pairs exceeding DQMT should not be scored.
- Coverage scores should reflect the fraction of the database network represented in the sample, so coverage for a full database compound equals its maximum; partial matches yield lower coverage values.
- The output table's per-network rows correspond one-to-one with sample networks entering the match step, and results are reproducible when re-run with the same config.ini thresholds.

## Limitations

- Matching quality depends on correct tuning of three thresholds (CSMT, NCMT, DQMT); poor calibration either drops true matches or floods results with false ones.
- Coverage alone does not guarantee a correct annotation — a partial network can still match a database compound, so hit score must be read jointly with coverage.
- PyINETA relies on 13C labeling and INADEQUATE connectivity; systems that cannot be labeled or lack sufficient double-quantum signal cannot produce meaningful network matches.
- Example2 in the repository requires raw files >100 MB that must be requested from the Edison lab, limiting out-of-the-box large-scale testing.

## Evidence

- [methods] Three thresholds govern matching: CSMT, NCMT, DQMT; hit and coverage scores are computed for each matched network.: [evidence span withheld — anchor status: not_found]
- [intro] PyINETA matches picked peak networks to a simulated INADEQUATE database to identify metabolites present in the query spectra.: "matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra"
- [readme] The match step is invoked by the PyINETA CLI with -s match, per the README usage.: [evidence span withheld — anchor status: external_doc]
- [abstract] PyINETA is an open-source platform for INADEQUATE network analysis that provides structural information of molecules.: "Here, we introduce PyINETA, Python-based INADEQUATE network analysis. PyINETA is an open-source platform that provides structural information of molecules using INADEQUATE"
