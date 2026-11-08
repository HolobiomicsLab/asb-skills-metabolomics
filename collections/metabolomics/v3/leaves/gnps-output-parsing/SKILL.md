---
name: gnps-output-parsing
description: Use when you hold DEREPLICATOR+ PSM results (e.g. score threshold 15,
  FDR = 0%) plus a precomputed GNPS spectral network, and must expand each identification
  into variant-containing metabolite families.
when_to_use_negative:
- You need fragment-graph scoring or decoy-based FDR estimation of spectra themselves
  — that is DEREPLICATOR+'s database-search step, not network-output parsing.
- Your spectra are not grouped into GNPS spectral clusters (no METABOLOMICS-SNETS-V2
  output), so no propagation via clusters is possible.
- The input is already a resolved list of metabolite families/variants — re-parsing
  GNPS output would be redundant.
tools:
- name: DEREPLICATOR+
  role: Produces the significant compound–spectrum matches whose identifications are
    propagated through the GNPS spectral network
  repo: http://mohimanilab.cbd.cmu.edu/software/
- name: GNPS
  role: Supplies the precomputed molecular networking clusters (METABOLOMICS-SNETS-V2
    'Clustered Spectra as MGF') onto which PSMs are propagated
  repo: http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:a47c571a17f678e7c8ff8260d58d584c63665aeb17ca311d5c8cad7b77152e3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEREPLICATOR+
  - GNPS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gnps-output-parsing

## Summary

Parse GNPS molecular networking output (METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF' clusters) together with a filtered DEREPLICATOR+ compound–spectrum match report to propagate seed identifications across spectral clusters and enumerate metabolite families and their variants.

## When to use

Use when you hold DEREPLICATOR+ PSM results (e.g. score threshold 15, FDR = 0%) plus a precomputed GNPS spectral network, and must expand each identification into variant-containing metabolite families.

## When NOT to use

- You need fragment-graph scoring or decoy-based FDR estimation of spectra themselves — that is DEREPLICATOR+'s database-search step, not network-output parsing.
- Your spectra are not grouped into GNPS spectral clusters (no METABOLOMICS-SNETS-V2 output), so no propagation via clusters is possible.
- The input is already a resolved list of metabolite families/variants — re-parsing GNPS output would be redundant.

## Inputs

- DEREPLICATOR+ compound–spectrum match report filtered to FDR = 0% identifications
- GNPS molecular network output: METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF' directory (clustered spectra)
- Spectra files in MGF or mzXML/mzML/mzData format (centroided LC-MS/MS)

## Outputs

- propagations.pdf (network graph rendering of each connected component)
- propagations_detailed.txt (per-PSM list of spectra from the same cluster and clusters at distance 1 and 2)
- propagations_short.txt
- variant_expansion_summary.tsv (seed metabolite, family size, propagated variants)

## How to apply

Load the DEREPLICATOR+ compound–spectrum match report and retain only identifications with FDR = 0% (here yielding 24 top-scoring seed metabolites at score threshold 15). Load the GNPS METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF' directory and index its spectrum clusters. For each significant PSM, propagate the identification to every spectrum in the same cluster and to clusters at network distance 1 and 2, recording one representative variant per cluster. Group propagated spectra into connected components, treating each component containing at least one identified spectrum as a metabolite family — DEREPLICATOR+'s propagation principle — and enumerate additional variants.

## Related tools

- **DEREPLICATOR+** (Produces the significant compound–spectrum matches whose identifications are propagated through the GNPS spectral network) — http://mohimanilab.cbd.cmu.edu/software/
- **GNPS** (Supplies the precomputed molecular networking clusters (METABOLOMICS-SNETS-V2 'Clustered Spectra as MGF') onto which PSMs are propagated) — http://gnps.ucsd.edu/ProteoSAFe/static/gnps-theoretical.jsp

## Examples

```
import matplotlib.pyplot as plt, networkx as nx; nx.draw(G, with_labels=True); plt.savefig('propagations.pdf')
```

## Evaluation signals

- Reproducing the reference analysis yields exactly 15 metabolite families and 557 additional variants from the 24 top-scoring seed metabolites.
- Every retained identification has FDR = 0% and meets the score threshold of 15.
- propagations_detailed.txt lists, for each significant PSM, all spectra from the same cluster and from clusters at network distance 1 and 2.
- Each reported metabolite family (connected component) contains at least one identified spectrum.
- variant_expansion_summary.tsv columns (seed metabolite, family size, propagated variants) sum consistently: variant counts across the 15 families total 557.

## Limitations

- If matplotlib and networkx are not installed, spectral network propagation graphs are generated in plain text format only (no PDF rendering).
- DEREPLICATOR+ is not applicable to all classes of compounds, so some seed spectra may be unidentifiable.
- DEREPLICATOR+'s simple shared-peaks scoring gives a high false-negative rate, limiting which identifications can seed the propagation.
- No version numbers are stated for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS, so reproducing exact GNPS network outputs may require matching unspecified versions.

## Evidence

- [discussion] DEREPLICATOR+ constructs the molecular network of all spectra and selects connected components with at least one identified spectrum.: "DEREPLICATOR+ constructs the molecular network of all spectra and selects connected components with at least one identified spectrum."
- [intro] GNPS as spectral repository underlying the networking step.: "Global Natural Products Social (GNPS) molecular networking project4 is a recent mass spectrometry data repository in the field of natural products"
- [readme] Rendering propagation graphs requires matplotlib/networkx; otherwise plain text only.: [evidence span withheld — anchor status: external_doc]
- [readme] Fallback behavior when rendering libraries are missing.: "If they are not installed, the propagation will be generated in a plain text format only (see `--spec-network` option)."
