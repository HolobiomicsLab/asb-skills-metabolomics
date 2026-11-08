---
name: msms-fragment-screening
description: Use when a blank-filtered LC-HRMS/MS feature table (e.g. AFFF samples)
  needs PFAS candidate flagging via MS/MS fragment matching, especially for precursor
  ions whose exact masses match nothing in the EPA catalogued PFAS list.
when_to_use_negative:
- Precursor ions already have exact-mass matches to catalogued PFAS in the EPA CompTox
  list (>8000 structures) — exact mass matching plus homologous-series detection are
  the primary evidence there.
- Data are MS1-only with no MS/MS spectra acquired — fragment screening cannot run
  without experimental tandem MS.
- The feature table has not been blank-filtered yet — run feature blank filtering
  against the solvent blanks first (3740 of 31,132 features in the AFFF example).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: FluoroMatch 2.0
  role: Automated non-targeted PFAS annotation pipeline of which fragment screening
    is one evidence layer feeding the confidence-level scoring
  repo: https://github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Module hosting the reverse fragment-screening step that screens precursor
    MS/MS against the 777-fluorine-fragment library
  repo: https://github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: End-to-end workflow (feature detection through scoring) that chains fragment
    screening with class-based MS/MS screening and predicted MS/MS
  repo: https://github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:d3b947217b955a7b399572bcc455b17d84617b89d75180ba7f29191e0f3ff11c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/InnovativeOmics/Core-Match
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  - LipidMatch
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# msms-fragment-screening

## Summary

Reverse MS/MS fragment screening that matches experimental precursor MS/MS spectra against a library of 777 fluorine-containing PFAS fragments to assign PFAS substructures and flag potential PFAS, including features with no exact-mass match to catalogued PFAS.

## When to use

Use when a blank-filtered LC-HRMS/MS feature table (e.g. AFFF samples) needs PFAS candidate flagging via MS/MS fragment matching, especially for precursor ions whose exact masses match nothing in the EPA catalogued PFAS list.

## When NOT to use

- Precursor ions already have exact-mass matches to catalogued PFAS in the EPA CompTox list (>8000 structures) — exact mass matching plus homologous-series detection are the primary evidence there.
- Data are MS1-only with no MS/MS spectra acquired — fragment screening cannot run without experimental tandem MS.
- The feature table has not been blank-filtered yet — run feature blank filtering against the solvent blanks first (3740 of 31,132 features in the AFFF example).

## Inputs

- Blank-filtered feature table from LC-HRMS/MS (vendor files from the AFFF sample and three solvent blanks)
- Experimental MS/MS spectra for each precursor ion (m/z 117-1189)
- Reconstructed library of 777 fluorine-containing PFAS fragments (task_005.expected_outputs[0])

## Outputs

- fragment_assignments.csv — per-feature table of assigned fluorine-containing fragments and PFAS substructures
- PFAS candidate flags (features with [SF5]-/[SO3F]- diagnostic fragments as unsaturated-PFAS candidates)

## How to apply

Load the blank-filtered feature table (produced upstream by feature detection and blank filtering, e.g. 3740 features from 31,132 in the legacy AFFF 1999 holding-tank sample) into the FluoroMatch Modular fragment-screening module, then load the reconstructed library of 777 fluorine-containing PFAS fragments. Restrict precursors to the PFAS mass-defect window of -0.116 to 0.268 (covering 98% of the EPA PFAS list) and m/z 117-1189, and reverse-screen each precursor's MS/MS against the fragment library to assign diagnostic fragments and PFAS substructures to each feature. Flag features carrying pentafluorosulfide ([SF5]-) and sulfonate ([SO3F]-) diagnostic fragments as unsaturated-PFAS candidates, such as the C8 AFFF candidate of a proposed new PFAS class with a pentafluorosulfide and sulfonic acid terminus. Write per-feature fragment/substructure assignments to fragment_assignments.csv, and let the assignments feed the comprehensive confidence-level scoring framework that integrates fragment evidence with exact mass, mass-defect, homologous-series (CF2), retention-time and predicted-MS/MS layers. This layer of evidence is what elevates features from 'potential' to 'probable' PFAS in the 96% feature-reduction cascade.

## Related tools

- **FluoroMatch 2.0** (Automated non-targeted PFAS annotation pipeline of which fragment screening is one evidence layer feeding the confidence-level scoring) — https://github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Module hosting the reverse fragment-screening step that screens precursor MS/MS against the 777-fluorine-fragment library) — https://github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (End-to-end workflow (feature detection through scoring) that chains fragment screening with class-based MS/MS screening and predicted MS/MS) — https://github.com/InnovativeOmics/Core-Match

## Examples

```
git clone https://github.com/InnovativeOmics/Core-Match; cp Scripts/* RapidTest/LipidMatch_Libraries/Scripts/
```

## Evaluation signals

- fragment_assignments.csv exists and parses as a well-formed table with one row per blank-filtered feature.
- The C8 candidate row contains [SF5]- and [SO3F]- fragment assignments (chemical plausibility left to expert review).
- Screened precursors fall within the PFAS mass-defect window (-0.116 to 0.268) and m/z 117-1189; features outside are not fragment-screened.
- Fragment-assigned features contribute to the probable-PFAS tier consistent with the published cascade (31132 -> 3740 blank-filtered -> 2290 potential -> 1238 probable PFAS, 39% scored E).
- Each assigned fragment maps to a named PFAS substructure, not just a bare m/z hit.

## Limitations

- Fragment screening assigns substructures, not full structures; chemical plausibility of novel classes (e.g. the proposed pentafluorosulfide/sulfonic-acid-terminated AFFF PFAS) requires expert review.
- Coverage depends on the 777-fragment library and the -0.116 to 0.268 mass-defect window, which covers only 98% of the EPA PFAS list — atypical PFAS outside the window will be missed.
- Evidence from fragments alone is insufficient; the confidence-level framework integrates multiple layers, and overreporting occurs if fragment hits are reported without corroboration.

## Evidence
