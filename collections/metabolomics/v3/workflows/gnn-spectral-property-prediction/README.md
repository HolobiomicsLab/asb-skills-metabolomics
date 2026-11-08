# gnn-spectral-property-prediction-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **featurize** — SMILES structures -> attributed molecular graphs (RDKit + PyTorch Geometric)  →  `chemical-graph-parsing-and-featurization`, `molecular-graph-construction-from-smiles`
2. **architect** — molecular graph representation -> GNN architecture (message-passing layers + property head)  →  `graph-attention-network-implementation`, `molecular-graph-neural-network-implementation`, `molecular-graph-gcn-encoder-training`
3. **train** — GNN architecture + labelled property dataset -> trained model checkpoint  →  `retention-time-regression-model-training--8849e07a1d6b4a9c`, `molecular-graph-representation-learning`, `graph-neural-network-training--5c016ac055bd20cd`
4. **predict** — trained GNN + candidate structures -> predicted property values  →  `graph-neural-network-training--ea0bd47505660ff7`, `pretrained-gnn-inference`, `neural-network-model-inference`, `nmr-spectrum-inference-decoding`
5. **rescore** — predicted property + experimental evidence -> re-ranked / filtered annotation candidates  →  `putative-identity-ranking-by-rt-error`, `candidate-frequency-ranking-for-denovo-annotation`, `precursor-mass-candidate-screening`, `retention-order-score-integration`, `casmi-challenge-candidate-ranking`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
