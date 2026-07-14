# Monte Carlo Parameter Estimation of the Li–Rinzel Calcium Oscillation Model

This project estimates Li–Rinzel calcium-model parameters from experimentally measured calcium concentration data. The model is simulated numerically, compared with the experimental signal, and fitted using Monte Carlo and systematic parameter searches.

## Project objective

The experimental calcium signal contains measurement noise and does not exactly match the standard model output. The goal is to identify parameter values that produce a simulated signal with similar:

- Time-domain behavior
- Oscillation frequency
- Statistical distribution

The main parameters investigated are `k3`, `v2`, `d5`, and `a2`.

## Model background

The Li–Rinzel model is a two-variable nonlinear dynamical system describing intracellular calcium oscillations. The state variables are:

- `Ca`: cytosolic calcium concentration
- `h`: IP₃-receptor inactivation variable

The governing equations are:

\[
\frac{dCa}{dt}=J_{chan}+J_{leak}-J_{pump}
\]

\[
\frac{dh}{dt}=\alpha_h(1-h)-\beta_hh
\]

The calcium fluxes are:

\[
J_{chan}=c_1v_1p^3n^3h^3(Ca_{ER}-Ca)
\]

\[
J_{leak}=c_1v_2(Ca_{ER}-Ca)
\]

\[
J_{pump}=v_3\frac{Ca^2}{Ca^2+k_3^2}
\]

where:

\[
p=\frac{IP_3}{IP_3+d_1}, \qquad n=\frac{Ca}{Ca+d_5}
\]

\[
Ca_{ER}=\frac{c_0-Ca}{c_1}
\]

The model is integrated using the explicit Euler method:

\[
Ca_{t+1}=Ca_t+\Delta t\frac{dCa}{dt}
\]

\[
h_{t+1}=h_t+\Delta t\frac{dh}{dt}
\]

## Computational workflow

```text
Load experimental calcium data and time values
        ↓
Remove invalid values and sort by time
        ↓
Convert milliseconds to seconds
        ↓
Correct the baseline using the first 20% of the data
        ↓
Normalize the signal using Min–Max scaling
        ↓
Simulate the Li–Rinzel model using Euler integration
        ↓
Interpolate the simulation onto the experimental time grid
        ↓
Calculate RMSE, KL divergence, and frequency difference
        ↓
Search the parameter space
        ↓
Visualize and compare the fitted signal
```

## Data preprocessing

The notebook:

1. Loads the fourth column of `3.xlsx` as the experimental calcium signal.
2. Loads the experimental time values.
3. Removes NaN and infinite values.
4. Sorts the data according to increasing time.
5. Converts time from milliseconds to seconds when necessary.
6. Shifts the time vector so that it begins at zero.
7. Estimates the baseline using the median of the first 20% of the signal.
8. Subtracts the baseline.
9. Applies Min–Max normalization:

\[
x_{norm}=\frac{x-\min(x)}{\max(x)-\min(x)}
\]

The provided dataset contains 249 valid samples covering approximately 307.47 seconds, with a median sampling interval of approximately 1.202 seconds.

## Parameter estimation methods

### Monte Carlo search

The primary search randomly samples 900 parameter combinations:

```text
k3 ~ Uniform(0.05, 0.15)
v2 ~ Uniform(0.05, 0.15)
d5 ~ Uniform(0.05, 0.15)
a2 ~ Uniform(0.10, 0.30)
```

For every sample, the code simulates the model, normalizes and interpolates the signal, detects peaks, calculates frequency difference, Gaussian KL divergence, and RMSE, then stores the result.

The main Monte Carlo score is:

\[
Score=|f_{exp}-f_{sim}|+D_{KL}(P\parallel Q)
\]

Lower values indicate a better combined frequency and distributional match.

### Systematic grid search

The notebook also evaluates a 30 × 30 grid of `k3` and `v2` values, giving 900 grid locations. At each grid location, `d5` and `a2` are randomly selected. This is therefore a hybrid grid-random search rather than a complete four-dimensional grid search.

## Evaluation metrics

### RMSE

\[
RMSE=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(x_{exp,i}-x_{sim,i})^2}
\]

