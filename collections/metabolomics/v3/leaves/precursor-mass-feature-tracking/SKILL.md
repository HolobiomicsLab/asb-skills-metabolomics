---
name: precursor-mass-feature-tracking
description: Use when an unannotated molecular feature defined by precursor m/z (e.g.,
  m/z 266.138, 280.153, 294.169) or an MS/MS spectrum must be searched across tissues
  or mapped from an animal model into public human metabolomics studies.
when_to_use_negative:
- The input is a processed feature table with no MS/MS peak lists — tissueMASST requires
  precursor mass plus fragment spectra to search.
- A fully validated (MSI Level 1) structural identification is required; spectral
  matching yields Level 2 or 3 annotations only.
- The spectrum was acquired with poor fragmentation or missing fragments, so reliable
  network edges and library matches cannot be formed.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: tissueMASST
  role: 'Primary search engine: queries precursor masses and MS/MS spectra of features
    across public metabolomics repositories grouped by tissue to bridge animal-model
    findings with human data.'
- name: GNPS
  role: Reference spectral libraries used for MS/MS spectral matching that assigns
    Level 2/3 annotations and supports the cross-tissue molecular network from which
    features are exported.
- name: SIRIUS
  role: Compound-class and formula prediction module (CSI:FingerID) applied to exported
    consensus spectra of unannotated features, e.g., classifying m/z 266.138, 280.153,
    and 294.169 as fatty-acyl carnitines.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:ec90cde177614df8c8c54dbf17b50aaced19a5a234c12ba02424e038568cdc87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - SIRIUS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# precursor-mass-feature-tracking

## Summary

Tracks an unknown or annotated molecular feature by its precursor m/z and MS/MS spectrum across tissues and public metabolomics repositories using tissueMASST, bridging mouse-model findings (e.g., microbially-modulated carnitines in the 3xTg atlas) to human AD datasets.

## When to use

Use when an unannotated molecular feature defined by precursor m/z (e.g., m/z 266.138, 280.153, 294.169) or an MS/MS spectrum must be searched across tissues or mapped from an animal model into public human metabolomics studies.

## When NOT to use

- The input is a processed feature table with no MS/MS peak lists — tissueMASST requires precursor mass plus fragment spectra to search.
- A fully validated (MSI Level 1) structural identification is required; spectral matching yields Level 2 or 3 annotations only.
- The spectrum was acquired with poor fragmentation or missing fragments, so reliable network edges and library matches cannot be formed.

## Inputs

- Consensus MS/MS spectrum for a molecular feature (precursor m/z + peak list, .mgf or .ms format)
- Molecular-network membership (e.g., carnitine subnetwork in the cross-tissue 3xTg/5xFAD network)
- Tissue source of the feature (brain, serum, liver, cecal content, feces)

## Outputs

- Tissue-level and study-level matches of the feature across public metabolomics repositories
- Putative compound-class assignment (e.g., fatty-acyl carnitine via SIRIUS/CSI:FingerID) with top molecular formula
- Mapping table linking the feature to human cohorts with associated phenotypes (e.g., aging, cognitive impairment)

## How to apply

Export the consensus MS/MS spectrum for the feature of interest (precursor mass plus peak list, e.g., as .mgf or individual .ms files from the cross-tissue molecular network) and search it with tissueMASST against public metabolomics repositories grouped by tissue type. Tissue-level hits supply evidence of where the feature occurs in public data; interpretation should respect that non-detection is not evidence of absence because study-specific extraction methods, workflows, and instrumentation influence what is detected. To assign identity to unannotated spectra that fall in a molecular-network subnetwork (e.g., the carnitine subnetwork), run SIRIUS with the CSI:FingerID compound-class module on the exported spectra and record top formula and top compound class (e.g., fatty-acyl carnitine). Combine tissueMASST hits with differential-abundance context from the source model — for example, phenylacetyl-carnitine mapped to human AD cohorts was positively associated with aging and cognitive impairment. Annotation confidence via GNPS library spectral matching should be reported as Metabolomics Standards Initiative Level 2 or 3, not fully validated structure.

## Related tools

- **tissueMASST** (Primary search engine: queries precursor masses and MS/MS spectra of features across public metabolomics repositories grouped by tissue to bridge animal-model findings with human data.)
- **GNPS** (Reference spectral libraries used for MS/MS spectral matching that assigns Level 2/3 annotations and supports the cross-tissue molecular network from which features are exported.)
- **SIRIUS** (Compound-class and formula prediction module (CSI:FingerID) applied to exported consensus spectra of unannotated features, e.g., classifying m/z 266.138, 280.153, and 294.169 as fatty-acyl carnitines.)

## Evaluation signals

- SIRIUS compound-class output for the searched spectra matches the expected class (e.g., fatty-acyl carnitine for m/z 266.138, 280.153, 294.169).
- tissueMASST returns hits in public datasets containing the queried precursor mass, and hits are interpretable at the tissue level.
- Search result is consistent with the source molecular network — the feature should map into the same subnetwork (e.g., carnitine subnetwork).
- Annotations obtained are reported at MSI Level 2 or 3, not overstated as fully validated structures.
- Human-cohort mapping reproduces the expected direction of association (e.g., phenylacetyl-carnitine positively associated with aging and cognitive impairment).

## Limitations

- Feature detection across tissueMASST reference datasets is confounded by study-specific extraction methods, workflows, and instrumentation; lack of detection does not mean absence in a tissue.
- GNPS library spectral matching yields only Level 2 or 3 Metabolomics Standards Initiative annotations.
- Some spectral pairs are missing from the cross-tissue molecular network due to poor fragmentation or missing fragments during MS/MS acquisition.
- Hundreds of differentially abundant molecules remain non-annotated, so many searched features will return no structural assignment.

## Evidence

- [abstract] Molecular feature tracking via tissueMASST, a mass spectrometry search tool we developed to bridge animal model findings with human data: "Molecular feature tracking via tissueMASST, a mass spectrometry search tool we developed to bridge animal model findings with human data"
- [intro] tissueMASST open-source MS/MS search across repositories: "we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories"
- [discussion] Non-detection caveat for tissueMASST: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation."
- [discussion] Annotation level via GNPS matching: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation"
- [abstract] tissueMASST identifies microbially-modulated phenylacetyl-carnitine as positively associated with aging and cognitive impairment across human AD studies: "identifies microbially-modulated phenylacetyl-carnitine as positively associated with aging and cognitive impairment across human AD studies"
