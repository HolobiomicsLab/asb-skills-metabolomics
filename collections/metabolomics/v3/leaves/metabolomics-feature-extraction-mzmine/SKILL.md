---
name: metabolomics-feature-extraction-mzmine
description: Use when raw LC-MS/MS files (e.g., MassIVE accession MSV000080673) must
  be converted into a per-sample feature table with extracted MS/MS spectra before
  spectral library matching of drugs or metabolites.
when_to_use_negative:
- Input is already a processed feature table or .mgf file from a prior MZmine run
  — go directly to library spectral matching instead
- The goal is clinical decision-making on individual drug exposure — the GNPS Drug
  Library readout is explicitly not intended for immediate clinical decisions
- Only the parent drug is expected and metabolized drug versions matter — existing
  MS/MS reference libraries capture only the parent drug for most compounds, so parent-only
  matching will miss exposures
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Feature detection, deconvolution, alignment, and MS/MS spectral extraction
    from raw LC-MS/MS files to build the consensus feature table
  repo: https://github.com/mzmine/mzmine
- name: GNPS Drug Library
  role: Downstream consumer of the extracted MS/MS spectra; spectral matching annotates
    features with drug identities and pharmacologic class metadata
- name: MassIVE
  role: Public repository from which the raw LC-MS/MS files (e.g., MSV000080673) are
    downloaded
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:76a8f3ee10af3fbbf9f764e40d68e9a7cd362454424065c8d9701ef3f80f0ca0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS Drug Library
  - MZmine
  - MassIVE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-feature-extraction-mzmine

## Summary

Extract, deconvolve, and align MS/LS-MS/MS features from raw untargeted metabolomics files using MZmine, producing a consensus feature table and extracted MS/MS spectra that can be submitted to the GNPS Drug Library for drug exposure readout.

## When to use

Use when raw LC-MS/MS files (e.g., MassIVE accession MSV000080673) must be converted into a per-sample feature table with extracted MS/MS spectra before spectral library matching of drugs or metabolites.

## When NOT to use

- Input is already a processed feature table or .mgf file from a prior MZmine run — go directly to library spectral matching instead
- The goal is clinical decision-making on individual drug exposure — the GNPS Drug Library readout is explicitly not intended for immediate clinical decisions
- Only the parent drug is expected and metabolized drug versions matter — existing MS/MS reference libraries capture only the parent drug for most compounds, so parent-only matching will miss exposures

## Inputs

- Raw LC-MS/MS data files downloaded from the MassIVE repository (e.g., MSV000080673, American Gut Project, 1993 individuals)
- MZmine batch/processing parameter configuration adjusted for the research question (m/z tolerance, deconvolution and alignment settings)

## Outputs

- Consensus feature table (aligned features with per-individual intensities across the cohort)
- Extracted/deconvoluted MS/MS spectra (e.g., .mgf) ready for submission to the GNPS Drug Library for spectral matching
- Annotated feature table with drug identities, metabolites, and pharmacologic class metadata after library matching

## How to apply

Download the raw LC-MS/MS files from the MassIVE repository (e.g., MSV000080673 from the American Gut Project cohort of 1993 individuals) and load them into MZmine. Run MZmine feature detection, peak deconvolution, and alignment across samples to build a single consensus feature table spanning the full cohort. Export the deconvoluted MS/MS spectra (e.g., as an .mgf file compatible with the GNPS molecular networking workflow) alongside the feature quantification table. Submit the extracted MS/MS spectra to the target library — here the GNPS Drug Library, which contains MS/MS reference spectra for drugs and their metabolites/analogs — for spectral matching and pharmacologic-class annotation. Search parameters such as m/z tolerance and minimum matching peaks must be checked and adjusted based on the research question before running downstream batch searches.

## Related tools

- **MZmine** (Feature detection, deconvolution, alignment, and MS/MS spectral extraction from raw LC-MS/MS files to build the consensus feature table) — https://github.com/mzmine/mzmine
- **GNPS Drug Library** (Downstream consumer of the extracted MS/MS spectra; spectral matching annotates features with drug identities and pharmacologic class metadata)
- **MassIVE** (Public repository from which the raw LC-MS/MS files (e.g., MSV000080673) are downloaded)

## Evaluation signals

- The consensus feature table spans all 1993 individuals in the cohort (no samples dropped during alignment)
- Downstream GNPS Drug Library matching detects 89 drugs whose pharmacologic classes match common prescriptions
- Reproduced demographic association matches the reference: NSAIDs/analgesics significantly more frequent in females (chi-square p = 8.5e-05)
- Exported MS/MS spectra are in a format accepted by the GNPS workflow (e.g., .mgf from MZmine molecular networking)

## Limitations

- Even with correct feature extraction, only metabolized versions of a drug present in a sample can lead to missed drug exposure if only the parent drug is considered, since existing MS/MS reference libraries capture mostly parent drugs
- Search parameters (m/z tolerance, minimum matching peaks) must be manually checked and adjusted per research question; automated defaults may not suit the dataset
- Drug annotations from spectral matching are research-grade empirical readouts, not clinical-grade exposure records

## Evidence

- [other] workflow card, American Gut analysis: [evidence span withheld — anchor status: not_found]
- [other] workflow card, MZmine processing steps: [evidence span withheld — anchor status: not_found]
- [readme] README, microbe_masst: [evidence span withheld — anchor status: external_doc]
- [intro] article intro: "GNPS Drug Library contains a collection of MS/MS reference spectra for drugs and their metabolites/analogs along with standardized, machine-readable pharmacologic metadata"