RMSE measures pointwise error between the experimental and simulated signals.

### Gaussian KL divergence

The signals are fitted to Gaussian distributions using maximum likelihood estimation:

\[
D_{KL}(P\parallel Q)=
\ln\left(\frac{\sigma_q}{\sigma_p}\right)+
\frac{\sigma_p^2+(\mu_p-\mu_q)^2}{2\sigma_q^2}-\frac{1}{2}
\]

KL divergence measures the difference between the overall distributions of the two signals.

### Frequency difference and FFT

Peak frequency is estimated from the average peak-to-peak interval:

\[
f=\frac{1}{\text{mean peak-to-peak time}}
\]

The notebook also uses the real Fast Fourier Transform to calculate the dominant frequency as an independent frequency-domain check.

## Reported results

### Best Monte Carlo result

```text
k3 = 0.05578
v2 = 0.05500
d5 = 0.06379
a2 = 0.14200

RMSE                    = 0.233024
Gaussian KL divergence  = 3.846850
Peak-frequency difference = 0.004669
Combined score           = 3.851519
```

Before-and-after comparison:

```text
                         RMSE       Gaussian KL
Before parameter fit     0.239715   25.266442
After Monte Carlo fit    0.233024    3.846850
```

### Best grid-search result

```text
k3 = 0.06034
v2 = 0.06724

RMSE                    = 0.218576
Gaussian KL divergence  = 4.220354
Peak-frequency difference = 0.000874
```

The notebook's separate RMSE-only comparison reports:

```text
Best Monte Carlo RMSE = 0.19750
Best grid RMSE        = 0.19735
```

These values differ from the combined-objective result because the Monte Carlo optimization uses frequency difference plus KL, while the final comparison selects the minimum RMSE.

## Visualizations

The notebook produces:

- Raw, baseline-corrected, and normalized signal plots
- Deterministic Li–Rinzel simulation plots
- Monte Carlo RMSE heatmaps
- 3D RMSE and objective surfaces
- KL-divergence parameter-space maps
- Before-versus-after time-series plots
- Gaussian PDF and KL-integrand plots
- Time-resolved KL-divergence plots
- Gamma-distribution comparison plots
- FFT spectra in linear and logarithmic scales
- An animation of the Monte Carlo search process

## Repository files

| File | Description |
|---|---|
| `AM_FINAL_PROJECT.ipynb` | Complete Python implementation and visualizations |
| `3.xlsx` | Experimental calcium-concentration data |
| `3_k3_V2.docx` | Project problem statement and initial model description |
| `GROUP_9_pptx.pdf` | Project presentation |

## Requirements

```text
numpy
pandas
matplotlib
scipy
scikit-learn
openpyxl
jupyter
```

Install the dependencies:

```bash
pip install numpy pandas matplotlib scipy scikit-learn openpyxl jupyter
```

Run the notebook:

```bash
jupyter notebook AM_FINAL_PROJECT.ipynb
```

## Important setup note

The notebook currently expects a separate `time.xlsx` file at:

```python
/content/drive/MyDrive/Applied_Mathematics/time.xlsx
```

To run locally, place the time file in the project directory and update `data_path` and `time_path`. The current repository includes `3.xlsx`, but does not include `time.xlsx`.

## Limitations and possible improvements

- The main objective uses KL divergence and frequency difference, but RMSE is not included in that score.
- Gaussian KL compares overall value distributions and does not directly measure temporal alignment.
- Peak-based and FFT-based frequency estimates are different measurements.
- The grid-search section randomly varies `d5` and `a2`, so it is a hybrid strategy.
- Explicit Euler integration could be replaced with a higher-accuracy or adaptive ODE solver.
- The parameter ranges are selected manually.
- A weighted objective containing RMSE, KL divergence, and frequency difference could provide a more balanced fit.
- Repeating Monte Carlo with multiple random seeds would quantify search uncertainty.

## Conclusion

This project demonstrates how a nonlinear biological ODE model can be calibrated against noisy experimental data using numerical simulation, signal preprocessing, stochastic parameter search, statistical comparison, and frequency analysis.
