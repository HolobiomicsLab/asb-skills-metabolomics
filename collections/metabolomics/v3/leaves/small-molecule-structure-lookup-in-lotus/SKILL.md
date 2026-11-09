---
name: small-molecule-structure-lookup-in-lotus
description: Use when you have measured m/z mass features that need putative structure
  annotations, or a set of known substrate/product structures (e.g. the 374 characterized
  biosynthetic structures) whose presence/absence in LOTUS must be tabulated.
when_to_use_negative:
- When MS/MS spectra are available and spectral library/networking-based annotation
  is desired — the article notes no MS/MS loading or library-based annotation is currently
  supported
- When stereoisomer distinctions matter — mass spectrometric data does not capture
  stereoisomers, so LOTUS-based mass matching cannot resolve them
- When metabolites are not covered by LOTUS or the user lacks a user-defined metabolite
  database to substitute
edam_operation: http://edamontology.org/operation_0361
edam_topics: []
tools:
- name: MEANtools
  role: Performs the adduct mapping of mass features and the SQLite query against
    LOTUS; provides the bundled SQLite-converted LOTUS database
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: LOTUS
  role: Reference metabolite-structure database queried to assign structures or test
    structure presence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:12b3421ddda3201dc9d7256efa1a2a18dd4ef325cba58ecdcd600bc53ded5781
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LOTUS
  - MEANtools
  edam_operation: http://edamontology.org/operation_0361
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# small-molecule-structure-lookup-in-lotus

## Summary

Assign candidate molecular structures to mass features (or check presence of known structures) by enumerating plausible adducts and querying the LOTUS metabolite database distributed with MEANtools as a pre-converted SQLite file (downloaded 10/10/2023), at a default 20 ppm mass-accuracy tolerance.

## When to use

Use when you have measured m/z mass features that need putative structure annotations, or a set of known substrate/product structures (e.g. the 374 characterized biosynthetic structures) whose presence/absence in LOTUS must be tabulated.

## When NOT to use

- When MS/MS spectra are available and spectral library/networking-based annotation is desired — the article notes no MS/MS loading or library-based annotation is currently supported
- When stereoisomer distinctions matter — mass spectrometric data does not capture stereoisomers, so LOTUS-based mass matching cannot resolve them
- When metabolites are not covered by LOTUS or the user lacks a user-defined metabolite database to substitute

## Inputs

- Metabolomics mass-feature list (m/z values) for adduct-based structure prediction
- List of expected adducts used to map mass signatures to neutral masses
- Set of known small-molecule structures/structure IDs to test for presence in LOTUS (e.g. the 374 substrate/product structures from Combined_small_middle_big_datasets.csv, Zenodo record 15697913)
- LOTUS SQLite database bundled in the MEANtools GitHub repository (version downloaded 10/10/2023)

## Outputs

- Per-structure matched/unmatched table for each queried structure ID (e.g. structure_presence_chisquare.csv)
- Putative metabolite structures assigned to mass features with their corresponding molecular structures
- Significance result of the overlap (chi-square statistic, degrees of freedom, p-value)

## How to apply

Load the LOTUS database in the SQLite format shipped in the MEANtools GitHub repository (version downloaded 10/10/2023); no re-download or format conversion is needed since RetroRules and LOTUS are already provided as SQLite for direct use with MEANtools. For metabolome annotation, map each mass signature with a list of expected adducts, compute the neutral-mass candidates, and query LOTUS within the default 20 ppm tolerance, which balances sensitivity and specificity. For database-overlap assessment (as in the validated task), match each of the 374 substrate/product structures from Combined_small_middle_big_datasets.csv (Zenodo record 15697913) against LOTUS structure entries, recording matched/unmatched per structure ID; the expected outcome is 132 of 374 present, with a chi-square of 32.353 (DF=1, p<0.001). Write the per-structure match table plus significance result (e.g. structure_presence_chisquare.csv) so the overlap claim is auditable.

## Related tools

- **MEANtools** (Performs the adduct mapping of mass features and the SQLite query against LOTUS; provides the bundled SQLite-converted LOTUS database) — github.com/IntegrativeBioinformaticsLab/meantools
- **LOTUS** (Reference metabolite-structure database queried to assign structures or test structure presence)

## Evaluation signals

- Match count against the known benchmark: 132 of 374 characterized substrate/product structures found in LOTUS
- Statistical check: chi-square = 32.353 with DF=1 and p < 0.001 on the presence/absence contingency
- Every mass-feature annotation must be traceable to an adduct mapping and a mass difference within the 20 ppm tolerance
- Output table contains one matched/unmatched record per input structure ID with no unmatched IDs left unreported

## Limitations

- Default 20 ppm tolerance trades sensitivity against specificity; mass-spectrometer resolution directly impacts the precision of detected m/z values and the confidence of adduct-based matching
- Mass spectrometric data does not capture stereoisomers, so structurally distinct isomers cannot be distinguished by this lookup
- The LOTUS snapshot is fixed (downloaded 10/10/2023); a user-defined metabolite database is needed for compounds outside LOTUS

## Evidence

- [discussion] adduct mapping then LOTUS query: "It assigns molecular structures to each mass feature of the metabolome by mapping them with a list of adducts and then querying LOTUS database (downloaded on 10/10/2023)."
- [discussion] SQLite availability: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
- [discussion] 20 ppm tolerance: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [supplementary] characterized-structures input file: "can be found at https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv"
