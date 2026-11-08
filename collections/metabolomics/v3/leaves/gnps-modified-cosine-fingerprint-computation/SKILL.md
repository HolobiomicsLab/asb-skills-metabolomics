---
name: gnps-modified-cosine-fingerprint-computation
description: Use when query MS/MS spectra (e.g., MassQL-filtered siderophore hits)
  must be converted into sample-wise similarity fingerprints to feed the NPOmix k-nearest
  neighbor classifier or compare spectra across paired genome–metabolome samples.
when_to_use_negative:
- You need full GNPS molecular networking outputs (cluster edges, library hits) rather
  than per-sample similarity fingerprints — the README notes 'we did not use the full
  classical GNPS molecular networking capabilities'
- The query spectra are isomers or homologs that modified cosine cannot resolve —
  NPOmix 'does not have the resolution to distinguish between isomers and regioselectivity
  of the various homologs'
- There are no paired samples or no spectra similar to the query in the sample set
  — fingerprints will be uninformative, as with fungal species whose BGCs match nothing
  in the training set
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Provides the pairwise modified cosine scoring used to compute maximum spectrum
    similarity of each query spectrum against each experimental sample
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: NPOmix
  role: Consumes the MS/MS fingerprint testing matrix; dereplication notebook logic
    is used to build the nonredundant MGF of query spectra
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: MassQL
  role: Upstream filter that extracts the iron-bound putative siderophore spectra
    that become the query spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:0e709df386c959e56d8fae261cd24b0e302a3a46ae9592a7e0552e33d1146430
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - MassQL
  - NPOmix
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# GNPS modified cosine fingerprint computation

## Summary

Computes MS/MS similarity fingerprints by scoring each query spectrum against all spectra in every paired experimental sample via GNPS pairwise modified cosine, producing the maximum-similarity vector used as NPOmix's KNN test matrix.

## When to use

Use when query MS/MS spectra (e.g., MassQL-filtered siderophore hits) must be converted into sample-wise similarity fingerprints to feed the NPOmix k-nearest neighbor classifier or compare spectra across paired genome–metabolome samples.

## When NOT to use

- You need full GNPS molecular networking outputs (cluster edges, library hits) rather than per-sample similarity fingerprints — the README notes 'we did not use the full classical GNPS molecular networking capabilities'
- The query spectra are isomers or homologs that modified cosine cannot resolve — NPOmix 'does not have the resolution to distinguish between isomers and regioselectivity of the various homologs'
- There are no paired samples or no spectra similar to the query in the sample set — fingerprints will be uninformative, as with fungal species whose BGCs match nothing in the training set

## Inputs

- Nonredundant MGF file of dereplicated query MS/MS spectra (e.g., MassQL-filtered putative siderophore spectra)
- Paired experimental LC-MS/MS sample spectra from the PoDP (1,040 sample sets, merged across fractions/culture conditions per genome)
- GNPS molecular networking job outputs (e.g., GNPS task b915d1f78c364bf4be9900047760d95c)

## Outputs

- MS/MS similarity fingerprint matrix (rows = query spectra, columns = 1,040 paired samples) of maximum modified cosine scores
- Testing matrix fed to the trained NPOmix KNN classifier to obtain voted GCF labels per spectrum

## How to apply

Dereplicate the query hits (e.g., the 380 MassQL iron-bound siderophore spectra) into a nonredundant MGF file using the NPOmix dereplication notebook logic. Then, via GNPS, compute the pairwise modified cosine score between each query spectrum and every MS/MS spectrum in each of the paired experimental samples (e.g., 1,040 sample columns from the PoDP dataset), retaining only the maximum similarity per sample. Note that this uses only GNPS's modified cosine scoring, not full molecular networking. The resulting fingerprint matrix (rows = query spectra, columns = samples) becomes the testing matrix for the NPOmix KNN classifier trained on BiG-SCAPE BGC fingerprints, which use the analogous 1 − raw-distance similarity with scores below the 0.7 cutoff treated as absent.

## Related tools

- **GNPS** (Provides the pairwise modified cosine scoring used to compute maximum spectrum similarity of each query spectrum against each experimental sample) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **NPOmix** (Consumes the MS/MS fingerprint testing matrix; dereplication notebook logic is used to build the nonredundant MGF of query spectra) — https://github.com/tiagolbiotech/NPOmix_python
- **MassQL** (Upstream filter that extracts the iron-bound putative siderophore spectra that become the query spectra)

## Examples

```
git clone https://github.com/tiagolbiotech/NPOmix_python.git && python run_npomix.py --npomix_inputMassSpecFile dereplicated_siderophore_queries.mgf --npomix_inputMetaFile MSMS_fingerprints_1040samples.tsv
```

## Evaluation signals

- Fingerprint matrix has exactly one row per dereplicated query spectrum and one column per paired sample (1,040 in the PoDP release)
- Each cell is the maximum modified cosine score in [0.0, 1.0] for that spectrum–sample pair, not an average or sum
- Query spectra absent from a sample yield similarity 0.0 (analogous to the 0.7 BiG-SCAPE absence cutoff, but no cutoff is applied to MS/MS cosine scores)
- Projected fingerprints should reproduce known links, e.g., the m/z 780 spectrum assigned to the siderophore GCF containing the Achromobacter xylosoxidans NH44784-1996 BGC with 100% Jaccard co-occurrence
- Reference spectra from the training set (18 known links in the first release) should classify to their experimentally validated GCFs at reported precision (top-1 = 81.8%)

## Limitations

- Modified cosine fingerprints cannot distinguish isomers or regioselectivity of homologs
- Precision may not be reproducible for datasets dissimilar to the PoDP training set
- Cryptic spectra without GNPS library matches can be fingerprinted, but resulting links require proper structure elucidation to confirm planar (2D) and absolute (3D) chemistry
- Complete genomes are preferred over fragmented metagenome-assembled BGCs, which reduce fingerprint comparability on the BGC side

## Evidence

- [readme] In the case of MS/MS fingerprints (Fig. 1D), we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample.: "In the case of MS/MS fingerprints (Fig. 1D), we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS"
- [readme] we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score between a pair of MS/MS spectra.: "we did not use the full classical GNPS molecular networking capabilities; we only used the functions required to calculate a modified cosine similarity score"
- [results] after MassQL filters iron-bound spectra from the LC-MS/MS profiles, NPOmix connected two of these putative siderophores with 100% Jaccard similarity: [evidence span withheld — anchor status: not_found]
- [discussion] NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs: "NPOmix (a correlation-based approach) does not have the resolution to distinguish between isomers and regioselectivity of the various homologs"
- [readme] for our first release, this testing matrix contained 15 reference MS/MS fingerprints (rows) for MS/MS reference spectra from the GNPS database: "for our first release, this testing matrix contained 15 reference MS/MS fingerprints (rows) for MS/MS reference spectra from the GNPS database"
