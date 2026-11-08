---
name: natural-abundance-correction
description: Use when you have unscaled isotopologue distribution tables (MIDs) from
  XCMS-pre-processed HRMS isotope-labeling experiments and need natural-abundance-corrected
  MIDs and average labeling values before flux analysis or clustering.
when_to_use_negative:
- Input is already an NA-corrected MIDs table or was produced by get_table_objects_NA_corrected,
  which applies NACorrectionFxnII internally — re-correcting would double-correct
- Data were acquired on nominal-resolution instruments, where dual-labeled isotopologues
  cannot be resolved and the correction model assumptions fail
- The tracer/label scheme is unknown, since IsoCorrectoR corrections require the correct
  elemental formula and label scheme to compute expected natural-abundance patterns
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SIMPEL
  role: R package providing NACorrectionFxnII and get_table_objects_NA_corrected,
    which wrap IsoCorrectoR to correct isotopologue distributions and average labeling
    values
  repo: https://github.com/SIMPELmetabolism/SIMPEL
- name: IsoCorrectoR
  role: Underlying natural abundance correction engine invoked by NACorrectionFxnII
    on the unscaled MIDs table
- name: XCMS
  role: Pre-processing tool that produced the HRMS feature table imported into SIMPEL
    before correction
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-05844-z
    title: 'SIMPEL: using stable isotopes to elucidate dynamics of context specific
      metabolism'
  source_content_hash: sha256:a77cf926907870864aeee51ba26db2690d1695a2343aeedb1cd6fa02ed170b72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SIMPELmetabolism/SIMPEL
  tools:
  - IsoCorrectoR
  - R
  - SIMPEL
  - XCMS
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# natural-abundance-correction

## Summary

Corrects isotopologue distributions from isotope-enriched HRMS metabolomics data for the contribution of naturally occurring heavy isotopes (13C, 15N, etc.), using SIMPEL's NACorrectionFxnII wrapper around IsoCorrectoR so that measured M+1, M+2... signals reflect biological labeling rather than natural abundance.

## When to use

Use when you have unscaled isotopologue distribution tables (MIDs) from XCMS-pre-processed HRMS isotope-labeling experiments and need natural-abundance-corrected MIDs and average labeling values before flux analysis or clustering.

## When NOT to use

- Input is already an NA-corrected MIDs table or was produced by get_table_objects_NA_corrected, which applies NACorrectionFxnII internally — re-correcting would double-correct
- Data were acquired on nominal-resolution instruments, where dual-labeled isotopologues cannot be resolved and the correction model assumptions fail
- The tracer/label scheme is unknown, since IsoCorrectoR corrections require the correct elemental formula and label scheme to compute expected natural-abundance patterns

## Inputs

- Unscaled isotopologue distribution table (MIDs) from MIDsTableNoScale applied to an XCMS-pre-processed HRMS feature table
- Label scheme object defining the isotopic tracer (e.g., [13C5,15N2]glutamine or [U-13C6]glucose)
- Compound database with elemental formulas (used internally by IsoCorrectoR for theoretical abundance patterns)

## Outputs

- Natural-abundance-corrected isotopologue distribution table (e.g., NA_corrected_MIDs.csv)
- Average labeling values per compound per time point
- For MS2 acyl-chain analysis: NA-corrected fractional enrichments convertible to natom equivalents of 13C

## How to apply

First generate an unscaled isotopologue distribution table with MIDsTableNoScale(XCMS_data, label_scheme) from the XCMS feature table. Then call NACorrectionFxnII(MIDs_table, label_scheme, output), which wraps IsoCorrectoR and requires the label scheme describing the tracer (e.g., [13C5,15N2]glutamine for the Arabidopsis dual-label experiment or [U-13C6]glucose for the Camelina lipidomics time course) so corrections match each compound's elemental formula. The function returns the NA-corrected MIDs table and average labeling values, which should be exported as a CSV (e.g., NA_corrected_MIDs.csv). For dual-isotope data, HRMS resolution matters: corrections must distinguish multivariate isotopologues (I = product of (Ne+1)) that nominal-resolution MS cannot resolve. Validation involves checking that natural-abundance contributions are removed from unlabeled (0 h) samples while label enrichment increases over the feeding time course (0-8 h Arabidopsis, 0-32 h Camelina).

## Related tools

- **SIMPEL** (R package providing NACorrectionFxnII and get_table_objects_NA_corrected, which wrap IsoCorrectoR to correct isotopologue distributions and average labeling values) — https://github.com/SIMPELmetabolism/SIMPEL
- **IsoCorrectoR** (Underlying natural abundance correction engine invoked by NACorrectionFxnII on the unscaled MIDs table)
- **XCMS** (Pre-processing tool that produced the HRMS feature table imported into SIMPEL before correction)

## Examples

```
library(IsoCorrectoR); library(SIMPEL); corrected <- NACorrectionFxnII(MIDs_table, label_scheme, output = "NA_corrected_MIDs.csv")
```

## Evaluation signals

- Unlabeled (0 h) samples show isotopologue distributions with natural-abundance contributions removed after correction
- Corrected average labeling values increase monotonically over the feeding time course (0-8 h glutamine for Arabidopsis; 0-32 h glucose for Camelina)
- Corrected MIDs and average labeling qualitatively reproduce Fig. 1/2 time-course patterns of the paper
- For MS2 acyl-chain data, summed pair weights (M0/M6, M3/M3, M6/M0) conserve the precursor isotopologue M6 fraction
- Corrected output CSVs (e.g., NA_corrected_MIDs.csv, MIDs_table_NA_corrected.csv) match Supplementary Data 1 Tables 8-9 patterns

## Limitations

- No software versions reported for SIMPEL or IsoCorrectoR, so reproducibility may depend on matching package versions
- Correction accuracy depends on correct compound formulas and label scheme; the number of multivariate isotopologues grows exponentially with labeled atoms (I = product of (Ne+1)), which complicates dual-isotope corrections
- Only packages listed in Depends are auto-attached in R; IsoCorrectoR must be explicitly attached with library() before calling SIMPEL correction functions

## Evidence

- [intro] SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections: "SIMPEL automates post-processing data analyses of isotope enriched metabolomics datasets, generates isotopologue distributions, performs natural abundance corrections"
- [methods] NACorrectionFxnII signature: "NACorrectionFxnII(MIDs_table, label_scheme, output)"
- [intro] HRMS dual-label resolution rationale: "High resolution mass spectrometry (HRMS) can distinguish metabolites (and isotopologues of heteroatoms) through accurate measurement of m/z values"
- [intro] Isotopologue combinatorics: [evidence span withheld — anchor status: not_found]
