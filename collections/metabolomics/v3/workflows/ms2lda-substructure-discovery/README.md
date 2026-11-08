# ms2lda-substructure-discovery-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw MS2 spectra (mgf/mzML/msp) -> cleaned, intensity-normalized bag-of-fragments corpus  →  `msms-noise-filtering`, `mgf-spectra-noise-filtering-and-preprocessing`, `mass-spectrometry-spectrum-preprocessing--99e25f8cc3a2c48f`, `mass-spectra-preprocessing-and-binning`
2. **lda_modeling** — bag-of-fragments corpus -> Mass2Motifs (LDA topic model over spectral documents)  →  `latent-dirichlet-allocation-topic-modeling`, `lda-topic-model-visualization`, `mass-spectrometry-fragmentation-motif-visualization`, `msms-fragment-bag-of-words-corpus-construction`
3. **motif_annotation** — Mass2Motifs -> putative substructure annotations (MotifDB + Spec2Vec embedding match)  →  `mass2motif-spectral-matching-annotation`, `motif-annotation-agreement-comparison`, `combinatorial-fragmentation-substructure-annotation`
4. **network_mapping** — annotated Mass2Motifs + GNPS molecular network -> motif-enriched network nodes  →  `molecular-network-mass2motif-mapping`, `molecular-network-motif-annotation`, `molecular-network-annotation-propagation--3c622bace7b9313a`, `mass-spectral-molecular-network-motif-mapping`
5. **report** — consolidate motifs + annotations + motif-enriched network into a substructure discovery report  →  `ms2lda-motif-summary-parsing`, `fragment-pseudo-spectrum-representation-of-motifs`, `mass2motif-membership-annotation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
