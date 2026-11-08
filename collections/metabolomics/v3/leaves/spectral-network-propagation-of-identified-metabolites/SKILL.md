---
name: spectral-network-propagation-of-identified-metabolites
description: Use when you have DEREPLICATOR+ compound–spectrum matches filtered at
  a score/FDR cutoff and a precomputed GNPS spectral network (clustered MGF), and
  you need to count metabolite families and additional variants revealed by spectral
  propagation.
when_to_use_negative:
- Input spectra have no GNPS molecular networking clusters (no 'Clustered Spectra
  as MGF' output), so propagation across distance-1/2 clusters is impossible.
- No identifications pass the FDR = 0% cutoff, so every connected component would
  lack a seed and propagation would amplify false positives.
- The target compound classes are outside DEREPLICATOR+'s scope — the article states
  it 'is not applicable to all the classes of compounds.'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Produces the compound–spectrum matches (MSMs) at 0% FDR whose identifications
    are propagated; also constructs the molecular network and selects connected components
    with at least one identified spectrum
  repo: github.com/ablab/npdtools
- name: GNPS
  role: Provides the precomputed METABOLOMICS-SNETS-V2 spectral network ('Clustered
    Spectra as MGF') over which identifications are propagated
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- name: DEREPLICATOR
  role: Predecessor dereplication tool for peptidic natural products; supplies the
    PSM-report workflow that DEREPLICATOR+ propagation extends to non-peptidic metabolites
  repo: github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:ae6609504b3a58d018ba083db256a54cf3c4dc754e10202924f310a5c7f2b5ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - GNPS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-network-propagation-of-identified-metabolites

## Summary

Propagates DEREPLICATOR+ identifications (PSMs with 0% FDR) across a GNPS molecular network (METABOLOMICS-SNETS-V2 clusters) to annotate structurally related variant spectra in connected components that contain at least one identified spectrum. This expands a small set of dereplicated metabolites into metabolite families and enumerates unsequenced/unnamed variants.

## When to use

Use when you have DEREPLICATOR+ compound–spectrum matches filtered at a score/FDR cutoff and a precomputed GNPS spectral network (clustered MGF), and you need to count metabolite families and additional variants revealed by spectral propagation.

## When NOT to use

- Input spectra have no GNPS molecular networking clusters (no 'Clustered Spectra as MGF' output), so propagation across distance-1/2 clusters is impossible.
- No identifications pass the FDR = 0% cutoff, so every connected component would lack a seed and propagation would amplify false positives.
- The target compound classes are outside DEREPLICATOR+'s scope — the article states it 'is not applicable to all the classes of compounds.'

## Inputs

- DEREPLICATOR+ compound–spectrum match (PSM) report filtered to FDR = 0% (score threshold 15), yielding the seed metabolites
- GNPS METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF' output directory (spectrum clusters)
- Cluster-to-cluster adjacency at network distances 1 and 2 from the GNPS spectral network

## Outputs

- spec_nets/propagations.pdf (networkx/matplotlib graphs of each connected component)
- spec_nets/propagations_detailed.txt (per-PSM propagation to same-cluster spectra and distance-1/2 clusters)
- spec_nets/propagations_short.txt
- variant_expansion_summary.tsv (seed metabolite, family size, propagated variants; 15 families, 557 additional variants)

## How to apply

Load the DEREPLICATOR+ compound–spectrum match report and retain only identifications with FDR = 0% (here, the 24 top-scoring metabolites at score threshold 15). Load the precomputed GNPS molecular networking output — the METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF' directory — and index its spectrum clusters. For each significant PSM, propagate the identification to every spectrum in the same cluster and to clusters at network distance 1 and 2, recording one representative variant per cluster. Group the propagated spectra into connected components; DEREPLICATOR+ selects components with at least one identified spectrum, and each such component is treated as a metabolite family. Render components as network graphs with matplotlib/networkx and compile a family-level expansion table (seed metabolite, family size, propagated variants). In the reference reproduction this recovered 15 metabolite families and 557 additional variants from the 24 seed metabolites.

## Related tools

- **DEREPLICATOR+** (Produces the compound–spectrum matches (MSMs) at 0% FDR whose identifications are propagated; also constructs the molecular network and selects connected components with at least one identified spectrum) — github.com/ablab/npdtools
- **GNPS** (Provides the precomputed METABOLOMICS-SNETS-V2 spectral network ('Clustered Spectra as MGF') over which identifications are propagated) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- **DEREPLICATOR** (Predecessor dereplication tool for peptidic natural products; supplies the PSM-report workflow that DEREPLICATOR+ propagation extends to non-peptidic metabolites) — github.com/ablab/npdtools

## Examples

```
python bin/dereplicator+.py --spec-network  # enable Spectral Network propagation graphs (matplotlib/networkx) for propagated identifications
```

## Evaluation signals

- The reproduction reports exactly 15 metabolite families and 557 additional variants from the 24 top-scoring DEREPLICATOR+ metabolites (score threshold 15, 0% FDR).
- propagations_detailed.txt lists, for each significant PSM, all spectra from the same cluster and from clusters at distance 1 and 2.
- Each connected component counted as a metabolite family contains at least one identified spectrum; exactly one representative variant is recorded per cluster.
- variant_expansion_summary.tsv is internally consistent: family sizes equal seed metabolites plus propagated variants, and all seeds appear in the FDR = 0% PSM report.
- spec_nets/ contains all three expected artifacts (propagations.pdf, propagations_detailed.txt, propagations_short.txt) and graphs render without matplotlib/networkx errors.

## Limitations

- DEREPLICATOR+ uses simple shared-peaks scoring, which 'results in a high false-negative rate'; improved scoring was not yet implemented, so some variants may be missed.
- 'DEREPLICATOR+ is not applicable to all the classes of compounds', limiting which metabolite families can be seeded and propagated.
- The vast majority of GNPS spectra remain uninterpreted ('a large dark matter of metabolomics'), so many clusters in the network will never receive a seed identification.
- The P-value recursion is only approximate for non-tree fragmentation graphs; unbiased MCMC estimation was described as possible but not implemented, affecting the FDR filter that defines the seeds.

## Evidence

- [discussion] Propagation targets components with an identified spectrum: "DEREPLICATOR+ constructs the molecular network of all spectra and selects connected components with at least one identified spectrum."
- [readme] Propagation renders graphs via matplotlib/networkx (README): [evidence span withheld — anchor status: external_doc]
- [intro] Propagation enables high-throughput variant identification: "DEREPLICATOR+ enables high-throughput identification of variants of known natural products by"
- [abstract] Propagation dramatically increases identifications over prior dereplication: "identify an order of magnitude more natural products than previous dereplication efforts"
- [discussion] Scope limitation of DEREPLICATOR+: "Similar to all the existing metabolomics tools, DEREPLICATOR+ is not applicable to all the classes of compounds."
