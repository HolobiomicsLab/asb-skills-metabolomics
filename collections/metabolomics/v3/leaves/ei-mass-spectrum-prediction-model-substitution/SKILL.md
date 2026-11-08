---
name: ei-mass-spectrum-prediction-model-substitution
description: Use when a PS2MS-style synthetic NPS database already exists and you
  want to evaluate whether a different in silico EI spectrum predictor (e.g., RASSP)
  improves or degrades ranked identification of GC/EI-MS queries relative to the NEIMS
  baseline.
when_to_use_negative:
- The analyte spectrum already matches an established reference spectrum (NIST, Wiley,
  SWGDRUG) in conventional library screening — PS2MS-style prediction is only triggered
  when no reference match is found.
- The goal is de novo structure prediction of an NPS from its spectrum, rather than
  re-scoring and ranking an already-enumerated candidate subset with a substituted
  spectrum predictor.
- The EI spectra come from a source where no candidate SMILES set has been enumerated/filtered,
  so there is nothing to predict spectra for.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: RASSP
  role: Substituted EI mass spectrum predictor; predicts spectra for all candidates
    in the synthetic NPS subset and saves them in MSP format
- name: NEIMS
  role: Baseline spectrum predictor whose top-1/top-20 rates (>50% top-1 with Jaccard,
    >85% top-20 with SMSF) serve as the reference for evaluating the RASSP substitution
- name: DeepEI
  role: Converts SWGDRUG query spectra into molecular fingerprints that feed the SMSF
    weighted-sum ranking
- name: PS2MS
  role: Host pipeline providing the synthetic NPS database, enumeration/detection
    steps, and similarity scoring that the RASSP predictions slot into
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Molecular toolkit used alongside Python to handle candidate SMILES and fingerprints
    in the PS2MS workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:d360b916767622bfb3ef57435d126a950ebdb13089357f09376745d237466f88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - RASSP
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ei-mass-spectrum-prediction-model-substitution

## Summary

Substitutes an alternative EI-MS prediction model (RASSP) for NEIMS within the PS2MS matching pipeline to test how the choice of spectrum predictor affects candidate ranking. It re-runs cosine + Jaccard spectrum similarity and SMSF fingerprint-integrated ranking on SWGDRUG queries, then benchmarks top-1/top-20 rates against the NEIMS baseline.

## When to use

Use when a PS2MS-style synthetic NPS database already exists and you want to evaluate whether a different in silico EI spectrum predictor (e.g., RASSP) improves or degrades ranked identification of GC/EI-MS queries relative to the NEIMS baseline.

## When NOT to use

- The analyte spectrum already matches an established reference spectrum (NIST, Wiley, SWGDRUG) in conventional library screening — PS2MS-style prediction is only triggered when no reference match is found.
- The goal is de novo structure prediction of an NPS from its spectrum, rather than re-scoring and ranking an already-enumerated candidate subset with a substituted spectrum predictor.
- The EI spectra come from a source where no candidate SMILES set has been enumerated/filtered, so there is nothing to predict spectra for.

## Inputs

- Filtered derivative candidate subset (SMILES) from the synthetic NPS database
- SWGDRUG query mass spectra (library-format EI spectra)
- RASSP model (public GitHub release) for EI spectrum prediction
- DeepEI model for converting query spectra to molecular fingerprints
- NEIMS baseline predictions for the same candidates (for comparison)

## Outputs

- RASSP-predicted EI spectra for all candidates in MSP format
- Per-query ranked candidate list under the SMSF weighted-sum metric (cosine + Jaccard, ±1 Da tolerance)
- Top-1 and top-20 hit rates for RASSP vs NEIMS, e.g., rassp_vs_neims_topk_comparison.csv

## How to apply

Load the filtered derivative candidate subset (SMILES) from the synthetic NPS database and the SWGDRUG query mass spectra. Run RASSP over every candidate and write predictions in MSP format, exactly as the NEIMS baseline does, so the downstream matcher is unchanged. For each query, compute cosine similarity plus Jaccard similarity over top peaks with a ±1 Da tolerance for isotopic variance between predicted and query spectra; this tolerance is what lifted NEIMS top-1 rates from below 40% to more than 50%. Convert query spectra to fingerprints with DeepEI and combine spectrum and fingerprint similarities into the SMSF weighted-sum metric to rank candidates. Finally, evaluate top-1 and top-20 hit rates and tabulate RASSP vs NEIMS side by side (e.g., rassp_vs_neims_topk_comparison.csv), judging RASSP against the NEIMS reference points of >50% top-1 (with Jaccard) and >85% top-20 (with SMSF) on SWGDRUG.

## Related tools

- **RASSP** (Substituted EI mass spectrum predictor; predicts spectra for all candidates in the synthetic NPS subset and saves them in MSP format)
- **NEIMS** (Baseline spectrum predictor whose top-1/top-20 rates (>50% top-1 with Jaccard, >85% top-20 with SMSF) serve as the reference for evaluating the RASSP substitution)
- **DeepEI** (Converts SWGDRUG query spectra into molecular fingerprints that feed the SMSF weighted-sum ranking)
- **PS2MS** (Host pipeline providing the synthetic NPS database, enumeration/detection steps, and similarity scoring that the RASSP predictions slot into) — https://github.com/jhhung/PS2MS
- **rdkit** (Molecular toolkit used alongside Python to handle candidate SMILES and fingerprints in the PS2MS workflow)

## Examples

```
python run_matching.py --predictor RASSP --candidates candidates.msp --queries swgdrug.msp --similarity smsf --tolerance 1 --topk 20 --out rassp_vs_neims_topk_comparison.csv
```

## Evaluation signals

- The comparison table reports both top-1 and top-20 rates for RASSP and NEIMS on the SWGDRUG queries.
- The NEIMS baseline column reaches >50% top-1 after adding Jaccard similarity and >85% top-20 with the SMSF metric.
- Every candidate in the filtered subset has a RASSP-predicted spectrum present in the MSP output before matching begins.
- The ±1 Da peak-matching tolerance is consistently applied when computing Jaccard similarity so isotopic peaks are not spuriously mismatched.
- Rankings combine spectrum and fingerprint similarity via the SMSF weighted-sum metric, not spectrum similarity alone.

## Limitations

- No version numbers are stated for PS2MS, NEIMS, DeepEI, or RASSP in the source context, making exact reproduction of the comparison difficult.
- The NEIMS-based system was originally trained solely on NIST library spectra, which can bias predicted spectra against SWGDRUG-style queries; the same transfer concern applies to the substitute predictor.
- Additional postprocessing may be needed beyond the ranked list to determine precise identities, since ranked candidates are only suggestions for further examination.

## Evidence

- [methods] Method applies cosine + Jaccard with tolerance: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] Baseline top-20 performance on SWGDRUG: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [readme] NEIMS used as the spectrum predictor for the synthetic database: "PS<sup>2</sup>MS employs NEIMS to predict the mass spectrum of the compounds in synthetic database"
- [readme] Fingerprints are merged into the MSP file used by matching: "The system merges the fingerprint into the msp file."
- [intro] Integrated spectrum + fingerprint ranking: "The integrated similarity metric between the analyte and enumerated compounds in the NPS database based on the mass spectrum and fingerprint are calculated, summarized, and ranked."
