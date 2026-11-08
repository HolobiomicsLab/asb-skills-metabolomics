---
name: kegg-compound-reference-mapping
description: Use when an untargeted LC-MS feature/peak list must be assigned initial
  KEGG compound candidates by m/z matching before any biological-context scoring,
  or when reproducing mWISE's matching stage to benchmark annotators against KEGG-only
  databases.
when_to_use_negative:
- The target reference database is not KEGG — mWISE only offers data to annotate against
  KEGG, so matching to HMDB, Lipid Maps, or T3DB requires xMSannotator or another
  tool
- The peaks are already matched to a compound database and only require downstream
  clustering or diffusion prioritization — re-running the matching stage would be
  redundant
- No adduct/fragment model is known for the acquisition mode, making m/z-to-compound
  matching unreliable without additional deconvolution
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3174
tools:
- name: mWISE
  role: Provides the matching Stage command that performs the KEGG reference mapping
    of LC-MS peaks
  repo: github.com/b2slab/mWISE
- name: Genomic Ranges
  role: R package used by the matching Stage to rapidly match all LC-MS peaks to the
    KEGG database
- name: R
  role: Runtime environment in which mWISE and the matching Stage execute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:bd0d3198ef61e2349f8360d02102a5c7d12f68c5104398d28e30b8adc1a55b42
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# kegg-compound-reference-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rapidly match LC-MS peak mass-to-charge ratios against the KEGG compound database using mWISE's matching Stage (built on the Genomic Ranges R package) over a configurable set of adducts and fragments, producing the initial candidate-compound annotations that feed clustering and diffusion prioritization.

## When to use

Use when an untargeted LC-MS feature/peak list must be assigned initial KEGG compound candidates by m/z matching before any biological-context scoring, or when reproducing mWISE's matching stage to benchmark annotators against KEGG-only databases.

## When NOT to use

- The target reference database is not KEGG — mWISE only offers data to annotate against KEGG, so matching to HMDB, Lipid Maps, or T3DB requires xMSannotator or another tool
- The peaks are already matched to a compound database and only require downstream clustering or diffusion prioritization — re-running the matching stage would be redundant
- No adduct/fragment model is known for the acquisition mode, making m/z-to-compound matching unreliable without additional deconvolution

## Inputs

- LC-MS peak/feature table with mass-to-charge (m/z) values per acquisition mode (e.g. Trypanosoma dataset split into positive and negative acquisition mode object lists)
- Set of adducts and fragments to consider during matching
- KEGG compound reference database entries

## Outputs

- Per-peak candidate list of KEGG compound identifiers (m/z-to-compound matches)
- Initial diffusion input labels vector seeds derived from matched candidates

## How to apply

Run the mWISE matching Stage, which uses the Genomic Ranges R package to rapidly match every LC-MS peak to the KEGG database while considering a set of adducts and fragments, so each peak receives a list of potential KEGG compound candidates. Prepare the input peak list first: remove features without signal and apply the 80% rule where intensity equals 0, mirroring the mWISE input filtering protocol. The matching is restricted to KEGG as the reference database — mWISE only offers data to annotate against KEGG, unlike tools such as xMSannotator that also cover HMDB, Lipid Maps, and T3DB. Accept that matching is only the first stage of the mWISE pipeline: the resulting candidate lists are subsequently passed to spectral/k-means/DBSCAN clustering and then to diffusion scoring in a FELLA or RClass network. Validate the stage by confirming that each peak's candidate set is non-empty for peaks with plausible adduct assignments and that the downstream cluster-based filter (e.g. reducing Assay 2 false positives from 17598 to 1671) operates on the matched candidates.

## Related tools

- **mWISE** (Provides the matching Stage command that performs the KEGG reference mapping of LC-MS peaks) — github.com/b2slab/mWISE
- **Genomic Ranges** (R package used by the matching Stage to rapidly match all LC-MS peaks to the KEGG database)
- **R** (Runtime environment in which mWISE and the matching Stage execute)

## Examples

```
Rscript -e "mWISE::matchingStage(peaks = peak_list, adducts = adduct_set, fragments = fragment_set, database = 'KEGG')"
```

## Evaluation signals

- Every LC-MS peak with a plausible adduct assignment receives a non-empty set of KEGG compound candidates from the matching Stage
- Counts of matched candidates per peak are consistent with the adduct/fragment model used (no candidates appear for adducts outside the configured set)
- Applying the subsequent cluster-based filter reduces the number of false positives carried into diffusion (e.g. Assay 2 FP dropping from 17598 to 1671), confirming the matched candidates were the actual input
- Recomputed m/z-to-KEGG matches reproduce the article's reported TP/FP counts for the matching stage, e.g. Assay 1 with 144 TP, 13 FN, and 157 references at matching sensitivity 0.92
- Downstream diffusion input label vector is built exclusively from the matched KEGG candidates, with no unmatched peaks silently entering the diffusion scoring step

## Limitations

- mWISE only offers data to annotate against the KEGG database — an acknowledged limitation relative to xMSannotator, which also covers HMDB, Lipid Maps, and T3DB
- Matching depends on the configured set of adducts and fragments; peaks with adducts outside this set will not be matched
- Matching alone does not disambiguate isomers or related metabolites; it must be followed by clustering and diffusion prioritization to produce a usable ranked annotation

## Evidence

- [methods] matching Stage uses Genomic Ranges to match LC-MS peaks to KEGG: [evidence span withheld — anchor status: not_found]
