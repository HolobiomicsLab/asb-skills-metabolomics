---
name: double-quantum-shift-matching-tolerance-configuration
description: 'Use when running the PyINETA matching step on 13C INADEQUATE peak networks:
  candidate network-to-database assignments must be gated by a double-quantum shift
  tolerance (DQMT, ppm) in addition to the carbon-shift (CSMT) and connectivity (NCMT)
  tolerances before scoring.'
when_to_use_negative:
- When the input is raw INADEQUATE FID/spectra that have not yet been peak-picked
  and clustered into networks — DQMT only gates the matching stage, not reading, referencing
  or peak picking
- When analyzing spectra from unlabeled samples without the 13C INADEQUATE double-quantum
  dimension, since there is no double-quantum shift to compare against DQMT
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: PyINETA
  role: Runs the match step that applies DQMT alongside CSMT and NCMT to filter candidate
    network–database pairs before assigning hit and coverage scores
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Hosted execution environment where PyINETA (and therefore DQMT configuration)
    is available
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:390383ae722a7ba2633a666fbf9ac7de533cc284962c47aa567286c44d1b53b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# double-quantum-shift-matching-tolerance-configuration

## Summary

Configures and applies the DQMT (double-quantum shift matching tolerance, ppm) threshold in PyINETA so that a picked peak network from an INADEQUATE spectrum only matches a simulated metabolite database entry when its double-quantum shift differences fall within tolerance, alongside the CSMT and NCMT checks, before hit and coverage scores are assigned.

## When to use

Use when running the PyINETA matching step on 13C INADEQUATE peak networks: candidate network-to-database assignments must be gated by a double-quantum shift tolerance (DQMT, ppm) in addition to the carbon-shift (CSMT) and connectivity (NCMT) tolerances before scoring.

## When NOT to use

- When the input is raw INADEQUATE FID/spectra that have not yet been peak-picked and clustered into networks — DQMT only gates the matching stage, not reading, referencing or peak picking
- When analyzing spectra from unlabeled samples without the 13C INADEQUATE double-quantum dimension, since there is no double-quantum shift to compare against DQMT

## Inputs

- Filtered peak networks from PyINETA's cluster and find steps of 13C INADEQUATE spectra
- Simulated INADEQUATE database of metabolite networks (for reference compounds)
- config.ini configuration file with CSMT (ppm), NCMT (Hz) and DQMT (ppm) thresholds

## Outputs

- Per-network match results table listing each matched database network with its hit score and coverage score
- Matched network plots (single-network vs database-metabolite overlays via -s singlePlot)

## How to apply

Load the filtered peak networks produced by PyINETA's cluster/find steps and the simulated INADEQUATE database of metabolites. In the config.ini file used by run_pyineta.py, set the DQMT parameter — the double-quantum shift matching tolerance in ppm — together with CSMT (carbon shift matching tolerance, ppm) and NCMT (network connectivity matching tolerance, Hz). During the match step, every peak in each sample network is compared to the simulated database: a peak matches below the CSMT threshold, and a network matches a database entry only when chemical shifts agree above NCMT and the double-quantum difference falls below DQMT. Only networks that pass these three thresholds proceed to hit and coverage scoring, where the hit score reflects how well the network matches the compound and the coverage score reflects the fraction of the database network represented in the sample. Rationale: the double-quantum dimension encodes carbon–carbon connectivity in INADEQUATE, so a permissive DQMT would incorrectly link peaks from different connectivities, while an overly tight DQMT discards true network matches; the tolerance must therefore be tuned jointly with CSMT and NCMT for the spectrometer's field and referencing quality.

## Related tools

- **PyINETA** (Runs the match step that applies DQMT alongside CSMT and NCMT to filter candidate network–database pairs before assigning hit and coverage scores) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Hosted execution environment where PyINETA (and therefore DQMT configuration) is available)

## Examples

```
python run_pyineta.py -c config.ini -s match
```

## Evaluation signals

- Matched networks in the output table must have double-quantum differences below the configured DQMT value in ppm; any assignment with a DQ above DQMT indicates a misconfiguration
- Hit and coverage scores must be present for every matched network — a network that passes CSMT/NCMT/DQMT but lacks a hit or coverage score signals a matching-step failure
- Coverage score should fall between 0 and 1, representing the fraction of the database network represented in the sample
- Compare matched-network lists against the example1_default_output reference to confirm the DQMT setting reproduces the expected assignments
- Varying DQMT should monotonically change the number of matched networks — non-monotonic or flat responses indicate the tolerance was not actually applied

## Limitations

- DQMT interacts with CSMT and NCMT: a network that passes two tolerances but fails the third is rejected, so threshold values must be co-tuned and cannot be validated in isolation
- Tolerance values are configuration-dependent and must be re-validated when spectra are referenced differently or acquired at a different field strength; the article does not state a single recommended default
- PyINETA assumes peak networks (ideally) come from the same compound; noise or overlapped peaks in the double-quantum dimension can cause incorrect matches even within DQMT

## Evidence

- [full_text] peaks match below threshold CSMT, networks match above NCMT chemical shift matches and below DQMT double-quantum difference: [evidence span withheld — anchor status: not_found]
- [readme] matches to a simulated INADEQUATE database of metabolites: "which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra"
- [intro] filters these picked peaks to identify networks of peaks: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound"
- [abstract] keeps track of all the pipeline parameters and outputs: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
