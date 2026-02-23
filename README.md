#  DengueGNN

**DengueGNN: In Silico Discovery of the Dengue NS1 Human Interactome via Siamese Graph Attention Networks and Proteome-Wide Screening**

Vinindu Siriwardhana — February 2026

![Graphic](https://github.com/user-attachments/assets/988a26f6-2adb-4d28-8c0f-25d27fc634ea)

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
*  Predicted known NS1 interactors (ACTB, UBB) with High Probability (>0.9) without training on NS1 data
*  Identified 3 biological hubs explaining severe dengue
*  Discovered a **novel predicted NS1–Carbonic Anhydrase interaction** (CA1/CA2/CA3)
*  HDOCK docking confirmed NS1–CA1 (confidence 0.8359) and NS1–AKT1 (confidence 0.8352) as high-confidence structural interactions
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
| Edge Weight   | distance                             |
| QC            | AlphaFold pLDDT > 50                 |

**Embeddings:** `esm2_t12_35M_UR50D` (35M parameter transformer)
**Structures:** AlphaFold DB v4 + PDB 4O6B (DENV-2 NS1)

---

##  Dataset

**Training data source:** STRING database (score ≥ 700)

* 56,182 human-human protein pairs
* Nearly 1:1 negative sampling
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
###  Predicted NS1–Human Interactome Map

<img width="1497" height="1620" alt="string_normal_image" src="https://github.com/user-attachments/assets/de35e963-7e47-4c2e-bc11-2756ef809701" />
Figure 1: Functional interaction network of the top 200 human proteins predicted by DengueGNN. Nodes represent human proteins; edges represent known human-human interactions (STRING v12.0).

---
##  Major Biological Findings

### 1️). Vascular Leak Hub

ACTB · CTNNB1 · RHOA
→ Direct structural destabilisation of endothelial junctions

### 2️) Cytokine Storm Hub

TNF · IL6 · TRAF6
→ Multi-point amplification of inflammatory signalling

### 3️) Novel Discovery Hub

ADIPOQ · CA1 · CA2 · CA3

**Most Novel Finding:**
Predicted NS1 binding to Carbonic Anhydrases (CA1/CA2/CA3)

Potential explanation for metabolic acidosis in dengue shock — never previously reported.

---

### HDOCK Molecular Docking Validation

Top GNN predictions were independently validated using HDOCK protein-protein docking.

| Interaction | Docking Score | Confidence Score |
|---|---|---|
| NS1 — CA1 | -231.41 | 0.8359 |
| NS1 — AKT1 | -231.16 | 0.8352 |

> Confidence scores > 0.80 indicate high likelihood of a genuine biological interaction. Both results were obtained independently of the GNN — purely from 3D structural complementarity.

**NS1-AKT1** showed exceptional consistency across all 10 docking models (confidence 0.74–0.84), suggesting a well-defined, structurally stable binding interface.

<img width="4000" height="2457" alt="NS1-AKT1_protein_interaction" src="https://github.com/user-attachments/assets/45eb8e37-c8ca-4fdc-af41-7cec84c8a2a2" />



**NS1-CA1** represents the most novel finding in this study.

<img width="4000" height="2457" alt="NS1-CA1 protien Interaction" src="https://github.com/user-attachments/assets/d58c560e-517e-4003-b08f-365e69ccfa71" />



---

##  Repository Structure

```
DengueGNN/
│
├── data/
|   ├── interaction_master_list_uniprot.csv
│   └── splits/
│       ├── train.csv
│       ├── val.csv
│       └── test.csv
├    
├── models/
│   └── best_dengue_gnn.pth
│
├── notebooks/
│   └── DengueGNN_Master_File.ipynb
|
├── results/
│   ├── ns1_human_interactome_results.csv
│   ├── top_200_ns1_interactors.csv
|   └── hdock_binding/
│       ├── NS1-AKT1/
|            ├── NS1- AKT1- model_1.pdb
│            ├── NS1-AKT1_protein_interaction.png
│            ├── ns1_human_interactome_results.csv
│            ├── top10_models.tar.gz
│            └── top_200_ns1_interactors.csv
|
│       ├── NS1-CA1/
|            ├── 3d Rendered transparency 0.3.png
│            ├── NS1 CA1 linkedin.png
│            ├── NS1- CA1.pdb
│            ├── NS1-CA1 protein Interaction.png
│            └── top10_models.tar.gz
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
pip install biopython pandas numpy tqdm requests openpyxl
```

---

### 2️) Download Required Data

Before running the notebook, ensure you have:
- Human PPI data from [STRING DB](https://string-db.org/) (score ≥ 700)
- AlphaFold structure files (downloaded automatically by the notebook)
- NS1 crystal structure: PDB ID `4O6B` (place in `data/structures/`)

---

### 3️) Run the Notebook

Open and run cells sequentially in `notebooks/DengueGNN_Master_File.ipynb`:

| Section | What it does |
|---|---|
| Data & Structure Download | Fetches AlphaFold PDB files for all unique proteins in STRING |
| Featurization (STRING proteins) | Builds ESM-2 + graph representations for ~717 unique STRING proteins, saves to `data/processed/` |
| Training | Trains the Siamese GATv2 model on STRING pairs, saves best checkpoint to `models/best_dengue_gnn.pth` |
| Proteome-Wide Featurization | Builds ESM-2 + graph representations for all 20,565 human proteins |
| Proteome Screening | Scores all 20,565 human proteins against NS1 |
| Results | Exports ranked CSV to `results/` |

> ⚠️ **Hardware note:** Trained and screened on an NVIDIA RTX 4050 (6GB VRAM).  
> Full proteome featurization + screening takes ~16 hours total.

---

### 4️) Pre-computed Results

If you want to explore the findings without re-running the full pipeline, all outputs are already provided in `results/`:

| File | Description |
|---|---|
| `ns1_human_interactome_results.csv` | Full proteome scores (20,565 proteins) |
| `top_200_ns1_interactors.csv` | Top 200 predicted interactors with gene names |
| `hdock_binding/NS1-AKT1/` | HDOCK docking models for NS1–AKT1 |
| `hdock_binding/NS1-CA1/` | HDOCK docking models for NS1–CA1 |


---

##  Why This Matters

This project demonstrates that:

* Viral-host interactome discovery can be performed without supercomputers
* Structural GNNs can identify biologically meaningful clusters
* Novel therapeutic hypotheses can emerge from AI-first discovery pipelines

---

##  License

MIT License

## Contact

Vinindu Manula Siriwardhana

https://www.linkedin.com/in/vinindu-siriwardhana/

vinidusiri@gmail.com

---

