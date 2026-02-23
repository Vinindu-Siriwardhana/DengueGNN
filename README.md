#  DengueGNN

**DengueGNN: In Silico Discovery of the Dengue NS1 Human Interactome via Siamese Graph Attention Networks and Proteome-Wide Screening**
Vinindu Siriwardhana — February 2026

Mapping how Dengue virus hijacks the human body using Graph Neural Networks.

---

## Overview

Dengue virus infects ~390 million people annually. Severe cases (Dengue Hemorrhagic Fever / Dengue Shock Syndrome) involve vascular leakage, cytokine storm, and metabolic acidosis — but the **complete molecular interaction map of NS1 with human proteins remains unknown**.

**DengueGNN** is a proteome-wide screening framework that:

* Learns structural protein–protein interaction (PPI) patterns from human-human data
* Applies a Siamese Graph Attention Network (GATv2)
* Screens 20,565 human proteins against Dengue NS1
* Produces a ranked interactome with biological cluster discovery

This entire pipeline runs on a consumer GPU (RTX 4050, 6GB VRAM).

---

## Key Contributions

*  Achieved **Test AUC = 0.8729** on held-out human PPI prediction
*  Screened entire human proteome against NS1 (PDB: 4O6B)
*  Rediscovered known NS1 interactors (ACTB, UBB) without training on NS1 data
*  Identified 3 biological hubs explaining severe dengue
*  Discovered a **novel predicted NS1–Carbonic Anhydrase interaction** (CA1/CA2/CA3)
*  100% open-source, zero cloud cost

---

##  Architecture

### Model: Siamese GATv2

* Two identical graph encoders with shared weights
* GATv2Conv layers (dynamic attention)
* Readout: Mean Pool + Max Pool
* Interaction head:

  ```
  [u ; v ; |u−v| ; u⊙v] → 3-layer MLP → probability
  ```

Output: Interaction probability ∈ [0,1]

---

##  Protein Representation

Each protein is converted into a graph:

| Component     | Description                          |
| ------------- | ------------------------------------ |
| Nodes         | Amino acid residues (Cα coordinates) |
| Node Features | 480-dim ESM-2 embeddings             |
| Edges         | Residues within 10 Å                 |
| Edge Weight   | 1 / distance                         |
| QC            | AlphaFold pLDDT > 50                 |

**Embeddings:** `esm2_t12_35M_UR50D` (35M parameter transformer)
**Structures:** AlphaFold DB v4 + PDB 4O6B (DENV-2 NS1)

---

##  Dataset

**Training data source:** STRING database (score ≥ 700)

* 56,182 human-human protein pairs
* 1:1 negative sampling
* Strict protein-level split (no leakage)

| Split      | Pairs  |
| ---------- | ------ |
| Train      | 39,722 |
| Validation | 8,230  |
| Test       | 8,230  |

The model never sees test proteins during training.

---

##  Performance

| Metric              | Value  |
| ------------------- | ------ |
| Test AUC            | 0.8729 |
| Avg Precision       | 0.8348 |
| Accuracy            | 78%    |
| Precision           | 0.82   |
| Recall              | 0.71   |
| False Positive Rate | 7.6%   |

High precision ensures low-cost experimental validation.

---

##  Proteome-Wide Screening

* Query: Dengue NS1 (PDB: 4O6B)
* Search space: 23,586 AlphaFold human proteins
* After filtering: 20,565 proteins scored
* Speed: ~1,300 proteins/hour
* Runtime: ~16 hours

Output: Ranked CSV of predicted NS1 interactors

---

##  Major Biological Findings

### 1️). Vascular Leak Hub

ACTB · CTNNB1 · RHOA
→ Direct structural destabilisation of endothelial junctions

### 2️) Cytokine Storm Hub

TNF · IL6 · TRAF6
→ Multi-point amplification of inflammatory signalling

### 3️) Novel Discovery Hub

AKT1 · ADIPOQ · CA1 · CA2 · CA3

**Most Novel Finding:**
Predicted NS1 binding to Carbonic Anhydrases (CA1/CA2/CA3)

Potential explanation for metabolic acidosis in dengue shock — never previously reported.

---

##  Repository Structure

```
DengueGNN/
│
│──Binding Results from HDOCK
|
├── Results/
│   ├── ns1_human_interactome_results.csv
│   ├── top_200_ns1_interactors.csv
│    
│ 
├── data/
│   ├── interaction_master_list_uniprot.csv
│   └── splits/
│       ├── train.csv
│       ├── val.csv
│       └── test.csv
│
├── models/
│   └── best_dengue_gnn.pth
│
├── notebooks/
│   └── DengueGNN_Master_File.ipynb
│
├── LICENSE
└── README.md
```

---

##  Technical Stack

| Component      | Detail                        |
| -------------- | ----------------------------- |
| Framework      | PyTorch 2.7.0                 |
| GNN            | PyTorch Geometric (GATv2Conv) |
| CUDA           | 12.4                          |
| GPU            | NVIDIA RTX 4050 (6GB)         |
| Training Time  | Overnight                     |
| Screening Time | ~16 hours                     |

---

##  Running the Project

### 1️) Install Dependencies

```bash
conda create -n denguegnn python=3.10
conda activate denguegnn

pip install torch torchvision torchaudio
pip install torch-geometric
pip install fair-esm
pip install biopython pandas numpy tqdm
```

---

### 2️) Train Model

```bash
python train.py
```

---

### 3️) Screen Human Proteome

```bash
python screen_proteome.py
```

Output:

```
ns1_human_interactome_results.csv
```

---

##  Recommended Experimental Validation

1. Co-IP or SPR validation for CA1/CA2
2. Docking (ClusPro / HADDOCK) for NS1–AKT1
3. In-silico mutagenesis of NS1 binding hotspots
4. STRING network enrichment of Top 200 genes

---

##  Why This Matters

This project demonstrates that:

* Viral-host interactome discovery can be performed without supercomputers
* Structural GNNs can identify biologically meaningful clusters
* Novel therapeutic hypotheses can emerge from AI-first discovery pipelines

Carbonic Anhydrase interaction alone may warrant standalone publication.

---

##  License

MIT License

## Contact

Vinindu Siriwardhana
in/vinindu-siriwardhana
vinidusiri@gmail.com


---

