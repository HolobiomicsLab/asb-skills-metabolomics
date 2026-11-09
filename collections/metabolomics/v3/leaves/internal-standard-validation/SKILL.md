---
name: internal-standard-validation
description: Use when a GC-MS experiment includes known internal or external standards
  and automated component-area extraction must be verified for accuracy before compound
  quantification is trusted.
when_to_use_negative:
- The dataset contains no known internal or external standards, so extracted areas
  cannot be checked against injected amounts
- Data are MS2, MS3, or ion mobility + MS2 acquisitions — uafR's algorithms depend
  on published spectra and do not yet apply
- Data come from instrument types other than GC-MS (e.g., Q-TOF, Orbitrap, TimsTOF
  HT, Astral, or LC-MS), where uafR is untested
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: uafR
  role: Provides mzExacto() to extract standard compound areas per sample and standardifyIt()
    to standardize tentatively identified chemical areas against internal/external
    standards
  repo: github.com/castratton/uafR
- name: R
  role: Runtime language in which the standards validation workflow (mzExacto on standards,
    correlation of areas vs. injected volume) is executed
- name: devtools
  role: Installs uafR from GitHub prior to running the validation workflow
  repo: github.com/castratton/uafR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:57cccaad4b7ab2bd049aa14708f3c88891503b977c4559135171434807c4a3ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# internal-standard-validation

## Summary

Validates automated GC-MS compound extraction and area aggregation in uafR by running mzExacto()/standardifyIt() on known internal standards and confirming that extracted peak areas correlate with injected standard amounts (R2 ≈ 0.83–0.9999).

## When to use

Use when a GC-MS experiment includes known internal or external standards and automated component-area extraction must be verified for accuracy before compound quantification is trusted.

## When NOT to use

- The dataset contains no known internal or external standards, so extracted areas cannot be checked against injected amounts
- Data are MS2, MS3, or ion mobility + MS2 acquisitions — uafR's algorithms depend on published spectra and do not yet apply
- Data come from instrument types other than GC-MS (e.g., Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS), where uafR is untested

## Inputs

- GC-MS hit-table CSV from Agilent Unknowns Analysis with required columns: Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- spreadOut() output data frame (the 'spread' object) with retention-time windows per chemical
- Vector of known internal-standard chemical names (e.g., the four standards used in the validation run)
- Injected standard volumes/concentrations across the 1 × 10-5 to 1 × 103 ng/μl series

## Outputs

- Data frame of extracted compounds with optimal retention time, exact mass, best match factor, and aggregated component area per sample
- extracted_areas.csv containing aggregated areas per chemical per sample
- R2 correlation values between extracted peak areas and injected standard volumes
- Standardized component areas (via standardifyIt()) relative to the internal/external standard

## How to apply

Load the Agilent Unknowns Analysis hit-table CSV so it carries the spreadOut() contract columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and reject duplicate peak keys. Run mzExacto(spread, query_chemicals, decontaminate = TRUE) on the known internal standards only, confirming the output contains exactly the expected number of extracted-compound rows (e.g., four rows for four standards), each with optimal retention time, exact mass, and aggregated component area per sample. Write the aggregated areas to extracted_areas.csv. Then assess accuracy by correlating extracted peak areas with the injected standard volume across the concentration series (1 × 10-5 to 1 × 103 ng/μl); in the published validation, R2 values ranged from 0.8273 to 0.9998 (Fig 2). Finally, use standardifyIt() to express tentative compound areas relative to the validated standard(s), remembering that manual chemical/biological sense-checking of areas is still required.

## Related tools

- **uafR** (Provides mzExacto() to extract standard compound areas per sample and standardifyIt() to standardize tentatively identified chemical areas against internal/external standards) — github.com/castratton/uafR
- **R** (Runtime language in which the standards validation workflow (mzExacto on standards, correlation of areas vs. injected volume) is executed)
- **devtools** (Installs uafR from GitHub prior to running the validation workflow) — github.com/castratton/uafR

## Examples

```
input_spread = spreadOut(input_dat); input_exacto = mzExacto(input_spread, c("Linalool","Methyl Salicylate","Limonene","alpha-Thujene"), decontaminate = TRUE); write.csv(input_exacto, "extracted_areas.csv")
```

## Evaluation signals

- Running mzExacto() with the four known internal standards yields exactly four extracted-compound rows
- extracted_areas.csv is produced with aggregated areas per chemical per sample, with 0 substituted where a chemical is absent
- Extracted peak areas correlate with injected standard volume with R2 in the validated 0.8273–0.9998 range (Fig 2)
- With decontaminate = TRUE, chemicals lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES are removed
- Every output row carries optimal retention time, exact mass, and best match factor, consistent with the mzExacto() return contract

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or LC-MS data
- Algorithms do not yet apply to MS2, MS3, or ion mobility + MS2 data until spectra are published for most chemicals in PubChem's database

## Evidence

- [abstract] correctly identified the known compounds with R2 values ranging from 0.827-0.999 along concentrations ranging from 1 × 10-5 to 1 × 103 ng/μl: "correctly identified the known compounds with R2 values ranging from 0.827-0.999 along concentrations ranging from 1 × 10-5 to 1 × 103 ng/μl"
- [results] Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2): "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2)"
- [intro] standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s): "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [readme] extract target compounds with `mzExacto()`, and standardize abundance with `standardifyIt()`.: [evidence span withheld — anchor status: external_doc]
