# PRA2003: Week 7- Updated README File

## Project Description
### Chemistry-themed project
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

## Week 1- Research Questions
**File:** `Week_1_deliverable.py`

The project aims to answer the following three questions:

1. What are the average counts of each molecular species and their statistical uncertainties?
2. Is there any asymmetry between the normal and the variant molecule?
3. Is there any asymmetry as a function of their momentum?

## Dataset, Input and File Structure
The input data consist of text files containing simulated molecular records organised into individual events. Each event begins with a header containing two integers: the event ID and the total number of molecules recorded in that event. The header is followed by the corresponding molecular records, with each row containing three momentum components and an integer identifying the molecule.

Each event follows the following format:
```text
<event_id> <number_of_molecules>
<px> <py> <pz> <molecule_id>
...
```

Header Row: The first integer identifies the simulation event, while the second specifies how many molecular records belong to that event.

Molecular records: Each subsequent row contains the momentum components in the x, y and z directions ((px), (py) and (pz)), followed by the molecule ID.

Molecule ID: An integer code used to distinguish between different molecular species and their corresponding isotopes or ionic forms.

The file output-Set0.txt is used for the initial momentum calculation, while output-Set1.txt to output-Set10.txt are used for further analysis. The Python scripts examine twelve molecule IDs grouped into six pairs, calculating their frequencies, statistical uncertainties, and momentum differences.
The Python scripts process these records to calculate molecular momentum magnitudes, determine the frequencies of selected molecular species, and estimate statistical uncertainties.

## Week 2- Calculating Momentum

**File:** `Week_2_deliverable.py`

In week 2, the task was to import data from a text file, retrieve molecule IDs and momentum components, convert the extracted strings into numerical values, and then finally compute the magnitude of each molecule’s momentum. The momenta were calculated using the following formula:

$$
p = \sqrt{p_x^2 + p_y^2 + p_z^2}.
$$

The program outputs a table showing each molecule ID alongside its calculated momentum magnitude. This provides the foundation for the data extraction and momentum calculations required in subsequent analyses. However, the script ignores the initial header and assumes that every remaining line contains four molecular values. Therefore, it cannot correctly process files containing multiple events without a more flexible parser that recognises each event header.

## Week 3- Calculating Average and Uncertainty

**File:** `Week_3_deliverable.py`

The program takes an input file name, counts occurrences of twelve selected molecule IDs, and computes the mean number of molecules per event with its Poisson uncertainty. Results for all twelve species are printed in a table.

For molecule ID $m$:

$$
\bar N_m = \frac{N_m}{N_{\mathrm{events}}},
$$

$N_m$ : Number of occurrences of a given molecule

$N_{\mathrm{events}}$ : Number of events analysed

Therefore, using Poisson statistics the uncertainty can be calculated as follows:

$$
\sigma_{\bar N_m} = \frac{\sqrt{N_m}}{N_{\mathrm{events}}}.
$$

The script handles filename input, rejects empty input, and catches missing-file and file-access errors. However, it counts only events that contain molecule records instead of all event headers, so empty events may be missed.

## Week 4- Sub-sampling Method

**File:** `Week_4_deliverable.py`

This step of the project reports how many molecules of each of twelve selected types are produced per event, together with an uncertainty, and checks whether the six particle/antiparticle pairs are produced equally often. The complete sample (5M events) is stored in ten files, `output-Set1.txt` to `output-Set10.txt`. Each file is treated as an independent subsample: it is analysed on its own, and the ten outcomes are then merged. The statistical uncertainty comes from how much the ten outcomes differ from one another (the sub-sampling method).

### Event parsing

Each file is read line by line:

- A line with two integers is an event header and counts as one event. Events with no molecule records are included in the event count.
- A line with four values (`px, py, pz, molecule ID`) is a molecule record. If its ID is one of the twelve selected IDs, it is added to that molecule's count.
- Any other line, or a molecule record before the first event header, stops the program with an error message.

The average number of molecule *m* per event in subsample *i* is

$$
\bar N_{m,i} = \frac{\text{number of molecules of type } m \text{ in subsample } i}{n_i},
$$

