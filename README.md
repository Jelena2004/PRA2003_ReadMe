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

Each event has the following format:
```text
<event_id> <number_of_molecules>
<px> <py> <pz> <molecule_id>
...
```

**Header row:** the first integer identifies the simulation event, while the second specifies how many molecular records belong to that event.

**Molecular records:** each subsequent row contains the momentum components in the x, y and z directions (px, py and pz), followed by the molecule ID.

**Molecule ID:** an integer code used to distinguish between different molecular species and their corresponding isotopes or ionic forms.

The file `output-Set0.txt` is used for the initial momentum calculation, while `output-Set1.txt` to `output-Set10.txt` are used for the further analysis. The Python scripts process these records to calculate momentum magnitudes, determine the frequencies of the twelve selected molecule IDs (grouped into six pairs), and estimate their statistical uncertainties and pair differences.

## Week 2- Calculating Momentum

**File:** `Week_2_deliverable.py`

In week 2, the task was to import data from a text file, retrieve molecule IDs and momentum components, convert the extracted strings into numerical values, and then finally compute the magnitude of each molecule's momentum. The momenta were calculated using the following formula:

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

where $N_m$ is the number of occurrences of the molecule and $N_{\mathrm{events}}$ is the number of events analysed.

Using Poisson statistics, the uncertainty is

$$
\sigma_{\bar N_m} = \frac{\sqrt{N_m}}{N_{\mathrm{events}}}.
$$

The script handles filename input, rejects empty input, and catches missing-file and file-access errors. However, it counts only events that contain molecule records instead of all event headers, so empty events may be missed. This is corrected in Week 4.

## Week 4- Sub-sampling Method

**File:** `Week_4_deliverable.py`

This step of the project reports how many molecules of each of twelve selected types are produced per event, together with an uncertainty, and checks whether the six particle/antiparticle pairs are produced equally often. The complete sample (5M events) is stored in ten files, `output-Set1.txt` to `output-Set10.txt`. Each file is treated as an independent subsample: it is analysed on its own, and the ten outcomes are then merged. The statistical uncertainty comes from how much the ten outcomes differ from one another (the sub-sampling method).

### Event parsing

Each file is read line by line:

- A line with two integers is an event header and counts as one event. Events with no molecule records are included in the event count.
- A line with four values (`px, py, pz, molecule ID`) is a molecule record. If its ID is one of the twelve selected IDs, it is added to that molecule's count.
- Any other line, or a molecule record before the first event header, stops the program with an error message.

For molecule type $m$ in subsample $i$, which contains $n_i$ events, the average number per event is

$$
\mu_{m,i} = \frac{C_{m,i}}{n_i},
$$

where $C_{m,i}$ is the number of molecules of type $m$ found in that file.

### Combining subsamples

To merge the ten subsample values, each one is multiplied by its event count and the products are summed and divided by the total number of events:

$$
\bar{\mu}_m = \frac{\sum_{i=1}^{K} n_i\,\mu_{m,i}}{\sum_{i=1}^{K} n_i}, \qquad K = 10.
$$

Weighting in this way lets larger files contribute more. When all files contain the same number of events, it reduces to the simple average of the ten values.

The uncertainty is taken from how far the ten subsample values scatter. It is calculated as the sample standard deviation (`np.std(..., ddof=1)`) around $\langle\mu_m\rangle$, the plain mean of the ten values:

$$
s_m = \sqrt{\frac{1}{K-1}\sum_{i=1}^{K}\left(\mu_{m,i} - \langle \mu_m \rangle\right)^2}.
$$

Here $s_m$ describes the scatter of the individual subsamples. The uncertainty of the combined value itself would be smaller, roughly $s_m/\sqrt{K}$, but the program reports $s_m$.

### Overall molecule-pair asymmetry

Six particle/antiparticle pairs are compared: `211/-211`, `321/-321`, `2212/-2212`, `3122/-3122`, `3312/-3312`, and `3334/-3334`.

For each pair, the positive and negative frequencies are subtracted separately in every subsample:

$$
\Delta_i = \mu_{+,i} - \mu_{-,i}.
$$

