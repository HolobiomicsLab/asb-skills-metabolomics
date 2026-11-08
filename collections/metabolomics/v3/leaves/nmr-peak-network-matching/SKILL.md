---
name: nmr-peak-network-matching
description: Use when you have filtered peak networks from an INADEQUATE NMR spectrum
  and need to identify metabolites by matching each network to a simulated INADEQUATE
  database using CSMT, NCMT, and DQMT thresholds.
when_to_use_negative:
- Input spectra are not 13C INADEQUATE experiments — the simulated database and network
  matching logic are specific to INADEQUATE peak connectivity
- Peaks have not yet been picked and clustered into networks — the matching step expects
  filtered networks as input, not raw spectra
- You need quantitative metabolite concentrations rather than compound identification/annotation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3216
tools:
- name: PyINETA
  role: Python pipeline that filters picked peaks into networks and matches them against
    the simulated INADEQUATE database via the match step
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Hosted platform where PyINETA (and thus the network-matching workflow) can
    be run without local installation
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:2fe32ad1955dd2c36dcca3fa21b0f87d16dfb42d4ac8f7a14e22e85856b96a0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3216
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# nmr-peak-network-matching

## Summary

Match filtered networks of picked INADEQUATE peaks against a simulated INADEQUATE database of metabolites to identify metabolites present in query spectra, scoring each candidate match with hit and coverage scores.

## When to use

Use when you have filtered peak networks from an INADEQUATE NMR spectrum and need to identify metabolites by matching each network to a simulated INADEQUATE database using CSMT, NCMT, and DQMT thresholds.

## When NOT to use

- Input spectra are not 13C INADEQUATE experiments — the simulated database and network matching logic are specific to INADEQUATE peak connectivity
- Peaks have not yet been picked and clustered into networks — the matching step expects filtered networks as input, not raw spectra
- You need quantitative metabolite concentrations rather than compound identification/annotation

## Inputs

- Filtered peak networks from INADEQUATE spectra (output of PyINETA clustering/finding module)
- Simulated INADEQUATE database of metabolites
- JRES spectrum and upstream Match output for the integration step

## Outputs

- Match output table listing per-network match results including hit score and coverage score for each matched network
- MatchJres output table listing for each INADEQUATE network whether complementary JRES peaks are present

## How to apply

Load the filtered peak networks produced by PyINETA's clustering/finding module and the simulated INADEQUATE database of metabolites. For each sample network, compare it against every database entry applying three match thresholds: CSMT (carbon shift matching tolerance, ppm), NCMT (network connectivity matching tolerance, Hz), and DQMT (double-quantum shift matching tolerance, ppm). Peaks match below threshold CSMT; networks match above NCMT chemical shift matches and below DQMT double-quantum difference. Score each candidate match with a hit score (how well the network matches the database compound) and a coverage score (fraction of the database network represented in the sample). Write per-network match results to the matching module's output table. Optionally follow up with the JRES integration step to attach complementary 13C-JRES peak information to each matched INADEQUATE network.

## Related tools

- **PyINETA** (Python pipeline that filters picked peaks into networks and matches them against the simulated INADEQUATE database via the match step) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Hosted platform where PyINETA (and thus the network-matching workflow) can be run without local installation)

## Examples

```
python <path_to_pyineta_repo>/run_pyineta.py -c config.ini -s match -o example1_output
```

## Evaluation signals

- Match output table exists with one row per input network and includes hit score and coverage score columns
- Run the JRES integration on a reference sample (e.g. deposited liver sample 30) and confirm 65 of 67 constructed INADEQUATE networks have corresponding JRES peaks in the MatchJres output
- Verify no sample network matched a database entry where peaks exceed CSMT or double-quantum differences exceed DQMT
- Cross-check example run output against the repository's example1_default_output folder
- Confirm that pipeline parameters and outputs are logged by PyINETA for annotation transparency

## Limitations

- Matching depends on the completeness of the simulated INADEQUATE database of metabolites; unmatched networks may represent compounds absent from the database
- The 13C-JRES integration step expects a deposited/complementary JRES spectrum; without it, only INADEQUATE-based matching is available
- Example2 in the repository requires contacting the Edison lab for raw files larger than 100 Mb, limiting out-of-the-box reproducibility for those inputs

## Evidence

- [intro] Simulated database matching with thresholds: "which it then matches to a simulated INADEQUATE database of metabolites to identify metabolites present in the query INADEQUATE spectra."
- [abstract] JRES integration result: "Our evaluation of PyINETA using a model mouse study showed that our pipeline successfully integrated INADEQUATE and 13 C-JRES."
- [readme] Match step availability in CLI: [evidence span withheld — anchor status: not_found]
- [abstract] Transparency of annotation: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
