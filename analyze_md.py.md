#!/usr/bin/env python3
"""
Quick Python utility for post-processing MD simulations.
- Reads MMPBSA results
- Plots free energy values
"""

import pandas as pd
import matplotlib.pyplot as plt

# Load MMPBSA results
df = pd.read_csv("FINAL_RESULTS_MMPBSA.dat", delim_whitespace=True, comment='#')

# Show columns and summary
print("Columns:", df.columns.tolist())
print(df.describe())

# Optional plot of total energy
if "DELTA TOTAL" in df.columns:
    plt.figure()
    df["DELTA TOTAL"].plot(title="MM/PBSA Free Energy", ylabel="ΔG (kcal/mol)")
    plt.xlabel("Frame")
    plt.tight_layout()
    plt.savefig("MMPBSA_energy_plot.png")
    print("Plot saved as MMPBSA_energy_plot.png")
else:
    print("No 'DELTA TOTAL' column found in MMPBSA output.")