The ten differences are merged with the same event-count weights as above, giving the mean difference $\bar{\Delta}$. Its uncertainty is the sample standard deviation of the ten differences, where $\langle\Delta\rangle$ is their plain mean:

$$
s_\Delta = \sqrt{\frac{1}{K-1}\sum_{i=1}^{K}\left(\Delta_i - \langle \Delta \rangle\right)^2}.
$$

Subtracting inside each subsample first means that fluctuations affecting both molecules of a pair together (for example, a subsample with more activity overall) are reflected in $s_\Delta$. Combining the two single-molecule uncertainties separately would overlook this shared behaviour.

To judge how large each difference is relative to this scatter, the program also reports

$$
r = \frac{|\bar{\Delta}|}{s_\Delta}.
$$

This is printed as "Difference / uncertainty". It is a quick descriptive indicator, not a p-value or a formal test. If $s_\Delta = 0$, the ratio is set to 0 when $\bar{\Delta} = 0$ and to infinity otherwise.

### Output

The program prints two tables: the twelve molecule averages with their subsampling uncertainties, and the six pair differences with their uncertainties and ratios. It also prints the number of events in each subsample and the total number of events analysed, which can be checked against reference event totals.

### Week 4 results

These values come from the earlier version of the analysis, before the event-parsing change that includes empty events in the event count, so they may shift slightly when the current script is rerun.

| Molecule | Mean per event | Uncertainty (spread of ten subsamples) |
|------:|------------:|------------:|
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

| Pair | Mean difference per event | Spread | Ratio |
|------|-----------------------:|---------:|------:|
| 211 / -211   | 0.03229671 | 0.00451870 | 7.15  |
| 321 / -321   | 0.00569100 | 0.00329367 | 1.73  |
| 2212 / -2212 | 0.02387314 | 0.00237277 | 10.06 |
| 3122 / -3122 | 0.00490256 | 0.00058316 | 8.41  |
| 3312 / -3312 | 0.00044067 | 0.00048793 | 0.90  |
| 3334 / -3334 | 0.00003551 | 0.00006854 | 0.52  |

### Discussion of the results from week 4

Every pair has a positive mean difference, and the pairs fall into three groups. 2212/-2212 (ratio 10.06), 3122/-3122 (8.41) and 211/-211 (7.15) have differences that span many times the subsample scatter, so they are the most pronounced asymmetries in the data. 321/-321 (1.73) lies in between: its difference is positive but only modestly larger than its scatter. For 3312/-3312 (0.90) and 3334/-3334 (0.52) the difference is smaller than the scatter, so these two pairs are compatible with equal production at this level of precision. Since six pairs are compared and the ratio is not a formal test, these statements are descriptive.

## Week 5- Momentum-dependent asymmetry

**File:** `Week_5_deliverable.py`

Week 4 asked whether the six particle/antiparticle pairs are produced equally often over the whole sample. Week 5 asks a follow-up question: does the difference depend on how much momentum the molecules carry? A pair that looks symmetric overall could still be asymmetric at low momentum and compensate at high momentum, or the other way round, so the difference is now measured in separate momentum bins.

### Momentum and binning

For every selected molecule record, the momentum magnitude is computed from the three components in the file:

$$
P = \sqrt{p_x^2 + p_y^2 + p_z^2}.
$$

The values are sorted into 20 equal-width bins covering 0 to 20 (in units of $10^{-20}$ kg m/s), so each bin is 1 unit wide. Bins include their lower edge, and a value exactly equal to 20 is placed in the last bin. Molecules with $P$ outside 0 to 20 are not binned; each subsample reports how many were left out.

### Method

**1. Frequency per bin.** In subsample $i$, which has $n_i$ events, the average number of molecules of type $m$ per event in momentum bin $b$ is

$$
f_{m,i,b} = \frac{C_{m,i,b}}{n_i},
$$

where $C_{m,i,b}$ is the number of such molecules whose momentum lies in bin $b$.

**2. Paired difference in each bin.** As in Week 4, the two members of a pair are subtracted within the same subsample:

$$
\Delta_{i,b} = f_{+,i,b} - f_{-,i,b}.
$$

