# EBA 3650 — Carbon Tax Revenue Recycling
**Group 6 | Puskar Raj Acharya, Peien Chen, Qianqian Cui**

**Research Question:** Which revenue recycling mechanism — universal lump-sum redistribution or targeted compensation — more effectively mitigates the regressive effects of carbon taxation, and what is the associated efficiency cost in terms of aggregate labour supply?

---

## Overview

Carbon taxation reduces greenhouse gas emissions but places a disproportionate burden on lower-income households, who spend a larger share of income on the dirty consumption good. Once the government collects tax revenue, the design of how that revenue is returned to households matters as much as the tax rate itself.

This project builds a **static heterogeneous-agent general equilibrium model** in Python and compares two recycling schemes:

- **Scheme A — Lump-sum:** every household receives the same transfer T = R / N
- **Scheme B — Targeted:** transfers decrease with income rank, scaled so the budget balances

Both schemes are evaluated on welfare change by income decile, carbon tax burden share, change in clean-good consumption, equity–emissions trade-offs, and efficiency cost measured by average welfare across the distribution.

---

## Repository Structure

```
carbon_tax_project/
│
├── main.ipynb          <- Complete working notebook (run this)
├── README.md           <- This file
```

---

## Model

### Households

The economy has N = 1,000 households indexed by wage w_i, drawn from a log-normal distribution and normalised to mean 1. Each household has a preference parameter gamma_i (expenditure share on the dirty good x) that decreases with income rank, plus small idiosyncratic noise.

Each household solves:

```
max  U = gamma * log(x) + (1 - gamma) * log(y) + alpha * log(1 - l)
s.t.
     x = gamma * m / p_x
     y = (1 - gamma) * m
     m = w_i * l + T_i
     p_x = 1 + tau
```

where:
- `x`       — dirty consumption good (taxed at rate tau)
- `y`       — clean consumption good (numeraire)
- `l`       — labour supply on (0, 1)
- `tau`     — carbon tax rate (ad valorem on x)
- `gamma_i` — expenditure share on x (higher for lower-income households)
- `w_i`     — household wage
- `T_i`     — government transfer received
- `alpha`   — leisure preference parameter

### Analytical Solution (First-Order Condition)

The FOC for labour gives a closed-form solution:

```
w / (w * l + T) = alpha / (1 - l)

=>  l* = (w - alpha * T) / (w * (1 + alpha))
```

Optimal spending then follows from the Cobb-Douglas demands given full income m = w * l* + T.

### Government Budget Constraint

The government recycles all carbon tax revenue back to households:

```
R = tau * phi * sum(x_i)
```

The equilibrium transfer is found by fixed-point iteration inside the GE solver, converging when both the equilibrium wage and transfer array change by less than tol = 1e-6.

### Labour Market

Firm labour demand:

```
L_d(w) = 0.8 / (w + 0.5)
```

The equilibrium wage w_eq clears the labour market (L_supply = L_demand). With T = 0, the closed-form FOC implies l* = 1 / (1 + alpha) for every household, so labour supply is flat and independent of the wage.

### Redistribution Schemes

|               | Scheme A (lump-sum)          | Scheme B (targeted)                                   |
|---------------|------------------------------|-------------------------------------------------------|
| Transfer rule | T_i = R / N (same for all)   | T_i = T_bar * (1 + lambda * (1 - rank_i)), normalised to R |
| Targeting     | None                         | Higher transfers to lower-income ranks                |
| Baseline lambda | —                          | 1.75                                                  |

---

## Calibration Parameters

| Parameter            | Symbol        | Baseline | Notes                                  |
|----------------------|---------------|----------|----------------------------------------|
| Carbon tax rate      | tau           | 0.20     | Ad valorem on dirty good x             |
| Leisure preference   | alpha         | 3.0      | Higher = stronger preference for leisure |
| Min gamma            | gamma_min     | 0.20     | Expenditure share on x, top earners    |
| Max gamma            | gamma_max     | 0.65     | Expenditure share on x, bottom earners |
| Targeting multiplier | lambda_target | 1.75     | Controls redistribution intensity      |
| Revenue scaling      | phi           | 1.0      | Fraction of tax revenue recycled       |
| Number of households | N             | 1,000    |                                        |
| Convergence tol      | tol           | 1e-6     |                                        |
| Max GE iterations    | max_iter      | 200      |                                        |

---

## Notebook Structure (`main.ipynb`)

Run all cells from top to bottom. All sections depend on the parameters and functions defined in Sections 1–4.

| Section | Content |
|---------|---------|
| 1  | Imports and parameters |
| 2  | Household setup: wage simulation, rank assignment, gamma distribution |
| 3  | Utility function and closed-form household solver |
| 4  | Firm labour demand, GE solver, helper functions (`add_utility`, `add_deciles`) |
| 5  | Labour supply vs. net wage graph |
| 6  | Scenario runs at tau = 0.20 (no-tax baseline, tax-only, lump-sum, targeted) |
| 7  | Carbon tax burden by decile (tax-only scenario) |
| 8  | Welfare change by decile relative to no-tax baseline |
| 9  | Change in clean-good consumption (y) by income rank, binned |
| 10 | Equity–emissions trade-off over tau grid [0.00, 0.10, 0.20, 0.30, 0.40] |
| 11 | Efficiency cost of recycling schemes over finer tau grid |
| 12 | Labour market equilibrium: supply and demand curves |

