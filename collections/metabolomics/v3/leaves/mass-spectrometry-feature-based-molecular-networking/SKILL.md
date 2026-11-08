---
name: mass-spectrometry-feature-based-molecular-networking
description: Use when a reanalysis of public metabolomics data (e.g. GNPS-MassIVE
  MSV000081832) needs molecular families grouped by MS/MS fragmentation similarity
  and tested for differential activity between phenotypes such as healthy vs caries
  samples.
when_to_use_negative:
- The question is about gene cluster abundance or expression from metagenomic/metatranscriptomic
  reads — that is BiG-MAP's mapping/analysis modules, not FBMN/PALS
- Raw MS instrument files (mzML/NetCDF) requiring de novo feature detection are all
  you have and the processed MGF/feature table has not been generated yet
- No phenotype metadata exists to feed PALS, so family-level differential activity
  cannot be scored
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: BiG-MAP
  role: bigmap-download entry point used to fetch the processed MGF spectra, feature
    quantification table, and sample metadata from GNPS-MassIVE accession MSV000081832
  repo: github.com/medema-group/BiG-MAP
- name: GNPS FBMN
  role: Feature-Based Molecular Networking workflow that clusters features into molecular
    families by spectral fragmentation similarity
- name: PALS
  role: Pathway Activity Level Scoring; computes set-based activity scores for each
    molecular family from the feature table given phenotype metadata
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:4d41dcde87ec7ab94868ebec152c2a5768f26a1e166a3691682aa99e3ad1bdc3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - GNPS FBMN
  - PALS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mass-spectrometry-feature-based-molecular-networking

## Summary

Cluster MS/MS metabolomics features into molecular families with Feature-Based Molecular Networking (FBMN) on GNPS, then score families for phenotype-associated activity with PALS, as done to link polyacrolein-like metabolites to reuterin gene cluster enrichment in oral caries data.

## When to use

Use when a reanalysis of public metabolomics data (e.g. GNPS-MassIVE MSV000081832) needs molecular families grouped by MS/MS fragmentation similarity and tested for differential activity between phenotypes such as healthy vs caries samples.

## When NOT to use

- The question is about gene cluster abundance or expression from metagenomic/metatranscriptomic reads — that is BiG-MAP's mapping/analysis modules, not FBMN/PALS
- Raw MS instrument files (mzML/NetCDF) requiring de novo feature detection are all you have and the processed MGF/feature table has not been generated yet
- No phenotype metadata exists to feed PALS, so family-level differential activity cannot be scored

## Inputs

- Processed MGF spectra file from a GNPS-MassIVE accession (e.g. MSV000081832)
- Feature quantification table (features x samples) from the same accession
- Sample metadata / phenotype labels (e.g. 25 healthy vs 24 caries oral samples)

## Outputs

- Molecular networking results grouping features into molecular families by MS/MS fragmentation similarity
- pals_activity_scores.csv — full per-family PALS activity score table
- acrolein_family_summary.csv — enrichment summary for (poly)acrolein-annotated molecular families

## How to apply

First fetch the public dataset's processed MGF spectra, feature quantification table, and sample metadata (BiG-MAP's bigmap-download entry point handles accession-based retrieval). Run Feature-Based Molecular Networking on the GNPS workflow using the downloaded MGF and feature table so features are clustered into molecular families by spectral fragmentation similarity. Then run PALS (Pathway Activity Level Scoring) on the feature table with the phenotype metadata (here 25 healthy vs 24 caries samples) to compute set-based activity scores per molecular family. Rank families by PALS activity score for the phenotype of interest and filter for families whose member features carry the annotation class of interest (e.g. polymer-like (poly)acrolein). Write the per-family PALS activity score table and the annotation-class enrichment summary as separate CSV outputs.

## Related tools

- **BiG-MAP** (bigmap-download entry point used to fetch the processed MGF spectra, feature quantification table, and sample metadata from GNPS-MassIVE accession MSV000081832) — github.com/medema-group/BiG-MAP
- **GNPS FBMN** (Feature-Based Molecular Networking workflow that clusters features into molecular families by spectral fragmentation similarity)
- **PALS** (Pathway Activity Level Scoring; computes set-based activity scores for each molecular family from the feature table given phenotype metadata)

## Examples

```
bigmap-download -A MSV000081832_accessions.txt -O ./msv000081832/
```

## Evaluation signals

- pals_activity_scores.csv contains an activity score for every molecular family produced by the FBMN clustering
- The (poly)acrolein-annotated molecular family with three candidate metabolites scores as enriched in healthy samples, matching the paper's reported finding
- Sample counts in the phenotype groups match the source metadata (25 healthy vs 24 caries samples)
- The enriched polyacrolein-like family is consistent with the metabolomic masses whose fragmentation patterns are consistent with (poly)acrolein found in samples where the reuterin biosynthetic gene cluster was enriched

## Limitations

- The reuterin biosynthetic gene cluster was 'enriched but not exclusively found in healthy samples', so metabolite-family enrichment is not expected to be phenotype-exclusive
- The BiG-MAP tool has been 'only tested on Ubuntu', which constrains the environment for the download step
- Annotation is 'polyacrolein-like' / 'consistent with (poly)acrolein' — candidate metabolite assignments rest on fragmentation-pattern similarity, not confirmed structures

## Evidence

- [abstract] A putative reuterin biosynthetic gene cluster from a Streptococcus strain was enriched (not exclusively) in healthy samples, and metabolomic data showed masses with fragmentation patterns consistent with (poly)acrolein: "we found a putative reuterin biosynthetic gene cluster from a Streptococcus strain to be enriched but not exclusively found in healthy samples; metabolomic data from the same samples showed masses"
- [other] Task card finding: polyacrolein-like family enriched in healthy samples: [evidence span withheld — anchor status: not_found]
- [readme] BiG-MAP download module retrieves samples by accession from public repositories: [evidence span withheld — anchor status: external_doc]
- [readme] BiG-MAP is tested only on Ubuntu: [evidence span withheld — anchor status: external_doc]