where $n_i$ is the event count in subsample $i$.

### Combining subsamples

The combined average is weighted by the number of events in each subsample:

$$
\bar N_m = \frac{\sum_{i=1}^{K} n_i \bar N_{m,i}}{\sum_{i=1}^{K} n_i}, \qquad K = 10.
$$

As a result, bigger subsamples have more weight, but if all subsamples have equal event counts, then the weighted and unweighted means are in congruence.

The subsampling uncertainty is the sample standard deviation of the ten subsample averages, calculated with the formula:

$$
s_m = \sqrt{\frac{1}{K-1}\sum_{i=1}^{K}\left(\bar N_{m,i} - \langle \bar N_m \rangle\right)^2}.
$$

This measures the spread between subsample results. It is noteworthy, this is not the standard error of the combined mean.

### Overall molecule-pair asymmetry

The following six particle/antiparticle pairs are examined: `211/-211`, `321/-321`, `2212/-2212`, `3122/-3122`, `3312/-3312`, and `3334/-3334`.

The difference is calculated within each subsample:

$$
D_i = \bar N_{+,i} - \bar N_{-,i}.
$$

The uncertainty of the ten paired differences is given by:

$$
\sigma_D = \mathrm{SD}(D_1, D_2, \ldots, D_K).
$$

To allow for the comparison in size of each obtained difference with subsample spread, the following is also calculated: 

$$
R = \frac{|\bar D|}{\sigma_D}.
$$

This is referred to as the "Difference / uncertainty". It is used as a descriptive difference-to-spread ratio, not as a formal Gaussian significance or hypothesis-test result. If $\sigma_D = 0$, the ratio is set to 0 when $\bar D = 0$ and to infinity otherwise.

### Output

The program prints two tables: the twelve molecule averages with their subsampling uncertainties, and the six pair differences with their uncertainties and ratios. It also prints the number of events in each subsample and the total number of events analysed, which can be checked against reference event totals.

### Week 4 results

The results presented below are from before the latest changes regarding event parsing were performed.

| Molecule ID | Average per event | Subsampling uncertainty |
|------------:|------------------:|------------------------:|
| 211   | 19.94950794 | 0.03274619 |
| -211  | 19.91721122 | 0.03187334 |
| 321   | 2.50914781  | 0.00477369 |
| -321  | 2.50345680  | 0.00550369 |
| 2212  | 1.20803410  | 0.00189633 |
| -2212 | 1.18416095  | 0.00241216 |
| 3122  | 0.27659851  | 0.00107440 |
| -3122 | 0.27169595  | 0.00098538 |
| 3312  | 0.03944116  | 0.00028368 |
| -3312 | 0.03900049  | 0.00040248 |
| 3334  | 0.00118710  | 0.00004168 |
| -3334 | 0.00115158  | 0.00005079 |

| Molecule pair | Average difference per event | Uncertainty | Difference / uncertainty |
|---------------|-----------------------------:|------------:|-------------------------:|
| 211 vs -211   | 0.03229671 | 0.00451870 | 7.15  |
| 321 vs -321   | 0.00569100 | 0.00329367 | 1.73  |
| 2212 vs -2212 | 0.02387314 | 0.00237277 | 10.06 |
| 3122 vs -3122 | 0.00490256 | 0.00058316 | 8.41  |
| 3312 vs -3312 | 0.00044067 | 0.00048793 | 0.90  |
| 3334 vs -3334 | 0.00003551 | 0.00006854 | 0.52  |

### Discussion of the results from week 4

The differences between the 211/-211, 2212/-2212, and 3122/-3122 pairs are big when compared to the observed subsample spread, meaning they exhibit the strongest descriptive asymmetries.

For 3312/-3312 and 3334/-3334, the ratios (0.90 and 0.52) are below 1, indicating that the differences fall within the reported spread and suggest approximate symmetry, while for 321/-321, the difference is 1.73 times the uncertainty, suggesting weaker evidence of a consistent difference.

## Week 5- Momentum-dependent asymmetry


