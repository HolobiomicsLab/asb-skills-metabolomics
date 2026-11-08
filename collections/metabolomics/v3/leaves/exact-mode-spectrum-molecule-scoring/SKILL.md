---
name: exact-mode-spectrum-molecule-scoring
description: Use when you have MGF, mzXML, or mzML spectra with known precursor m/z
  and want to identify the intact candidate molecule (no variant mass shifts) by scoring
  fragments against a chemical database such as NPAtlas.
when_to_use_negative:
- Precursor mass differs from the candidate molecule's mass due to a variant modification
  — in that case use VInSMoC's variable mode, since exact mode only matches molecules
  whose masses match the query spectra's precursor m/z.
- Spectra were already peak-merged and peak-filtered upstream and you cannot adjust
  the advanced Peak Merge Threshold / Peaks Per Window settings — defaults assume
  unprocessed spectra.
- Charge state exceeds 1 — maximum absolute charge of 1 restricts conversion to [M+H]+
  or [M-H]-; multiply charged data requires changing this setting, which exact-mode
  defaults do not address.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: VInSMoC
  role: 'Mass spectral search tool run in exact mode: builds metabolite and fragmentation
    graphs from candidates, lazily expands fragments explained by query peaks, matches
    fragments within product ion tolerance, and computes peak count scores.'
  repo: https://github.com/mohimanilab/VInSMoC
- name: run.npanalysis.org
  role: Web server that hosts VInSMoC; provides project/data/task management, task
    parameter forms (input data, search mode, ion mode, tolerances, charge, minimum
    score), and a results page for task IDs.
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:79ee0497393cbfc06b8543166db322b99fe81578ddb8bf02b9ce15307e791789
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - run.npanalysis.org
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# exact-mode-spectrum-molecule-scoring

## Summary

Exact-mode scoring of query mass spectra against a candidate molecule database with VInSMoC: only candidates whose mass matches the spectrum's precursor m/z are scored, using adaptive fragmentation-graph expansion, [M+H]+/[M-H]- adduct conversion, and a peak count score.

## When to use

Use when you have MGF, mzXML, or mzML spectra with known precursor m/z and want to identify the intact candidate molecule (no variant mass shifts) by scoring fragments against a chemical database such as NPAtlas.

## When NOT to use

- Precursor mass differs from the candidate molecule's mass due to a variant modification — in that case use VInSMoC's variable mode, since exact mode only matches molecules whose masses match the query spectra's precursor m/z.
- Spectra were already peak-merged and peak-filtered upstream and you cannot adjust the advanced Peak Merge Threshold / Peaks Per Window settings — defaults assume unprocessed spectra.
- Charge state exceeds 1 — maximum absolute charge of 1 restricts conversion to [M+H]+ or [M-H]-; multiply charged data requires changing this setting, which exact-mode defaults do not address.

## Inputs

- Query mass spectrum file in MGF, mzXML, or mzML format (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287)
- Candidate chemical database (e.g. NPAtlas) accessible through run.npanalysis.org
- Task parameters: search mode (exact), fragmentation mode (Metabolite or Peptide), ion mode (positive/negative), precursor and product ion tolerances, maximum absolute charge, minimum score reporting threshold

## Outputs

- Task results page on run.npanalysis.org listing the best hit per input scan exceeding the minimum score threshold, with each molecule linked to its original chemical database entry
- Per-scan peak count scores for scored candidate molecules

## How to apply

Run VInSMoC with search mode set to 'exact' and fragmentation mode 'Metabolite' (or 'Peptide' for generalized peptides), matching ion mode to the input spectrum and leaving maximum absolute charge at 1 so only [M+H]+ or [M-H]- adducts are used to convert m/z to masses. For each candidate molecule, build the metabolite graph via a single-pass depth-first traversal, then construct depth-1 fragmentation graph nodes using Hopcroft-Tarjan cut-vertex finding; compute deeper fragmentation-graph fragments lazily — a fragment at depth > 1 is computed only when its parent has been explained by a query peak, and higher-depth fragments are stored to avoid duplicate computation. Preprocess the spectrum by merging peaks within the Peak Merge Threshold (keeping at least Minimum Peak Count peaks) and filtering to keep the Peaks Per Window most intense peaks per Peak Filtering Window m/z window. Match theoretical fragments to experimental peaks within the product ion tolerance, compute the exact-mode peak count score per candidate, and report hits whose score exceeds the configured minimum score; each reported molecule links back to its original database entry (e.g. NPAtlas).

## Related tools

- **VInSMoC** (Mass spectral search tool run in exact mode: builds metabolite and fragmentation graphs from candidates, lazily expands fragments explained by query peaks, matches fragments within product ion tolerance, and computes peak count scores.) — https://github.com/mohimanilab/VInSMoC
- **run.npanalysis.org** (Web server that hosts VInSMoC; provides project/data/task management, task parameter forms (input data, search mode, ion mode, tolerances, charge, minimum score), and a results page for task IDs.) — https://run.npanalysis.org

## Evaluation signals

- The task on run.npanalysis.org reaches the 'Finished' state with no 'error' status on the input data point (data parsed successfully after upload).
- Each reported scan has a best-hit molecule whose peak count score exceeds the configured minimum score reporting threshold.
- For each hit, the theoretical fragment-to-peak matches fall within the configured product ion tolerance.
- Every reported molecule is linked to its entry in the original chemical database (e.g. NPAtlas), confirming traceability from result back to source database record.
- With maximum absolute charge 1 and positive ion mode, precursor-mass matching used only the [M+H]+ adduct for m/z-to-mass conversion.

## Limitations

- Current tools for database search of experimental spectra against molecule libraries are not scalable; VInSMoC addresses this with lazy fragmentation-graph expansion, but exact mode still only matches masses equal to the precursor m/z (no variant mass shifts).
- Advanced peak preprocessing defaults (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) should be left unchanged unless spectra were preprocessed before upload — altering them without understanding the raw data can distort matching.
- Fragmentation-graph computation is adaptive: fragments at depth > 1 are computed only if the parent fragment was explained by a query peak, so very sparse spectra may leave deep fragments unevaluated.
- Data upload requires the declared file extension to match the actual format; malformed inputs surface as an error status after parsing.

## Evidence

- [full_text] Adaptive fragmentation strategy: a fragment at depth > 1 is computed only when its parent fragment has been explained by a query peak.: [evidence span withheld — anchor status: not_found]
