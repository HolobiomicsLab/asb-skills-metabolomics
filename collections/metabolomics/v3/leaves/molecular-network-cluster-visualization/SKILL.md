---
name: molecular-network-cluster-visualization
description: Use when a set of identified spectra (PSMs/compound–spectrum matches)
  must be expanded to related spectra via GNPS molecular network clusters, with variants
  grouped into families and rendered as network graphs.
when_to_use_negative:
- Spectra are not organized into GNPS molecular network clusters (e.g., isolated spectra
  with no spectral network output), so no propagation across network distance 1–2
  is possible
- The query metabolite class is outside the tool's scope — the article notes DEREPLICATOR+
  'is not applicable to all the classes of compounds'
- Only a visualization of raw (unidentified) clusters is needed with no identified
  spectra to seed families — components without at least one identified spectrum are
  not treated as metabolite families
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: Produces the significant compound–spectrum matches (FDR-filtered) whose identities
    are propagated through the molecular network
  repo: github.com/ablab/npdtools
- name: GNPS
  role: Provides the molecular networking output (METABOLOMICS-SNETS-V2 clustered
    spectra) whose components are rendered as network graphs
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- name: matplotlib/networkx
  role: Python libraries used to render Spectral Network propagation graphs; if absent,
    propagation falls back to plain text
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:d2e43bd576d0a48313425f957050c9f266362080f2614fc61f3d01204a29ccd6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR+
  - GNPS
  - matplotlib/networkx
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# molecular-network-cluster-visualization

## Summary

Propagates database-search identifications (e.g., DEREPLICATOR+ PSMs at FDR 0%) across GNPS molecular network clusters and renders each connected component — a metabolite family — as a network graph, enumerating spectral variants of the identified seed metabolites.

## When to use

Use when a set of identified spectra (PSMs/compound–spectrum matches) must be expanded to related spectra via GNPS molecular network clusters, with variants grouped into families and rendered as network graphs.

## When NOT to use

- Spectra are not organized into GNPS molecular network clusters (e.g., isolated spectra with no spectral network output), so no propagation across network distance 1–2 is possible
- The query metabolite class is outside the tool's scope — the article notes DEREPLICATOR+ 'is not applicable to all the classes of compounds'
- Only a visualization of raw (unidentified) clusters is needed with no identified spectra to seed families — components without at least one identified spectrum are not treated as metabolite families

## Inputs

- DEREPLICATOR+ compound–spectrum match report (PSMs filtered to FDR = 0%)
- Precomputed GNPS molecular networking output (METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF' directory)
- Cluster/spectrum index (cluster memberships and inter-cluster network distances)

## Outputs

- propagations.pdf — network graph rendering of each metabolite-family connected component
- propagations_detailed.txt — per significant PSM, all spectra from the same cluster and from clusters at distance 1 and 2
- propagations_short.txt — condensed propagation report
- variant_expansion_summary.tsv — family-level expansion (seed metabolite, family size, propagated variants)

## How to apply

Load the DEREPLICATOR+ compound–spectrum match report and retain only identifications with FDR = 0% (in the SpectraActiSeq reproduction, the top 24 metabolites at score threshold 15). Index the precomputed GNPS METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF' output and, for each significant PSM, propagate the identification to every spectrum in the same cluster and to clusters at network distance 1 and 2, recording one representative variant per cluster. Group propagated spectra into connected components, treating each component containing at least one identified spectrum as a metabolite family. Render each connected component as a network graph with matplotlib/networkx (as NPDtools does for Spectral Network propagation graphs), writing propagations.pdf plus detailed and short text reports, and compile a family-level expansion table (seed metabolite, family size, propagated variants). The rationale follows the article's approach: DEREPLICATOR+ constructs the molecular network of all spectra and selects connected components with at least one identified spectrum, enabling high-throughput identification of natural product variants.

## Related tools

- **DEREPLICATOR+** (Produces the significant compound–spectrum matches (FDR-filtered) whose identities are propagated through the molecular network) — github.com/ablab/npdtools
- **GNPS** (Provides the molecular networking output (METABOLOMICS-SNETS-V2 clustered spectra) whose components are rendered as network graphs) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
- **matplotlib/networkx** (Python libraries used to render Spectral Network propagation graphs; if absent, propagation falls back to plain text)

## Examples

```
import matplotlib.pyplot as plt, networkx as nx; nx.draw(component_graph, with_labels=True); plt.savefig('propagations.pdf')
```

## Evaluation signals

- Reproduction from the 24 top-scoring DEREPLICATOR+ metabolites (score threshold 15, 0% FDR) must yield exactly 15 metabolite families and 557 additional variants
- propagations_detailed.txt must list, for each significant PSM, all spectra from the same cluster and from clusters at network distance 1 and 2
- Each cluster contributes at most one representative variant (no duplicate cluster-level variants per family)
- Every rendered connected component must contain at least one identified spectrum (seed PSM)
- variant_expansion_summary.tsv must reconcile: total propagated variants across 15 families equals 557

## Limitations

- The simple shared peaks scoring behind the identifications results in a high false-negative rate, so some spectra of a variant family may go unidentified
- The vast majority of GNPS spectra remain uninterpreted ('dark matter of metabolomics'), limiting how far propagation can annotate novel variants
- No version numbers are stated for DEREPLICATOR+ or GNPS; propagation behavior may vary across tool versions
- matplotlib/networkx are optional for NPDtools propagation — without them only plain-text propagation output is generated, no PDF graphs

## Evidence

- [discussion] DEREPLICATOR+ constructs the molecular network of all spectra and selects connected components with at least one identified spectrum.: "DEREPLICATOR+ constructs the molecular network of all spectra and selects connected components with at least one identified spectrum."
- [intro] DEREPLICATOR+ enables high-throughput identification of variants of known natural products by spectral networks: "DEREPLICATOR+ enables high-throughput identification of variants of known natural products by"
- [readme] For presenting Spectral Network propagation graphs, MetaMiner requires matplotlib and networkx Python libraries.: [evidence span withheld — anchor status: external_doc]
- [methods] The latest version is available in the Natural Product Discovery toolkit (NPDtools): [evidence span withheld — anchor status: not_found]
