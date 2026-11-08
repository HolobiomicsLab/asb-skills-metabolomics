---
name: pretrained-deep-neural-network-inference-msms
description: Use when an unknown metabolite's MS/MS spectrum (e.g., from Shimadzu
  LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) must be matched to candidate structures
  in a known-spectra database rather than via direct library match.
when_to_use_negative:
- When the target metabolite has no structure-related metabolite available in the
  database — DeepMASS identification is valid only if a structure-related metabolite
  is present
- When a direct spectral library match exists for the unknown, since direct database
  matching does not need similarity scoring
- When you need to process the removed MetDNA experimental dataset directly — the
  repo no longer ships those spectra and cannot be run out of the box
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: DeepMASS
  role: Provides the pretrained deep-neural-network structural-similarity scorer and
    candidate ranking workflow used to identify unknowns from MS/MS spectra
  repo: https://github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that acquired the external-test MS/MS biomarker spectra to be identified
    by inference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:1453fb2c32f8cdab74d1b03ea09abafc93740ad81aec524a4879d64fe60e6e9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pretrained-deep-neural-network-inference-msms

## Summary

Run the trained DeepMASS deep-neural-network scorer to predict structural similarity between unknown MS/MS metabolite spectra and known database compounds (e.g., 662 KEGG compounds), then rank candidates so unknowns are identified through structurally related known metabolites.

## When to use

Use when an unknown metabolite's MS/MS spectrum (e.g., from Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) must be matched to candidate structures in a known-spectra database rather than via direct library match.

## When NOT to use

- When the target metabolite has no structure-related metabolite available in the database — DeepMASS identification is valid only if a structure-related metabolite is present
- When a direct spectral library match exists for the unknown, since direct database matching does not need similarity scoring
- When you need to process the removed MetDNA experimental dataset directly — the repo no longer ships those spectra and cannot be run out of the box

## Inputs

- Unknown-metabolite MS/MS spectra files placed in data/spectra (external test spectra, e.g., male-infertility biomarkers from Shimadzu LC-ESI-IT-TOF / Bruker Compact LC-ESI-QTOF)
- Pretrained DeepMASS model (model-40V.zip, unzipped into /model/40V)
- KEGG reference library of MS/MS spectra for 662 compounds defining the search space

## Outputs

- Per-spectrum ranking of KEGG database metabolites by DeepMASS structural-similarity score
- Top-hit identification labels indicating whether the structure-related metabolite is recovered
- Identification results table (e.g., external_shimadzu_identification.csv)

## How to apply

Download the pretrained model (model-40V.zip) and unzip it into the /model/40V directory of the DeepMASS repo, then place the unknown MS/MS spectra files into data/spectra. Run inference (example.py) so the network scores structural similarity of each unknown spectrum against every metabolite in the KEGG reference library of 662 compounds. Rank the database metabolites by DeepMASS score per spectrum and take the top hit as the putative identification. Judge the identification valid only when the structure-related (true) metabolite is available in and recovered from the database, as in the male-infertility external test on the Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF. The rationale is that structurally related metabolites often have related MS/MS spectra, so similarity scoring extends the identification range of existing MS/MS databases.

## Related tools

- **DeepMASS** (Provides the pretrained deep-neural-network structural-similarity scorer and candidate ranking workflow used to identify unknowns from MS/MS spectra) — https://github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument that acquired the external-test MS/MS biomarker spectra to be identified by inference)

## Examples

```
git clone https://github.com/hcji/DeepMASS; unzip model-40V.zip -d model/40V; cp spectra/* data/spectra/; python example.py
```

## Evaluation signals

- The structure-related (true) metabolite is recovered as the top-ranked hit for each external biomarker spectrum
- The result table reproduces the reported external identification performance (metric_external_shimadzu_identification)
- Leave-one-out cross-validation on the 662 KEGG compounds matches the paper's reported evaluation
- Identifications are counted valid only when the structure-related metabolite is available in and recovered from the database
- Model files are correctly unzipped into /model/40V and spectra resolve from data/spectra before inference

## Limitations

- The MetDNA experimental spectra dataset was removed from the repository (2019/04/29), so the package cannot be run directly; users must train their own model with an in-house database or wait for public-database spectra
- Identification succeeds only when a structure-related metabolite is present in the database
- Reproducing the paper's results requires the original dataset, which is restricted to that purpose by its owner

## Evidence

- [readme] Entry point for identification: [evidence span withheld — anchor status: external_doc]
- [readme] Model installation path: [evidence span withheld — anchor status: external_doc]
- [readme] Dataset removal caveat: [evidence span withheld — anchor status: external_doc]