---

## Output Figures

| Section | Title                                      | Key Message                                                        |
|---------|--------------------------------------------|--------------------------------------------------------------------|
| 5       | Labour Supply and Net Wage                 | With T = 0, l* is constant: 1 / (1 + alpha)                       |
| 7       | Carbon Tax Burden by Decile                | Raw regressivity: lower deciles bear a higher burden share         |
| 8       | Welfare Change by Decile                   | Central result: targeted transfers benefit lower deciles more      |
| 9       | Change in Clean-Good Consumption (Binned)  | Redistribution shifts consumption toward y for low-income ranks    |
| 10      | Equity–Emissions Trade-off                 | Higher tau reduces emissions but targeted scheme gains equity faster |
| 11      | Efficiency Cost of Recycling Schemes       | Average welfare effect of each scheme across the tau range         |
| 12      | Labour Market Equilibrium                  | Flat supply intersects downward-sloping demand at w_eq             |

---

## Setup and Requirements

### Python Version

Python 3.10 or higher is recommended.

### Dependencies

```
numpy >= 1.24
pandas >= 2.0
matplotlib >= 3.7
```

Install with:

```bash
pip install numpy pandas matplotlib
```

### How to Run

1. Open a terminal in the project folder and launch Jupyter:

```bash
jupyter notebook main.ipynb
```

2. Run all cells from top to bottom using **Kernel > Restart & Run All**.

3. To change any model parameter (tax rate, targeting multiplier, sample size), edit the constants at the top of **Section 1**. All downstream cells read from those variables.

---

## Key Results (Baseline: tau = 0.20, lambda = 1.75, N = 1,000)

**Equilibrium wages across scenarios:**

| Scenario             | w_eq (approx) |
|----------------------|---------------|
| No-tax baseline      | converged     |
| Tax-only (no recycle)| converged     |
| Lump-sum             | converged     |
| Targeted             | converged     |

*(Exact values printed in Section 6 when the notebook is run.)*

### Qualitative Findings

1. **Without redistribution**, the carbon tax is regressive: lower-income deciles, who have higher gamma (larger share of spending on the dirty good), bear a disproportionately large burden relative to their labour income.

2. **Scheme A (lump-sum rebate)** reduces regressivity. Because all households receive the same nominal transfer, lower-income households come out ahead in net terms.

3. **Scheme B (targeted transfers)** produces stronger redistribution for low-income ranks. Bottom households receive a transfer above the mean (scaled by lambda = 1.75), leading to larger welfare gains and a bigger increase in clean-good consumption at the bottom of the distribution.

4. **Efficiency cost:** Both schemes reduce aggregate labour supply relative to the no-tax case, due to the income effect from transfers. Targeted transfers have a somewhat larger negative effect on low-income labour supply because those households receive higher transfers.

5. **Equity–emissions frontier:** Higher tax rates reduce aggregate emissions (total x falls). The targeted scheme produces larger equity gains per unit of emissions reduction than the lump-sum scheme at the same tax rate.

---

## Limitations

- **Static model only.** No dynamic savings or capital accumulation.
- **Exogenous wages.** The wage distribution is fixed; the model does not allow carbon taxation to shift labour demand or pre-tax wages.
- **Constant gamma dispersion.** The decreasing gamma profile is a calibrated approximation. Empirical energy expenditure shares vary more continuously with income.
- **Simple two-parameter transfer rule.** Scheme B uses a linear rank-based formula. A continuous optimal transfer schedule would be more realistic but is outside the scope of this course project.
- **No energy price feedback.** tau is exogenous. In a full model, the tax would affect the producer price of x and feed back into demand.

---

## References

Goulder, L. H., Hafstead, M. A. C., and Williams, R. C. (2016). General equilibrium impacts of a federal clean energy standard. *American Economic Journal: Economic Policy*, 8(2), 186–218.

Metcalf, G. E. (2019). On the economics of a carbon tax for the United States. *Brookings Papers on Economic Activity*, Spring 2019, 405–458.

Ohlendorf, N., Jakob, M., Minx, J. C., Schroeder, C., and Steckel, J. C. (2021). Distributional impacts of carbon pricing: A meta-analysis. *Environmental and Resource Economics*, 78(1), 1–42.

Fremstad, A., and Paul, M. (2019). The impact of a carbon tax on inequality. *Ecological Economics*, 163, 88–97.

Ramsey, F. P. (1927). A contribution to the theory of taxation. *The Economic Journal*, 37(145), 47–61.

McKinney, W. (2022). *Python for Data Analysis: Data Wrangling with pandas, NumPy, and Jupyter* (3rd ed.). O'Reilly Media.

---

## Course Alignment

| Weeks  | Topic covered in this project |
|--------|-------------------------------|
| 2–3    | Fixed-point iteration in the GE solver (wage and transfer convergence) |
| 4      | Consumer optimisation via the analytical FOC for labour supply |
| 5      | Simulation of wage distributions with `np.random.lognormal` |
| 7–8    | General equilibrium framework; welfare and burden analysis by decile |
| 9      | Equity–efficiency trade-off across tau and lambda grids |

---
