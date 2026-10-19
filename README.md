# 🚲 Spatiotemporal Evolving Patterns of Bike-Share Mobility Networks
*Empirical Replication and Methodological Extension of Wang et al. (2021) on Citi Bike Metropolitan Networks*

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![NetworkX](https://img.shields.io/badge/NetworkX-3.0%2B-orange.svg)](https://networkx.org/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.2%2B-F7931E.svg?logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![Folium](https://img.shields.io/badge/Folium-Interactive%20GIS-77B829.svg?logo=leaflet&logoColor=white)](https://python-visualization.github.io/folium/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Academic Paper](https://img.shields.io/badge/Physica%20A-Wang%20et%20al.%20(2021)-red.svg)](bike_sharing.pdf)
[![Exam Report](https://img.shields.io/badge/Exam%20Report-PDF%20Document-blue.svg)](REPORT_BIKE_SHARING.pdf)

---

## 📌 Executive Summary

This study implements a mathematically rigorous, fully explainable graph mining framework to analyze urban micro-mobility dynamics, replicating and extending the analytical pipeline formalised by **Wang, Kim et al. (2021)** (*Physica A: Statistical Mechanics and its Applications*, [DOI: 10.1016/j.physa.2021.126819](https://doi.org/10.1016/j.physa.2021.126819)).

While the original paper focused on dockless bike-sharing in Singapore, this investigation adapts the methodology to a docked metropolitan transit system: **Citi Bike (New York City / Jersey City)**, analyzing **23,318 real-world trips** from April 2021.

In compliance with white-box data mining principles, no deep learning or black-box estimators are used. All structural patterns, spatial hubs, functional partitions, and commuting corridors are derived deterministically using **spectral graph theory, linear algebra (Perron-Frobenius), Markov chain stationary distributions, and spatial network analysis**.

---

## 🔬 Methodological Pillars

```mermaid
flowchart TD
    A[Raw Citi Bike Trip Logs<br/>23,318 Rides] --> B[Three-Stage Quality Cleaning Pipeline]
    B -->|Bounding Box + Short Trip (<60s) + Haversine Speed (>35 km/h)| C[Directed Weighted Mobility Graph G=<V, E, W>]
    
    C --> D1[Custom Weighted PageRank<br/>Power Iteration with Destination-Attraction Teleport]
    C --> D2[Centrality Triangulation<br/>PageRank vs In-Strength vs Betweenness vs Closeness]
    C --> D3[Dual Community Detection<br/>Louvain vs Normalized Spectral Clustering L_sym]
    C --> D4[Spatial Flow Neighborhood Modeling<br/>k-NN Trajectory Graph & Emergent Corridors]
    
    D1 --> E[Hub Identification & Sensitivity Analysis on d]
    D2 --> E
    D3 --> F[Functional Urban Zoning & Partition Concordance]
    D4 --> G[Interactive Folium GIS Web Map]
```

### 1. Data Cleaning & Spatial Sanity
Following Section 3 of Wang et al. (2021), raw logs are processed through three sequential filters:
* **Spatial Bounding Box Filter:** Discarding coordinates outside $[40.5^\circ\text{N}, 41.0^\circ\text{N}] \times [-74.3^\circ\text{W}, -73.8^\circ\text{W}]$ and null station identifiers.
* **Short-Trip Truncation:** Removing rides $< 60$ s (false unlocks / defective bikes).
* **Speeding & Fleet Rebalancing Filter:** Excluding trips with average speed $>35$ km/h computed via the **Haversine formula**, isolating true human cycling from maintenance van rebalancing.

### 2. Weighted PageRank with Destination Attraction (Eq. 7 in the Paper)
Urban bike flows are strongly asymmetric. Following Eq. (7) of Wang et al., the transition matrix is asymmetric column-stochastic, and the teleportation vector $\mathbf{v}$ is personalized by destination-attraction (In-Strength):

$$\mathbf{p}^{(t+1)} = d \mathbf{M} \mathbf{p}^{(t)} + \left(1 - d + d \sum_{k \in \mathcal{D}} p_k^{(t)}\right) \mathbf{v}$$

where:
* $M_{ij} = \frac{W_{ji}}{s_j^{out}}$ represents flow transition probability from station $j$ to $i$;
* $v_i = \frac{s_i^{in}}{\sum_k s_k^{in}}$ captures intrinsic destination attraction;
* $\mathcal{D}$ denotes dangling nodes ($s_j^{out} = 0$), re-injecting probability mass without loss.

The formulation is implemented *from-scratch* via **Power Iteration** and cross-validated against NetworkX ($\Delta < 3 \times 10^{-6}$), with exponential convergence verified by the $L_1$ error decay.

### 3. Centrality Triangulation
Stations are characterized across four complementary structural dimensions:
1. **Weighted PageRank:** Confluence and systemic equilibrium attractor.
2. **In-Strength ($s_i^{in}$):** Gross incoming ride volume.
3. **Betweenness Centrality:** Critical bridging role along shortest topological paths.
4. **Closeness Centrality:** Geometric accessibility over an inverted-distance friction graph.

### 4. Dual Community Detection: Louvain vs Normalized Spectral Clustering
To delineate functional urban partitions, two orthogonal clustering paradigms are evaluated:
* **Louvain Modularity Optimization ($O(N \log N)$):** Greedy modularity maximization on weighted undirected projections.
* **Normalized Spectral Clustering ($O(N^3)$):** Eigendecomposition of the symmetric normalized Laplacian:
  $$L_{sym} = I - D^{-1/2} W D^{-1/2}$$
  followed by $K$-Means embedding in the subspace of the smallest non-trivial eigenvectors.

### 5. Spatial Flow Neighborhood Modeling & Corridor Identification
Adapting Section 4.1 of the paper, individual trip trajectories $\mathbf{f}_i = (x_i^{orig}, y_i^{orig}, x_i^{dest}, y_i^{dest})$ are embedded in $\mathbb{R}^4$. A $k$-Nearest Neighbors graph ($k=15$) captures flow spatial coherence. Louvain community detection on this trajectory graph extracts **24 emergent mobility corridors**, exported to an interactive GIS map.

---

## 📊 Key Empirical Findings

### Top Stations by Weighted PageRank vs Multi-Centrality Metrics

| Station ID | Station Name | PageRank | Betweenness | Closeness | In-Strength | Dominant Urban Function |
|:---:|:---|:---:|:---:|:---:|:---:|:---|
| **3186** | **Grove St PATH** | **0.066003** | **0.437657** | **13.287204** | 1,514 | Primary Multimodal Gateway |
| **3199** | **Newport Pkwy** | **0.062356** | 0.158521 | 11.219313 | 1,424 | Waterfront Residential Hub |
| **3192** | **Liberty Light Rail**| 0.056003 | 0.012531 | 11.810705 | 1,231 | Transit Feeder / Commuting Point |
| **3203** | **Hamilton Park** | 0.048712 | 0.211779 | 12.552644 | 1,096 | Mixed Residential Anchor |
| **3276** | **Marin Light Rail** | 0.045173 | 0.056078 | 12.096991 | 1,000 | Light Rail Confluence Hub |
| **3202** | **Newport PATH** | 0.044464 | 0.102757 | 11.587597 | 1,011 | Secondary Commuting Connector |
| **3792** | **Columbus Dr at Exchange Pl** | 0.043562 | 0.018484 | 10.902785 | 993 | Financial / Commercial Core |
| **3639** | **Harborside** | 0.037651 | 0.057644 | 11.169473 | 854 | Waterfront Commercial Gateway |
| **3185** | **City Hall** | 0.035823 | 0.017231 | 10.416205 | 809 | Civic / Administrative Hub |
| **3205** | **JC Medical Center** | 0.034497 | 0.000313 | 10.706582 | 772 | Specialized Service Destination |

> **Key Insight:** `Grove St PATH` dominates all four metrics simultaneously, operating as the single most vital bottleneck and multimodal transfer hub of the urban system. Stations like `Liberty Light Rail` have high PageRank but low betweenness, indicating destination terminus status rather than intermediary transit routing.

---

## 🗺️ Interactive GIS Map

The repository includes a standalone interactive Leaflet/Folium visualization:
* **File:** [`corridoi_mobilita_jersey_city.html`](corridoi_mobilita_jersey_city.html)
* **Features:** OpenStreetMap base layer, color-coded functional communities, proportional PageRank station nodes, and vector trajectories depicting the 24 primary cycling corridors with tooltips and trip volumes.

---

## 📂 Repository Structure

```text
.
├── REPORT_BIKE_SHARING.pdf           # Comprehensive academic exam report (14 pages, PDF)
├── bike_sharing_methodology.ipynb     # Main research notebook (fully executed & commented)
├── April2021.csv.zip                  # Raw Citi Bike trip records (compressed archive)
├── corridoi_mobilita_jersey_city.html # Interactive Folium GIS corridor visualization
├── bike_sharing.pdf                   # Reference scientific paper (Wang et al., 2021)
├── requirements.txt                   # Reproducible Python dependencies
├── .gitignore                         # Standard Python & Jupyter ignore patterns
├── LICENSE                            # MIT Open Source License (Alberto Mergoni)
└── README.md                          # Executive documentation & study briefing
```

---

## 🚀 Quickstart & Reproducibility

### 1. Clone the Repository
```bash
git clone https://github.com/Alby1224/bike-share-mobility-networks.git
cd bike-share-mobility-networks
```

### 2. Set Up Virtual Environment
```bash
python -m venv venv
# Windows:
.\venv\Scripts\activate
# Linux / macOS:
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Run the Methodology Notebook
```bash
jupyter notebook bike_sharing_methodology.ipynb
```
*Note: The notebook automatically reads `April2021.csv.zip` directly via Python's `zipfile` module without requiring manual decompression.*

---

## 📖 Scientific References

1. **Wang, J., Kim, I., et al. (2021).** *Spatiotemporal evolving patterns of bike-share mobility networks and their associations with land-use conditions before and after the COVID-19 outbreak.* **Physica A: Statistical Mechanics and its Applications**, 586, 126819. [DOI: 10.1016/j.physa.2021.126819](https://doi.org/10.1016/j.physa.2021.126819)
2. **Citi Bike NYC / Jersey City.** Open Trip Data (April 2021). Motivate / Lyft Urban Mobility.

---

## 📄 License
This project is released under the **MIT License**. See [LICENSE](LICENSE) for full legal text.
