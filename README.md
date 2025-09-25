
# Bayesian Statistics Project — ScotsSec (mlmRev)

> Multilevel Bayesian logistic regression of exam attainment in Scottish schools.

## Overview
This repository contains the code and report for my Bayesian Statistics class project. I model a binary outcome indicating whether a student’s standardized exam score at age 16 (`attain01`) crosses a performance threshold, using student-level predictors and school-level grouping to capture hierarchical structure. The analysis is implemented in R with **rstanarm** and follows a compare–select–diagnose workflow.  

## Dataset
- **Source:** `ScotsSec` dataset from the **mlmRev** R package (students in Scotland).  
- **Variables (selected):** `attain` (original score), **`attain01`** (binarized outcome), `verbal` (reasoning score), `sex` (F/M), `social` (numeric social class), `primary`, `second` (school IDs).  

## Objectives
1. Exploratory data analysis (distributions, missingness, outliers).  
2. Build a sequence of Bayesian models that progressively incorporate group structure.  
3. Compare candidate models with **LOOIC** and select the best.  
4. Diagnose fit and draw inference on **school-level random effects**, asking: *Does the primary school matter?*

## Modeling Approach
Let \(y_i=\texttt{attain01}\sim \text{Bernoulli}(p_i)\) with \(\text{logit}(p_i)=\eta_i\). Individual covariates: **verbal**, **sex** (baseline M), **social**. Grouping: **primary** school \(k(i)\) and **secondary** school \(j(i)\). I fit the following candidates (M0–M7):  

- **M0:** Intercept-only.  
- **M1:** Fixed effects: `verbal + sex + social`.  
- **M2:** *[alias in code]* same fixed-effects baseline (for convenience/consistency).  
- **M3:** M1 + random intercepts by **primary**: `(1 | primary)`.  
- **M4:** M1 + random intercepts by **second**: `(1 | second)`.  
- **M5:** M1 + random intercepts **and slopes** for `verbal` and `social` by **primary**: `(1 + verbal + social | primary)`.  
- **M6:** M1 + random intercepts **and slopes** for `verbal` and `social` by **second**: `(1 + verbal + social | second)`.  
- **M7:** M1 + random intercepts **and slopes** for both **primary** and **second**.  

All models are fit with **binomial (logit)** link via `stan_glm` / `stan_glmer` (rstanarm).  

## Model Selection
Models are compared using **LOOIC** (lower is better), computed via the `loo` package. I tabulate `elpd_loo`, its SE, `p_loo`, `looic`, and the count of high Pareto-\(k\) points (\(k>0.7\)) to flag reliability issues; I also include a dot-and-interval plot of LOOIC with ±2·SE bars. The **best model** is chosen as the one with the smallest LOOIC.  

> In the final inference section, I focus on the **primary-school** random effects from the selected model and visualize fixed effects alongside school intercepts with **50%–95% credible intervals**. The primary-school random-intercept SD is reported around **0.49 (log-odds)**, indicating meaningful between-school heterogeneity.  

## Key Findings (brief)
- **Individual effects:** higher `verbal` and higher `social` class are associated with increased odds of attaining the threshold; the `sex` coefficient (F vs M) is estimated jointly with these covariates.  
- **School effects:** there is clear **between-primary-school** variability in attainment odds; several primary schools exhibit notably positive or negative shifts relative to the grand mean.  
- **Conclusion:** *Yes, the primary school matters.* Accounting for school-level grouping is essential for fit and interpretation. 

## EDA Notes
- Distribution plots for outcome and covariates, missingness counts, and boxplots by school are included.  
- A small number of extreme outliers in `verbal` (≈ 11 of ~3000) are identified and removed using the 1.5·IQR rule.

## Repository Structure
```
.
├── project_bs.Rmd        # RMarkdown source (analysis + report)
├── project_bs.pdf        # Rendered report
└── README.md             # This file
```

## Reproducibility
- **R version / packages:** listed at the top of the Rmd and loaded programmatically (install-if-missing helper included).  
- **Fitting:** `rstanarm::stan_glm/stan_glmer` with cached chunks enabled in knitr to avoid refitting when not needed.  
- **Model comparison & plots:** `loo`, `bayesplot`, `ggplot2`, plus small utilities for clean variable naming and interval plots.  

## How to Run
1. Open `project_bs.Rmd` in RStudio (or run with `rmarkdown::render("project_bs.Rmd")`).  
2. Ensure **rstanarm** and other packages are installed; the Rmd will try to install any missing packages.  
3. Knit to **HTML** or **PDF** to reproduce all figures, tables, and model fits.  
