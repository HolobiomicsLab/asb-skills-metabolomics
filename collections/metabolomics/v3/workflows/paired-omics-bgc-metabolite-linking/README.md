# paired-omics-bgc-metabolite-linking-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **bgc_mine** — assembled genomes -> detected BGCs (antiSMASH genome mining)  →  `biosynthetic-gene-cluster-prediction-antismash`, `biosynthetic-gene-cluster-annotation-parsing`, `genome-mining-bgc-detection`
2. **bgc_tokenize** — GenBank BGC records -> Pfam-domain tokens and sub-cluster motifs (iPRESTO)  →  `biosynthetic-domain-curation-via-ec-annotation`, `pfam-domain-tokenisation`, `biosynthetic-gene-domain-tokenisation`, `presto-top-motif-matching`
3. **gcf_cluster** — BGCs -> gene cluster families (BiG-SCAPE similarity clustering)  →  `biosynthetic-gene-cluster-similarity-network-analysis`, `gcf-prevalence-distribution-analysis-across-genomes`, `gene-cluster-family-grouping`, `gcf-genome-based-class-annotation`
4. **mf_network** — paired LC-MS/MS spectra -> molecular families (GNPS feature-based molecular networking)  →  `gnps-molecular-networking-workflow`, `feature-based-molecular-networking--7b025a18b065ae1c`, `gcms-molecular-networking`, `feature-based-molecular-networking--a1d4da5bf30d5f86`
5. **link_score** — GCFs + MFs + strain co-occurrence -> ranked GCF-MF links (NPLinker Metcalf/hypergeometric scoring)  →  `gcf-molecular-family-link-scoring`, `metcalf-co-occurrence-scoring`, `metcalf-strain-correlation-scoring`, `hypergeometric-score-standardisation`
6. **report** — consolidate scored links + provenance into a ranked BGC-metabolite link table  →  `validated-link-rank-evaluation`, `bgc-msms-candidate-link-scoring`, `bgc-spectrum-link-ranking`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
