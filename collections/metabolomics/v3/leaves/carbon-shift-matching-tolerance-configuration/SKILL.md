---
name: carbon-shift-matching-tolerance-configuration
description: Reference for setting match thresholds when scoring filtered peak networks
  from 13C INADEQUATE spectra against a simulated INADEQUATE database of metabolites.
when_to_use_negative:
- The input spectra are unlabeled and peaks have not yet been picked and clustered
  into networks — the matching step operates on PyINETA's network output, not raw
  FIDs.
- You only need peak picking or referencing; those are separate steps (load, pick)
  in the PyINETA pipeline.
- You want to annotate compounds without a simulated INADEQUATE database available
  — PyINETA matches against this database, not general spectral libraries.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: Python package whose matching module applies the CSMT/NCMT/DQMT tolerances
    to score peak networks against the simulated INADEQUATE database
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Runtime for run_pyineta.py, which accepts the config file holding the match
    thresholds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:4bd2e0ed5bcd7ad6b08fc49c0949b4d9f36a4d1262d33a96842a5e2126e30e16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# carbon-shift-matching-tolerance-configuration

## Summary

Configure and apply the CSMT (carbon shift matching tolerance) and NCMT (connectivity matching tolerance) thresholds in PyINETA's database-matching module, deciding when a picked peak network corresponds to a metabolite in the simulated INADEQUATE database.

## When to use

Reference for setting match thresholds when scoring filtered peak networks from 13C INADEQUATE spectra against a simulated INADEQUATE database of metabolites.

## When NOT to use

- The input spectra are unlabeled and peaks have not yet been picked and clustered into networks — the matching step operates on PyINETA's network output, not raw FIDs.
- You only need peak picking or referencing; those are separate steps (load, pick) in the PyINETA pipeline.
- You want to annotate compounds without a simulated INADEQUATE database available — PyINETA matches against this database, not general spectral libraries.

## Inputs

- Filtered peak networks from PyINETA's cluster/find steps
- Simulated INADEQUATE database of metabolite networks
- config.ini with match threshold parameters (CSMT, NCMT, DQMT)

## Outputs

- Per-network match results with hit score and coverage score for each matched database compound
- Matched compound annotations for the query INADEQUATE spectra

## How to apply

After PyINETA's clustering and network-finding steps produce filtered peak networks, run the matching step, which compares each sample network to every database entry under three tolerances: CSMT (carbon chemical-shift matching tolerance in ppm), NCMT (network connectivity matching tolerance in Hz), and DQMT (double-quantum shift matching tolerance in ppm). A network is considered a match when each peak falls below the CSMT chemical-shift threshold, connectivity matches above NCMT, and the double-quantum difference falls below DQMT. Set these tolerances in the config.ini file that run_pyineta.py requires, since they govern which compound networks are retained as hits. Rationale: the tolerances trade off annotation sensitivity against false positives on 13C-labeled INADEQUATE spectra, and PyINETA records the parameters alongside outputs to keep annotation reproducible.

## Related tools

- **PyINETA** (Python package whose matching module applies the CSMT/NCMT/DQMT tolerances to score peak networks against the simulated INADEQUATE database) — https://github.com/edisonomics/PyINETA
- **Python** (Runtime for run_pyineta.py, which accepts the config file holding the match thresholds)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output -s match
```

## Evaluation signals

- Every reported match must have both a hit score and a coverage score in the matching module's output table.
- Peaks in matched networks should satisfy the CSMT chemical-shift threshold and DQCT double-quantum difference constraints used during scoring.
- Cross-check example1 results against the provided example1_default_output folder.
- The config.ini used for the run must record the match tolerances, consistent with PyINETA's design of keeping all pipeline parameters with outputs.
- Matched compounds should only be those present in the supplied simulated INADEQUATE database.

## Limitations

- The matching step depends on the simulated INADEQUATE database supplied; compounds absent from that database cannot be annotated.
- Match quality is threshold-dependent — overly loose CSMT/NCMT/DQCT tolerances produce spurious hits, overly tight ones miss true metabolites.
- PyINETA provides fragment information for unknown compounds, but these remain unidentified rather than resolved by the database match.
- Inputs and temporary files for larger examples (e.g. example2) exceed 100 Mb and require contacting the Edison lab for raw files.

## Evidence

- [full_text] Peaks match below threshold CSMT, networks match above NCMT chemical shift matches and below DQMT double-quantum difference, then each matched network receives hit and coverage scores: [evidence span withheld — anchor status: not_found]
- [intro] match networks to simulated database: "which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [readme] README example run command: [evidence span withheld — anchor status: not_found]
- [abstract] PyINETA tracks parameters: "PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
