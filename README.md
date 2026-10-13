# PRA2003: Week 7- Updated README File

## Project Overview: Chemistry Theme
This project applies Python to investigate simulated molecular species formed during combustion at high temperatures. The analysis is carried out in several stages, beginning with importing molecular data and calculating momentum, followed by examining the occurrence of different molecules and estimating their statistical uncertainties. The final stage compares molecular species and isotope pairs, with particular attention to how their differences depend on momentum.

The dataset is divided into individual simulation events, each consisting of a header and a series of molecular records. Every molecule is represented by an integer ID and three momentum components corresponding to the x, y, and z directions. The investigation concentrates on twelve specific IDs, grouped into six pairs for comparison.

### Molecules investigated

| Molecule ID | Molecule type |
|---:|---|
| 211 | Carbon monoxide |
| -211 | Carbon-13 monoxide |
| 321 | Nitric oxide |
| -321 | Ionised NO |
| 2212 | Water |
| -2212 | Heavy water (D2O) |
| 3122 | Methane (CH4) |
| -3122 | Methyl ion (CH3-) |
| 3312 | Ethylene (C2H4) |
| -3312 | Ionised ethylene (C2H3-) |
| 3334 | Ozone (O3) |
| -3334 | Superoxide anion (O2-) |
