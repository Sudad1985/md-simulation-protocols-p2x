# MD Simulation Input Files — P2X Receptors

This document provides example **AMBER MD simulation input files** for **P2X receptor models** embedded in a **DMPC membrane** and solvated with **TIP3P water**. The workflow covers **energy minimisation**, **heating**, **equilibration**, **production**, and **post-processing** (trajectory analysis + **MM/PBSA / MM/GBSA** setup).

> Compatible with **AMBER 14+** and **AmberTools** (GPU examples use `pmemd.cuda`).

---

## 1) Energy Minimisation

**File:** `min.in`

```fortran
 &cntrl
  imin=1, maxcyc=10000,
  ncyc=5000, ntb=1, ntp=0,
  ntf=1, ntc=1, ntpr=50,
  ntwr=2000, cut=10.0,
 /
```

Purpose: relax solvent/lipids and remove steric clashes before dynamics.

---

## 2) Heating (0 → 303 K)

**File:** `heat.in`

```fortran
 &cntrl
  imin=0, ntx=1, irest=0,
  ntb=1, cut=10.0,
  ntt=3, gamma_ln=1.0,
  tempi=0.0, temp0=303.0,
  ntc=2, ntf=2,
  nstlim=25000, dt=0.002,
  ntpr=500, ntwx=500, ntwr=5000,
  ntr=1, restraint_wt=10.0,
  restraintmask='!:WAT & !:Na+ & !:Cl-'
 /
```

Purpose: gentle heating with backbone restraints to preserve structure while solvent equilibrates.

---

## 3) Equilibration (NPT, 303 K)

**File:** `equil.in`

```fortran
 &cntrl
  imin=0, ntx=5, irest=1,
  ntb=2, cut=10.0,
  ntt=3, gamma_ln=1.0,
  temp0=303.0,
  ntp=1, taup=2.0,
  ntc=2, ntf=2,
  nstlim=250000, dt=0.002,
  ntpr=500, ntwx=500, ntwr=5000,
  ntr=1, restraint_wt=1.0,
  restraintmask='!:WAT & !:Na+ & !:Cl-'
 /
```

Purpose: density/pressure stabilisation, relax restraints before production.

---

## 4) Production MD (NPT, 303 K)

**File:** `prod.in`

```fortran
 &cntrl
  imin=0, ntx=5, irest=1,
  ntb=2, cut=10.0,
  ntt=3, gamma_ln=1.0,
  temp0=303.0,
  ntp=1, taup=2.0,
  ntc=2, ntf=2,
  nstlim=2500000, dt=0.002,
  ntpr=500, ntwx=500, ntwr=5000
 /
```

Typical lengths: 100–500 ns (adjust `nstlim`). Output trajectory: `prod.nc`.

---

## 5) Example Run Commands

GPU production:
```bash
pmemd.cuda -O -i prod.in -o prod.out   -p P2X7.prmtop -c equil.rst   -r prod.rst -x prod.nc
```

CPU production:
```bash
pmemd -O -i prod.in -o prod.out   -p P2X7.prmtop -c equil.rst   -r prod.rst -x prod.nc
```

---

## 6) Trajectory Analysis (cpptraj quick-start)

Example: RMSD, strip waters/ions, write a fitted trajectory.
```bash
cpptraj P2X7.prmtop <<'EOF'
trajin prod.nc
autoimage
rms ToFirst :1-999@CA
strip :WAT,Na+,Cl-
trajout prod_stripped.nc
EOF
```

---

## 7) MM/PBSA and MM/GBSA Setup

### 7.1 Generate Topologies for Complex / Receptor / Ligand

From the **solvated complex** topology (`P2X7_solv.prmtop`) use `ante-MMPBSA.py` to create stripped topologies:

```bash
# Complex (protein + ligand, no waters/ions)
ante-MMPBSA.py -p P2X7_solv.prmtop -c complex.prmtop   -s ':WAT,Na+,Cl-'

# Receptor (protein only)
ante-MMPBSA.py -p P2X7_solv.prmtop -r receptor.prmtop   -s ':WAT,Na+,Cl-,:LIG'

# Ligand only (uses ligand residue name, e.g., LIG)
ante-MMPBSA.py -p P2X7_solv.prmtop -l ligand.prmtop   -s ':WAT,Na+,Cl-,:1-999'   # adjust receptor residue mask as needed
```

> **Note:** Replace `:LIG` and `:1-999` with your actual ligand and receptor masks.

### 7.2 MM/PBSA & MM/GBSA Input File

**File:** `mmpbsa.in`

```bash
&general
  startframe=1, endframe=1000, interval=10,
  verbose=1, keep_files=2,
/
&gb
  igb=5, saltcon=0.100,
/
&pb
  istrng=0.100,
/
# Optional per-residue decomposition (uncomment to use)
#&decomp
#  idecomp=1,
#/
```

### 7.3 Run MMPBSA.py

```bash
MMPBSA.py -O   -i mmpbsa.in   -o FINAL_RESULTS_MMPBSA.dat   -sp P2X7_solv.prmtop   -cp complex.prmtop   -rp receptor.prmtop   -lp ligand.prmtop   -y prod.nc
```

Tips:
- Adjust `startframe`/`endframe`/`interval` to sample equilibrated frames only.
- For long trajectories, consider **stripping** water/ions for speed (already done by topologies) and **downsampling** with `interval`.
- Add `-deo decomp.dat` if using per-residue decomposition.

---

## 8) Practical Notes

- **System:** protein in **DMPC** membrane + **TIP3P** water.
- **Constraints:** SHAKE on bonds to H; 2 fs timestep.
- **Thermostat/barostat:** Langevin at 303 K; NPT with isotropic pressure coupling (Berendsen or Monte Carlo).
- **Replicates:** run ≥2 independent replicas with different random seeds for robustness.
- **Hardware:** GPU (`pmemd.cuda`) recommended for production.

---

## 9) Citation Pointers

- AMBER 14/16 Manuals (Case et al.)  
- Roe & Cheatham, JCTC 2013 — cpptraj  
- Huang et al., PLOS ONE 2014 — P2X receptor MD

---
