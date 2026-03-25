# Causal Inference: Coupon A/B Test Analysis

This repository implements a comprehensive causal inference analysis comparing Restricted (R) vs Universal (U) birthday coupons. The analysis evaluates treatment effects on redemption rates and post-period outcomes using linear regression with robust standard errors and causal forests.

## Project Overview

The study processes member demographics (`members-group.csv`) and sales data (`sales-2024.csv`, `sales-2025.csv`) from a beverage chain. Key analyses include balance checks, Average Treatment Effects (ATE) on redemption, and Heterogeneous Treatment Effects (HTE) across demographics, product preferences, and purchase frequency. Post-period outcomes cover spending, visits, and cups purchased.

## Key Components

- **Data Processing**: Cleans sales data, computes baseline purchase frequency (2024 cups), tea/coffee preferences, age from birthdays.
- **Balance Checks**: Numeric (age, frequency) and categorical (gender, card level) balance tables with t-tests and proportion tests.
- **ATE Estimation**: Linear regression (robust SE) and GRF causal forest for redemption probability.
- **HTE Analysis**: Subgroup effects by age (<45), product preference (tea/coffee), card level (Diamond/Gold/Silver), frequency (above/below median).
- **Post-Period Outcomes**: ATE/HTE on spend, visits, cups within 1-month post-coupon window.

## Requirements

- R 4.0+
- Core: `tidyverse`, `knitr`, `kableExtra`, `lubridate`, `janitor`
- Econometrics: `sandwich`, `lmtest`, `broom`
- Causal ML: `grf` (Generalized Random Forest)
- Visualization: `showtext`, `ggplot2`, `scales`

Install via:
```r
install.packages(c("tidyverse", "knitr", "kableExtra", "lubridate", "sandwich", "lmtest", "grf", "showtext"))
```

## Usage

1. Place `members-group.csv`, `sales-2024.csv`, `sales-2025.csv` in repository root.
2. Run: `rmarkdown::render("inferenceKao-Bei-1.Rmd")`.
3. Outputs include balance tables, ATE tables with 95% CIs, feature importance plots, subgroup HTE tables. 

## Results Structure

| Analysis | Methods | Key Outputs |
|----------|---------|-------------|
| Balance Check | t-tests, prop tests | Summary statistics table (means, SEs, p-values) |
| Redemption ATE | LM (robust), Causal Forest | ATE comparison, feature importance bar chart |
| HTE Subgroups | Interaction terms, Causal Forest | 8 subgroups × 3 outcomes (tables) |
| Post-Period | Spend, Visits, Cups | ATE tables + HTE subgroup tables per outcome |

Seed: 5166 for reproducibility; K=5 silhouette optimal clustering reference in related work. 

## Code Highlights

**Core ATE estimation**:
```r
# Linear regression with robust SE
m <- lm(redeemed ~ treat + Age + BaselineFreq + gender + card_level, data = df_q2)
r <- coeftest(m, vcov = vcovHC(m, type = "HC1"))

# Causal Forest (GRF)
cf <- causal_forest(Xc, Yc, Wc, num.trees = 2000)
ate <- average_treatment_effect(cf)
```

**Balance table construction** (continuous + categorical):
```r
# Numeric: means/SDs + t-tests
# Categorical: proportions + prop.tests
summary_table_display <- summary_table %>% 
  mutate(P_value = format.pval(P_value, digits = 3, eps = 0.001))
```

**Subgroup HTE** (interaction terms + causal forest per subgroup). 

## Key Findings

- Comprehensive balance checks confirm RCT validity across covariates.
- Dual estimation (LM + CF) provides robust ATE benchmarks.
- Feature importance reveals key heterogeneity drivers (age, frequency, preferences).
- Post-period analysis extends beyond redemption to business outcomes.

## Limitations

- Requires proprietary CSV data (not included).
- Assumes coupon randomization; no pre-trends validation.
- Fixed subgroups; no continuous HTE surfaces.
- Causal forest convergence depends on sample variation.

## Author

Ting-Ying Lai  
National Taiwan University  
Data Science & Econometrics Portfolio
