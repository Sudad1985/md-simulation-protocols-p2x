# MD Simulation Protocols for P2X Receptors

This repository contains **molecular dynamics (MD) simulation input files and protocol** for simulating **human P2X receptors** (e.g., hP2X7R, hP2X1R) embedded in a **DMPC membrane** with **TIP3P water**, plus **MM/PBSA & MM/GBSA** input for binding free-energy estimation.

## 📦 Contents
```
md-simulation-protocols-p2x/
├── MD_simulation_protocol.md
├── min.in          # energy minimisation
├── heat.in         # heating 0 → 303 K
├── equil.in        # NPT equilibration
├── prod.in         # production MD
└── mmpbsa.in       # MM/PBSA & MM/GBSA input for MMPBSA.py
```

## 🧰 Software
- AMBER 14+ / AmberTools (pmemd / pmemd.cuda, cpptraj, MMPBSA.py)
- GPU acceleration recommended for production (pmemd.cuda).

## 🚀 Quick Start
1. Minimise (`min.in`), heat (`heat.in`), equilibrate (`equil.in`), produce (`prod.in`).
2. Generate stripped topologies for complex/receptor/ligand with `ante-MMPBSA.py`.
3. Run `MMPBSA.py` with `mmpbsa.in` over your production trajectory (`prod.nc`).

## 📚 References
- Case et al., AMBER 14/16 Manuals  
- Roe & Cheatham, JCTC 2013 — cpptraj  
- Huang et al., PLOS ONE 2014 — P2X receptor MD

---

⭐ *Adapt the masks and residue ranges to your specific systems (ligand name, protein residue count, etc.).*