**3. Combining the subsamples.** For each bin, the mean difference over the ten subsamples and its uncertainty are

$$
\bar{\Delta}_b = \frac{1}{K}\sum_{i=1}^{K}\Delta_{i,b},
\qquad
s_b = \sqrt{\frac{1}{K-1}\sum_{i=1}^{K}\bigl(\Delta_{i,b}-\bar{\Delta}_b\bigr)^2},
\qquad K = 10 .
$$

The uncertainty is the sample standard deviation (`np.std(..., ddof=1)`), so it describes the scatter between subsamples, not the uncertainty of the mean itself. Unlike the Week 4 overall difference, this mean gives every subsample the same weight; the two agree when the subsamples have equal event counts.

**4. Ratio.** For each bin the program also reports

$$
r_b = \frac{|\bar{\Delta}_b|}{s_b}.
$$

This is a rough guide to how large a difference is compared with the scatter, not a formal significance. If $s_b = 0$, the ratio is 0 when $\bar{\Delta}_b = 0$ and infinite otherwise. This can happen in sparse bins, for example at high momentum for the rare pair 3334 / -3334.

### Output

For each pair the script saves one figure, `momentum_asymmetry_<positive ID>_<negative ID>.png`, giving six figures in total. Each figure plots the mean difference $\bar{\Delta}_b$ against momentum, with one point per bin placed at the bin centre. The same numbers are also saved as CSV files in `csv_results`.

To read a point: its height is the production difference in that momentum bin. Above the dashed zero line, the positive-ID molecule is the more common one; below it, the negative-ID molecule is. The vertical bar is $s_b$ and shows how far the ten subsample values spread around the mean. The horizontal bar only shows which momentum interval the bin covers and is not an error on $P$.

### Conclusion

The asymmetry is not uniform in momentum. Where it is clear (211, 2212 and 3122), it is concentrated at low momentum and becomes small at high momentum. The pair 321 / -321 shows only a small positive difference at higher momentum, in line with its weak overall result, while the pairs with no overall asymmetry (3312 and 3334) show nothing in any momentum bin. In the three pairs with a clear difference, the sign stays the same across the whole range, which supports a real effect rather than a feature of one bin.

These conclusions are descriptive. The uncertainties come from only ten subsamples, bins at high momentum contain few molecules, and many bins and pairs are examined, so isolated features should not be over-interpreted. Confirming a real asymmetry would require a formal statistical test that accounts for multiple comparisons.

## Usage

### Prerequisites

- Python 3.10 or newer
- A terminal on macOS, Linux or Windows
- The Python packages `numpy` and `matplotlib`
- The ten data files `output-Set1.txt` to `output-Set10.txt`

### Installing the packages

```bash
pip install numpy matplotlib
```

### How to run the project

1. **Get the files.** Download or clone the repository and open a terminal in the project folder.

2. **Add the data.** Place `output-Set1.txt` to `output-Set10.txt` in the same folder as the script. The program checks that all ten files are present before it starts and lists any that are missing.

3. **Run the analysis.**

   ```bash
   python Week_5_deliverable.py
   ```

4. **Close the plot windows.** Each of the six momentum plots opens in its own window. The program continues to the next plot once the current window is closed. Each plot is named:

 ```bash
   Figure_nr._particle pair.png
   ```

5. **Collect the results.** When the run finishes, the following are available:

   - **In the terminal:** the average frequency per event for each molecule, the six pair differences with their ratios, and the momentum-bin tables.
   - **Plots:** six images named `momentum_asymmetry_<positive ID>_<negative ID>.png`, saved in the project folder.
   - **CSV files:** the numerical results, saved in the folder `csv_results`.

### Troubleshooting

- Files not found: make sure you run the script from the folder that contains the data files, or move the data files into the folder you are running it from.
- "Invalid data on line ..." error: a file contains a line that is neither an event header nor a molecule record. Check that the data file is not corrupted or incomplete.
- Module not found: install the packages again with `pip install numpy matplotlib`.

#### Author: Jelena Stefanic (i6347709)
#### Course: PRA2003- Programming
