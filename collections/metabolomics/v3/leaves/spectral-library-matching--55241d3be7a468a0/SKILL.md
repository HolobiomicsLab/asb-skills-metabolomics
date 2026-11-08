---
name: spectral-library-matching
description: Use when a GC/EI-MS (70 eV) analyte spectrum fails to match any known
  reference spectra (NIST, Wiley, or SWGDRUG libraries) and you need to rank potential
  NPS identities by combined spectral and fingerprint similarity against a synthetic
  derivative database.
when_to_use_negative:
- When the analyte's mass spectrum already matches a known reference spectrum — conventional
  library matching against NIST/Wiley/SWGDRUG is sufficient and cheaper.
- When the chemical structure of the NPS is already known and you only need its predicted
  spectrum — use NEIMS directly rather than the fingerprint-augmented matching workflow.
- When analyte spectra come from ionization methods other than 70 eV EI (e.g., LC-MS/MS),
  since the cosine and Jaccard metrics and the DeepEI fingerprint model were validated
  on EI spectra.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PS2MS
  role: Orchestrating system that computes the integrated SMSF score between the unknown
    analyte and each database candidate, then ranks candidates by similarity score.
  repo: https://github.com/jhhung/PS2MS
- name: DeepEI
  role: Converts the unknown analyte's mass spectrum into its corresponding molecular
    fingerprint (without explicit structure information) and computes fingerprints
    for database compounds, which are merged into the MSP file.
- name: NEIMS
  role: Predicts the EI mass spectrum of every compound in the synthetic NPS database
    so that spectra can be compared against the unknown analyte.
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:77b672619361da877e944ce90cb1792c61cae90ecf3b801fdc0ff1a161bdab38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - python
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# spectral-library-matching

## Summary

Ranks candidate compound identities for an unknown GC/EI-MS analyte by computing an integrated SMSF similarity score — a weighted sum of mass-spectrum cosine similarity and molecular-fingerprint Jaccard similarity — against a reference or synthetic NPS database, leveraging DeepEI-predicted fingerprints to supplement spectral matching when the analyte structure is unknown.

## When to use

Use when a GC/EI-MS (70 eV) analyte spectrum fails to match any known reference spectra (NIST, Wiley, or SWGDRUG libraries) and you need to rank potential NPS identities by combined spectral and fingerprint similarity against a synthetic derivative database.

## When NOT to use

- When the analyte's mass spectrum already matches a known reference spectrum — conventional library matching against NIST/Wiley/SWGDRUG is sufficient and cheaper.
- When the chemical structure of the NPS is already known and you only need its predicted spectrum — use NEIMS directly rather than the fingerprint-augmented matching workflow.
- When analyte spectra come from ionization methods other than 70 eV EI (e.g., LC-MS/MS), since the cosine and Jaccard metrics and the DeepEI fingerprint model were validated on EI spectra.

## Inputs

- Unknown analyte EI mass spectrum from GC/EI-MS (70 eV electron energy)
- NEIMS-predicted EI mass spectra of compounds in the reference or synthetic NPS database
- Molecular fingerprints of unknown analyte, predicted via DeepEI from its mass spectrum
- Reference spectrum library (e.g., NIST, Wiley, or SWGDRUG MS library) or enumerated synthetic NPS database in MSP format

## Outputs

- Ranked list of candidate NPS identities per unknown analyte, ordered by SMSF score
- Per-analyte table of cosine similarity, Jaccard similarity, and combined SMSF scores
- Top-1 and top-20 recovery statistics for a labeled test set (e.g., SWGDRUG substances)

## How to apply

Load the analyte's EI mass spectrum (from GC/EI-MS at 70 eV) and the NEIMS-predicted EI mass spectra of candidates in the NPS database (e.g., the synthetic derivative database built by substituting hydrogens on a cathinone core). Use the DeepEI model to convert each analyte's mass spectrum into its corresponding molecular fingerprint without explicit structure information. Compute cosine similarity between analyte and candidate spectra (with a ±1 Da mass tolerance, as used by PS2MS) and Jaccard similarity on the overlap of top-intensity peaks' fingerprints. Combine these as a weighted-sum SMSF score for every analyte-candidate pair, then rank all candidates per analyte and record the correct match's rank. The Jaccard component is critical: incorporating it raises top-1 correct-prediction proportion from below 40% to above 50%. Validate by confirming that over 85% of SWGDRUG test substances (or over 90% for NIST) rank within the top 20 predictions.

## Related tools

- **PS2MS** (Orchestrating system that computes the integrated SMSF score between the unknown analyte and each database candidate, then ranks candidates by similarity score.) — https://github.com/jhhung/PS2MS
- **DeepEI** (Converts the unknown analyte's mass spectrum into its corresponding molecular fingerprint (without explicit structure information) and computes fingerprints for database compounds, which are merged into the MSP file.)
- **NEIMS** (Predicts the EI mass spectrum of every compound in the synthetic NPS database so that spectra can be compared against the unknown analyte.)

## Examples

```
python drug-detection/run_detection.py --analyte unknown.msp --database synthetic_nps.msp --output ranked_matches.csv
```

## Evaluation signals

- Verify that the top-20 recovery proportion on the SWGDRUG test set exceeds 85% (reported value) when using the SMSF metric against the synthetic NPS database.
- Confirm that the cosine-only top-1 correct-prediction proportion falls below 0.40 and the cosine+Jaccard integrated metric exceeds 0.50, reproducing the reported improvement.
- Check that every analyte-candidate pair has both a cosine spectral similarity and a Jaccard fingerprint similarity before combining them into the SMSF score.
- Ensure the ±1 Da mass tolerance is applied consistently between analyte and candidate spectra during peak matching.
- The output ranking table should contain one row per candidate with a similarity score and a rank for the correct identification.

## Limitations

- Existing in silico EI mass spectra prediction methods (e.g., NEIMS) are limited to cases where the chemical structure of the NPS is already known; this skill therefore depends on a prebuilt synthetic database derived from enumerated derivatives of known cores.
- The number of enumeration levels used to build the synthetic database affects results, and the article does not specify the exact count used.
- Additional postprocessing and anticipated additional features may be needed to further enhance prediction accuracy and determine precise identities beyond the ranked list.

## Evidence

- [abstract] PS2MS simultaneously examines the chemical fingerprint and mass spectrum against the putative NPS database using integrated: "When the mass spectrum of an analyte does not match any known reference, PS2MS simultaneously examines the chemical fingerprint and mass spectrum against the putative NPS database using integrated"
- [methods] The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity.: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [readme] The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score.: "The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score."
- [methods] over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [intro] The spectra are initially compared with reference spectrum libraries such as the National Institute of Standards and Technology (NIST),6 Wiley,7 and SWGDRUG8 MS libraries: "The spectra are initially compared with reference spectrum libraries such as the National Institute of Standards and Technology (NIST),6 Wiley,7 and SWGDRUG8 MS libraries"
- [readme] PS2MS employs NEIMS to predict the mass spectrum of the compounds in synthetic database: "PS2MS employs NEIMS to predict the mass spectrum of the compounds in synthetic database"
