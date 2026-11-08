---
name: peptide-psm-filtering-from-fragpipe-output
description: Use when MSFragger/FragPipe PSM tables (searched at 1% protein FDR) must
  be filtered to confident peptide/protein sets before generating MALDI-MS peptide
  images in MSIght.
when_to_use_negative:
- Input is already a filtered, protein-level feature table rather than raw PSM-level
  FragPipe output
- The goal is MSI dataset pre-processing, histology co-registration, or segmentation
  (use MSIght's cluster_msi / registration workflows instead)
- Peptide FDR or PSM-level FDR below 1% protein FDR has not been established upstream,
  since the cascade presumes 1% protein FDR identifications
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MSIght
  role: Platform whose LC-MS/MS workflow applies the unique-peptide, unique-monoisotopic-mass,
    and >=5-peptide-per-protein filters to MSFragger PSM output
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Language in which MSIght and the filtering cascade are implemented and distributed
- name: Jupyter
  role: Notebook application of MSIght that explains the analysis pipeline step-by-step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:416b05e19978009cabd0dd362f8bd585b96b3fee3f8009201a5f26a9ae4d0f6e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# peptide-psm-filtering-from-fragpipe-output

## Summary

Applies MSIght's successive LC-MS/MS PSM filtering cascade (unique peptides, unique monoisotopic mass within 0.02 Da, and >=5 peptides per protein at 1% protein FDR) to MSFragger/FragPipe PSM output so that only confidently identified peptides are used for MALDI-MS image annotation.

## When to use

Use when MSFragger/FragPipe PSM tables (searched at 1% protein FDR) must be filtered to confident peptide/protein sets before generating MALDI-MS peptide images in MSIght.

## When NOT to use

- Input is already a filtered, protein-level feature table rather than raw PSM-level FragPipe output
- The goal is MSI dataset pre-processing, histology co-registration, or segmentation (use MSIght's cluster_msi / registration workflows instead)
- Peptide FDR or PSM-level FDR below 1% protein FDR has not been established upstream, since the cascade presumes 1% protein FDR identifications

## Inputs

- MSFragger/FragPipe PSM output table from deposited LC-MS/MS results (searched at 1% protein FDR, e.g., against the Mus musculus proteome)
- Protein accession and monoisotopic mass annotations per PSM

## Outputs

- Stepwise filtered peptide/protein table (filtered_protein_results.csv) after each successive filter
- Count summary of peptides and proteins remaining after the unique-peptide, unique-monoisotopic-mass, and >=5-peptides-per-protein filters

## How to apply

Load the MSFragger/FragPipe PSM output table (e.g., searched against the Mus musculus proteome at 1% protein FDR) from the deposited LC-MS/MS results. First filter to PSMs with unique peptide sequences, removing peptides mapping to multiple proteins. Next apply MSIght's monoisotopic mass uniqueness filter, requiring each peptide's monoisotopic mass to be unique within the 0.02 Da tolerance (a 0.1 Da margin is noted as an alternative) so only unambiguous m/z values are imaged. Aggregate qualifying peptides by protein and retain only proteins supported by at least 5 qualifying peptides. Tally peptides and proteins remaining after each successive filter and compare counts against the paper's Figure 4 and Supplemental File 2; the cascade removes up to 70% of initially identified peptides while no more than 50% of initially identified proteins are filtered out.

## Related tools

- **MSIght** (Platform whose LC-MS/MS workflow applies the unique-peptide, unique-monoisotopic-mass, and >=5-peptide-per-protein filters to MSFragger PSM output) — https://github.com/lingjunli-research/MSIght
- **Python** (Language in which MSIght and the filtering cascade are implemented and distributed)
- **Jupyter** (Notebook application of MSIght that explains the analysis pipeline step-by-step)

## Examples

```
git clone https://github.com/lingjunli-research/MSIght.git && pip install msight
```

## Evaluation signals

- filtered_protein_results.csv exists and is well-formed tabular output
- Stepwise peptide counts match the values reported in Figure 4 and Supplemental File 2 of the paper
- Stepwise protein counts match the values reported in Figure 4 and Supplemental File 2
- Total peptide removal across the cascade is consistent with the reported 'up to 70%' reduction, while protein removal stays within the reported <=50% bound
- Every retained protein is supported by at least 5 qualifying unique peptides, and every retained peptide's monoisotopic mass is unique within the 0.02 Da tolerance

## Limitations

- The unique-peptide plus unique-monoisotopic-mass filters can remove up to 70% of initially identified peptides, substantially shrinking the set of imaged species
- The 0.02 Da monoisotopic-mass uniqueness tolerance has an alternative 0.1 Da margin; choice of tolerance changes which peptides survive filtering
- Proteins supported by fewer than 5 qualifying peptides are discarded, dropping identifications that may still be biologically relevant

## Evidence

- [results] successive filters — unique peptides, then a unique monoisotopic mass filter — removed up to 70% of the initially identified peptides: [evidence span withheld — anchor status: not_found]
- [results] no more than 50% of the initially identified proteins were filtered from the results (Figure 4): [evidence span withheld — anchor status: not_found]
