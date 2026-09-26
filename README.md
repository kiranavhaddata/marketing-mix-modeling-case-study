#Marketing Mix Modeling (MMM) — Leakage-Safe OLS Case Study

Project type: End-to-end weekly Marketing Mix Modeling and budget optimization

Modeling framework: OLS MMM with leakage-safe transformations, chronological rolling cross-validation, untouched final holdout validation, contribution decomposition, ROAS analysis, response curves, and fixed-budget optimization

Notebook: MMM_Leakage_Safe_Case_Study_Finalized.ipynb

Primary planning unit: Weekly

Media families: Print, TV, Meta, Instagram, YouTube

Target variable: Sales_Volume_kgs

Revenue field: Sales_Value_INR

Currency: INR (₹)

1. Executive Summary

This project builds a full Marketing Mix Model (MMM) for a consumer brand using weekly sales, media inputs, media spend, promotion, price, trend, and seasonality data.

The central technical objective is to prevent validation leakage. In a time-series MMM, leakage can happen even when the regression itself is trained only on historical observations: transformation parameters such as scaling references, saturation parameters, and price-centering values can accidentally be learned using future observations. This project explicitly prevents that by fitting learned quantities only on the appropriate training period and applying the frozen parameters forward in time.

The modeling workflow has four major layers:

Data preparation — ingest, clean, validate, align, and merge the weekly source datasets.

Leakage-safe MMM construction — scale media, aggregate media families, create controls, add Fourier seasonality, apply geometric adstock, apply zero-anchored logistic saturation, tune transformations with chronological rolling cross-validation, and fit an OLS model.

Business decomposition — use the locked full-history historical refit to estimate media contributions, incremental sales, incremental revenue, ROAS, and response curves.

Budget optimization — convert weekly spend into modeled response using a historical spend-to-input bridge and optimize the allocation of a fixed total budget across the five media families while accounting for adstock, saturation, and the fitted OLS response.

The final budget recommendation is explicitly model-implied. It is a planning output derived from the fitted MMM, not a guaranteed causal outcome.

2. Business Objective

The project is designed to answer the following business questions:

How do media, promotions, price, trend, and seasonality relate to weekly sales?

How much incremental sales volume is attributed to each media family under the fitted model?

What incremental revenue does the model associate with each media family?

What is the historical ROAS by media family and fiscal year?

How does modeled response change as spend increases?

Given a fixed total weekly media budget, how should spend be reallocated across media families to maximize modeled incremental media revenue?

The final optimization is deliberately framed as an allocation problem rather than a simple historical-ROAS ranking. Historical ROAS is an average efficiency statistic; a forward allocation problem must recognize diminishing returns. The optimizer therefore evaluates the modeled response curve at different spend levels and searches for the allocation that maximizes modeled incremental media revenue subject to explicit constraints.

3. Core Technical Story: Validation Leakage as a Pipeline Problem

The most important methodological principle in the notebook is:

Validation leakage is not only a regression-training problem; it can occur anywhere a learned transformation uses future observations.

The project addresses the main leakage paths as follows:

Pipeline component

Leakage control

Media-family scaling

Scaling references are learned from the first 80% outer-training period and then frozen.

Trend

Trend uses the first observed week as a deterministic origin rather than a full-sample centering statistic.

Price centering

The price reference is learned from the outer training period; inner CV uses a fold-specific training mean.

Adstock

Recursive adstock is causal: each week uses current and prior media values only.

Saturation

Saturation reference statistics are learned only from the relevant training rows.

Adstock/saturation tuning

Parameters are selected using chronological rolling validation inside the development period.

Final holdout

The final 20% of observations remains untouched during tuning and development-model fitting.

Historical refit

The all-history refit is performed only after the validation gate and is used for historical attribution/business analysis, not for claiming out-of-sample performance.

This distinction is essential: the full-history model is useful for historical contribution, ROAS, response curves, and budget planning, but it is not the basis for the notebook's genuine out-of-sample validation claim.

4. Data Inputs

The finalized notebook expects four CSV files in the same folder as the notebook, or in /mnt/data when running in the supplied environment.

Required input files

Consumer Brand_Soap_Sales.csv
Consumer Brand_Media_Input.csv
Consumer Brand_Media_Spends.csv
Consumer Brand_Soap_Promo.csv

4.1 Sales data

The sales source provides the response and revenue information used by the MMM and downstream business analysis.

Key fields used in the notebook include:

Week

Week_Start

Sales_Volume_kgs

Sales_Value_INR

AVG_Price_Per_kgs

Sales_Volume_kgs is the OLS target variable.

Sales_Value_INR is the actual sales-value field used for revenue translation.

AVG_Price_Per_kgs is retained as a price control and is mean-centered using a training-referenced value before modeling.

4.2 Media input data

The media-input source contains campaign/activity variables. The notebook maps these campaign variables into five media families.

4.3 Media spend data

The media-spend source is preserved separately from the response/input dataset so that spend is available for ROAS and budget optimization.

Spend is aggregated into the same five media families used by the MMM.

4.4 Promotion data

Promotion and trade-promotion variables are carried into the modeling base and later treated as controls alongside price, trend, holiday variables, and seasonality.

5. Media-Family Structure

The notebook uses the following campaign-to-family mapping.

Print

Print_Gentle_Start

Print_Moms_Trust

TV

TV_Classic_Care_15s

TV_Apollo_Gentle_Bath_20s

TV_Motherbrand_Safest_Touch_15s

TV_Soap_Core_Rejuvenate_15s

Meta

Meta_First_Touch_Reach

Meta_Gentle_Bath_Video

Meta_Moms_Trust_Retargeting

Instagram

Instagram_Baby_Bath_Reels

Instagram_Mom_Creator_Stories

Instagram_Gentle_Skin_Carousel

YouTube

YouTube_Safest_Touch_15s

YouTube_Gentle_Bath_20s

YouTube_Mom_Stories_30s

YouTube_Soap_Core_Bumper_6s

For each week, the campaigns inside a family are summed into a family-level total before the main MMM transformations.

For example:

Meta_t =
    Meta_First_Touch_Reach_t
  + Meta_Gentle_Bath_Video_t
  + Meta_Moms_Trust_Retargeting_t

The same family structure is then reused for spend aggregation, contribution, ROAS, response curves, and budget optimization so that the business outputs stay aligned with the modeling variables.

6. Data Preparation and Quality Controls

The notebook starts by standardizing the input data before modeling.

Numeric and date handling

Week is converted to numeric.

Week_Start is converted to a date using dayfirst=True.

Other numeric columns are converted using numeric coercion after removing common formatting artifacts such as commas and placeholder values.

Column names are stripped and standardized.

Weekly integrity checks

The notebook verifies:

required time fields exist in every dataset;

weeks are sorted chronologically;

duplicate week values are not present;

a week maps to a consistent Week_Start value;

week coverage is compared across sales, media input, media spend, and promotion datasets;

merged master data does not contain unexpected missing values;

media spend aligns with the MMM weeks used downstream.

The master MMM dataset combines sales, media inputs, and promotion data, while media spend is retained separately for ROAS analysis.

7. Media Scaling

Media campaigns can exist on very different numerical scales. To make the family-level modeling variables numerically comparable, the notebook performs leakage-safe scaling.

The key rule is that the scaling references are learned from the first 80% of the chronological observations only.

The notebook also retains full-period maxima for descriptive inspection, but those full-period statistics are not used to learn the modeling transformation.

This prevents future holdout observations from changing the scale used by the model.

8. Modeling Base

The modeling base contains:

weekly sales volume;

training-referenced price;

five family-level media inputs;

available promotion and trade-promotion variables;

holiday variables when present;

deterministic time trend;

Fourier seasonality terms.

Trend

The time trend is based on the first observed week:

Trend = Week - first observed Week

This avoids using a full-sample mean or other future-dependent centering quantity.

Fiscal-year convention

The notebook labels calendar years as:

2023 -> FY23
2024 -> FY24
2025 -> FY25

The original Week_Start date is preserved so the fiscal-year convention can be changed later if the business uses a non-calendar fiscal year.

9. Seasonality: Fourier Terms

Weekly sales can contain recurring annual patterns that are unrelated to media activity. The notebook uses Fourier terms as a compact seasonal representation.

The principal annual seasonality representation uses a sine/cosine pair, with the model using the available Fourier_Sin_1 and Fourier_Cos_1 variables when those columns are present.

Fourier terms are deterministic functions of time. Their inclusion should still be judged through training-period validation rather than by optimizing against the final holdout.

10. Media Carryover: Geometric Adstock

MMM media effects often persist beyond the week in which activity occurs. The notebook models this carryover using geometric cumulative adstock:

[
Adstock_t = x_t + \alpha \cdot Adstock_{t-1}
]

where:

x_t = current week's media input;

alpha = retention/decay parameter;

Adstock_t = current week's carryover-adjusted media level.

Baseline adstock parameters

The notebook defines initial family-level values as:

Media family

Alpha

Print

0.40

TV

0.65

Meta

0.30

Instagram

0.25

YouTube

0.40

These values are used as initial/modeling parameters in the pipeline, with transformation tuning subsequently performed using chronological validation.

The corresponding half-life is calculated as:

[
Half\ Life = \frac{\log(0.5)}{\log(\alpha)}
]

Adstock is causal: the calculation for week t never uses future media activity.

11. Nonlinear Response: Zero-Anchored Logistic Saturation

To represent diminishing returns, the notebook applies logistic saturation after adstock.

A standard logistic curve has a positive value at zero input when its midpoint is positive. That can create an undesirable positive media contribution during weeks with no effective media. The notebook therefore uses a normalized, zero-anchored logistic transformation.

The function is normalized so:

Saturation(0) = 0

and the transformed response approaches 1 at sufficiently high input.

Conceptually:

[
S(x) = \frac{L(x)-L(0)}{1-L(0)}
]

where L(x) is the ordinary logistic function.

The transformed value is clipped to the [0, 1] interval.

Saturation parameterization

During tuning, the notebook derives a baseline midpoint from the median positive adstock and a baseline steepness from the interquartile range:

[
k_{base} = \frac{2\ln(3)}{IQR}
]

The candidate grid varies:

alpha: 0.00, 0.20, 0.40, 0.60, 0.80

saturation midpoint multiplier: 0.75, 1.00, 1.25

saturation slope multiplier: 0.50, 1.00, 2.00, 4.00

Crucially, saturation reference statistics are recalculated inside each training fold during rolling cross-validation, so validation rows do not affect the transformation parameters.

12. Price Treatment

AVG_Price_Per_kgs is retained as a demand control.

The modeling version is centered using a training-referenced mean:

[
Price_Centered = Price - Mean(Price_{training})
]

For inner rolling validation, the mean is recalculated using only the rows belonging to that fold's training portion.

This prevents future price observations from influencing the transformation.

13. Leakage-Safe Model Tuning

The notebook separates model tuning from final holdout evaluation.

Outer split

The data is split chronologically:

first 80% = development/training period;

final 20% = untouched holdout period.

No holdout observation is used to tune the transformation parameters or fit the development model.

Inner rolling validation

Within the development period, the notebook creates chronological validation folds. The configured fold endpoints are based on:

55% of development period
70% of development period
85% of development period

with the validation length set to approximately 10% of the development period and never allowed to cross into the final holdout.

Candidate grid

Adstock and saturation are evaluated through a grid of candidate parameters. For each candidate configuration and fold:

transform the media using the candidate adstock;

learn saturation reference statistics from the fold's training rows only;

apply the frozen saturation parameters to the validation rows;

center price using the fold's training mean only;

fit OLS on the fold's training rows;

calculate validation RMSE.

The selected parameters are those associated with the strongest chronological validation performance according to the notebook's tuning procedure.

14. OLS MMM Specification

The final OLS predictor set consists of the five saturated media-family variables plus available controls.

The media predictors are:

Print_Saturated
TV_Saturated
Meta_Saturated
Instagram_Saturated
YouTube_Saturated

The control block can include:

Price_Mean_Centered

holiday variables beginning with Holiday_

promotion variables beginning with Promotion_

trade-promotion variables beginning with TradePromo_

Trend_Centered or Trend

Fourier_Sin_1 and Fourier_Cos_1 (or the legacy Sin_52 / Cos_52 naming when available)

The exact final set is generated programmatically from columns that exist in the modeling data.

The regression is estimated using statsmodels OLS with an intercept.

15. Genuine Holdout Validation

The development OLS is fitted only on the first 80% of observations.

It is then used to predict the final 20% without any refitting on those rows.

The notebook reports:

RMSE;

MAE;

MAPE;

WAPE.

It also produces actual-vs-predicted and residual diagnostics.

The holdout is the notebook's genuine evidence of out-of-sample predictive performance.

Important interpretation rule

The full-history model created later in the notebook must not be described as an out-of-sample model. It is a historical refit performed after the validation gate.

16. Coefficient-Direction Audit

The notebook performs a diagnostic review of model coefficient directions.

This audit is intended to identify:

negative media coefficients;

unexpected price direction;

other signs that warrant business or modeling investigation.

The notebook does not force coefficients to expected signs merely because a sign is inconvenient. A coefficient sign is treated as a diagnostic result that should be investigated in light of data coverage, collinearity, specification, and business context.

17. Historical Full-Data Refit

After transformation/model specification selection and holdout evaluation, the notebook refits the locked specification on all available history.

The object is:

full_data_ols_model

This historical refit is then used for:

weekly contribution decomposition;

fiscal-year contribution summaries;

incremental sales;

incremental revenue;

ROAS;

response curves;

budget optimization.

This model is intentionally separated from the holdout-performance claim.

18. Contribution Decomposition

The notebook calculates media contribution using the fitted full-history OLS coefficients and the corresponding saturated media variables.

For a given media family:

SaturatedMedia_t \times OLS\ Beta
]

The media contribution outputs are combined with controls and baseline-related components so that the decomposition can be reconciled to the model's predicted sales.

Outputs are produced at weekly and fiscal-year levels.

19. Incremental Revenue and ROAS

Incremental sales

The MMM produces media-attributed incremental sales in kilograms.

Revenue translation

The notebook uses actual sales-value data from the sales source to derive revenue translations.

For fiscal-year reporting, average realized price is calculated as:

\frac{FY\ Sales\ Value}{FY\ Sales\ Volume}
]

Incremental media revenue is then:

Incremental\ Media\ Sales
\times
FY\ Average\ Price
]

ROAS

For a media family:

\frac{Incremental\ Media\ Revenue}{Media\ Spend}
]

If spend is zero, ROAS is undefined and is represented as NaN rather than forcing an artificial value.

ROAS is available at:

weekly level;

fiscal-year level;

media-family level;

total-media level;

overall FY23–FY25 level.

20. Response Curves

The response-curve layer translates the fitted media transformation into modeled incremental response as media activity increases.

The response follows the same conceptual chain used in the MMM:

Media input
    ↓
Geometric adstock
    ↓
Zero-anchored logistic saturation
    ↓
OLS media coefficient
    ↓
Modeled incremental sales
    ↓
Revenue translation

The curves are intended to show the shape of modeled response and the presence of diminishing returns rather than to imply that every point on the curve is equally supported by observed data.

21. Budget Optimization — Methodology

Budget optimization is implemented as the final stage of the notebook.

21.1 Objective

The optimizer reallocates the current historical average weekly media budget across the five media families:

Print

TV

Meta

Instagram

YouTube

The objective is:

[
\max \sum_j Incremental\ Revenue_j
]

subject to:

[
\sum_j Spend_j = Current\ Total\ Weekly\ Budget
]

and:

[
0 \le Spend_j \le 1.25 \times Historical\ Max\ Weekly\ Spend_j
]

The 1.25× cap is an explicit extrapolation guard. It limits the optimizer from allocating materially more spend to a family than the observed historical range supports.

21.2 Why the optimizer does not simply rank ROAS

Historical ROAS is an average efficiency measure over observed spend.

A budget optimizer must ask a different question:

What is the modeled incremental revenue generated by the next unit of spend at the proposed spending level?

Because the model contains saturation, the incremental return changes as spend changes. A family with a strong historical ROAS may not remain equally efficient as additional spend is added.

Therefore, the optimizer uses the nonlinear response implied by the fitted adstock, saturation, and OLS coefficient.

22. Spend-to-Input Bridge

The MMM is fit to transformed media inputs, while optimization decisions are expressed in INR spend. The notebook therefore creates a historical family-level bridge between the two.

For each family it estimates a through-origin relationship:

[
Model\ Input \approx Spend_{INR} \times InputPerINR
]

The slope is learned from all overlapping historical weeks with positive spend.

The notebook records:

historical weeks with spend;

total historical spend;

total historical model input;

Input_Per_INR;

through-origin bridge R².

A non-positive bridge slope causes the optimizer to stop rather than produce a meaningless allocation.

This bridge is a planning approximation: it assumes the historical relationship between spend and media input remains useful for planning the optimized allocation.

23. Optimization Response Function

For each media family, the optimizer performs the following calculation:

Step 1 — Spend to weekly media input

Spend \times InputPerINR
]

Step 2 — Convert to steady-state adstock

The planning response uses a constant-spend steady-state representation:

\frac{Weekly\ Input}{1-\alpha}
]

This is the infinite-horizon solution of the geometric adstock recursion under constant weekly input.

Step 3 — Apply the same logistic saturation

The steady-state adstock is passed through the notebook's locked zero-anchored logistic saturation function using the fitted family-specific k and x0 parameters.

Step 4 — Apply the fitted OLS media coefficient

OLS\ Beta \times Saturation
]

Step 5 — Translate incremental sales to revenue

The optimizer uses one common historical weighted average planning price:

\frac{Total\ Sales\ Value}{Total\ Sales\ Volume}
]

Then:

Incremental\ Sales \times Planning\ Price
]

The common price is a translation scalar; therefore, within the fixed-budget allocation problem, it does not change the relative allocation that maximizes modeled incremental sales/revenue.

24. Optimization Algorithm

The notebook uses SciPy's constrained numerical optimizer with the SLSQP method.

Initialization

The starting allocation is the historical average weekly spend by family.

Constraints

Non-negative spend per family.

Upper bound = 1.25× historical maximum weekly spend for that family.

Sum of optimized weekly spend must equal the historical average total weekly budget.

Feasibility check

Before optimization, the notebook verifies that the sum of the allowed family upper bounds is large enough to accommodate the total planning budget.

Convergence check

The optimizer must return a successful status. If it fails to converge, the notebook raises an error rather than silently accepting an unreliable allocation.

25. Finalized Budget Output

The final business-facing section is explicitly titled:

FINALIZED MMM BUDGET OPTIMIZATION RESULT

It reports:

Top-level totals

current average weekly media budget;

optimized weekly media budget;

annual planning budget using 52 weeks;

current-budget modeled incremental sales;

optimized modeled incremental sales;

modeled sales uplift per week and year;

current-budget modeled incremental revenue;

optimized modeled incremental revenue;

modeled revenue uplift per week and year;

current-budget scenario ROAS;

optimized scenario ROAS.

Family-level allocation table

The final allocation table contains:

Media_Family

Current_Avg_Weekly_Spend_INR

Current_Budget_Share_%

Optimized_Weekly_Spend_INR

Optimized_Budget_Share_%

Spend_Change_INR_per_Week

Spend_Change_%

Optimized_Annual_Spend_INR

Marginal_Revenue_per_Additional_INR

OLS_Beta

Model_Response_Flag

The Marginal_Revenue_per_Additional_INR field is intended to show the local modeled incremental revenue associated with a further unit of spend at the optimized point.

26. Interpreting the Final Recommendation

The final allocation should be read as:

The media-family spend mix that maximizes modeled incremental media revenue under the notebook's fixed-budget, response-function, and extrapolation constraints.

It should not be interpreted as:

a guarantee of future sales;

a causal estimate immune to omitted-variable bias;

a replacement for media buying judgment;

proof that one channel will always outperform another;

a recommendation to exceed the historical domain materially.

The result is conditional on the fitted MMM, the spend-to-input bridge, the steady-state adstock assumption, the saturation function, and the planning-price translation.

Particular caution is warranted for any family with:

a weak spend-to-input bridge;

a non-positive OLS media coefficient;

limited active weeks;

spend levels near or above the historical maximum;

high uncertainty or unstable model diagnostics.

27. Generated Output Files

The notebook writes business-ready and audit-friendly CSV outputs under the mmm_outputs folder.

Core modeling outputs

Consumer Brand_MMM_Master_Numeric.csv
Consumer Brand_MMM_Modeling_Base.csv
Consumer Brand_MMM_Model_Base_Adstocked.csv
Consumer Brand_MMM_Model_Base_Adstock_Saturated.csv
Consumer Brand_MMM_Final_Adstock_Saturated_Tuned.csv

Contribution and incremental-sales outputs

Consumer Brand_MMM_Weekly_Contribution.csv
Consumer Brand_MMM_FY23_FY24_FY25_Contribution.csv
Consumer Brand_MMM_Total_Contribution_FY23_FY25.csv
Consumer Brand_MMM_Weekly_Incremental_Sales_Revenue.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_Incremental_Sales.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_Incremental_Revenue.csv

Sales and ROAS outputs

Consumer Brand_MMM_FY23_FY24_FY25_Sales_Revenue_Summary.csv
Consumer Brand_MMM_Weekly_Media_ROAS.csv
Consumer Brand_MMM_Weekly_ROAS_Summary.csv
Consumer Brand_MMM_FY_Media_ROAS.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_ROAS.csv
Consumer Brand_MMM_Overall_FY23_FY25_Media_ROAS.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_Spend.csv

Response-curve outputs

Consumer Brand_MMM_Response_Curves.csv
Consumer Brand_MMM_Response_Curve_Summary.csv

Model diagnostics

Consumer Brand_MMM_Actual_Predicted_Residuals.csv

Budget optimization outputs

Consumer Brand_MMM_Budget_Optimization_Parameters.csv
Consumer Brand_MMM_Optimized_Budget_Allocation.csv
Consumer Brand_MMM_Spend_to_Input_Bridge.csv
Consumer Brand_MMM_FINAL_Budget_Optimization_Result.csv
Consumer Brand_MMM_FINAL_Budget_Optimization_Summary.csv

The two files beginning with FINAL_ are intended to be the easiest outputs for a business stakeholder to review.

28. Recommended Project Structure

A clean project directory can be organized as:

MMM_Project/
│
├── MMM_Leakage_Safe_Case_Study_Finalized.ipynb
├── README_MMM_Leakage_Safe_Case_Study.md
│
├── Consumer Brand_Soap_Sales.csv
├── Consumer Brand_Media_Input.csv
├── Consumer Brand_Media_Spends.csv
├── Consumer Brand_Soap_Promo.csv
│
└── mmm_outputs/
    ├── Consumer Brand_MMM_Master_Numeric.csv
    ├── Consumer Brand_MMM_Modeling_Base.csv
    ├── Consumer Brand_MMM_Final_Adstock_Saturated_Tuned.csv
    ├── Consumer Brand_MMM_Weekly_Contribution.csv
    ├── Consumer Brand_MMM_Weekly_Media_ROAS.csv
    ├── Consumer Brand_MMM_Response_Curves.csv
    ├── Consumer Brand_MMM_Budget_Optimization_Parameters.csv
    ├── Consumer Brand_MMM_Spend_to_Input_Bridge.csv
    ├── Consumer Brand_MMM_FINAL_Budget_Optimization_Result.csv
    └── Consumer Brand_MMM_FINAL_Budget_Optimization_Summary.csv

29. How to Run the Notebook

Step 1 — Place the files together

Keep the finalized notebook and all four required CSVs in the same directory.

The notebook first checks the current working directory and then /mnt/data if the files are not found in the current directory.

Step 2 — Open the notebook

Run the notebook in Jupyter Notebook, JupyterLab, VS Code, or another compatible environment with the required Python libraries installed.

Step 3 — Run cells sequentially

The notebook is intentionally structured as a sequential pipeline. Run from the beginning through the validation and business-output sections.

The budget optimization section depends on objects created earlier in the notebook and therefore should not be run in isolation.

Step 4 — Review the validation gate

Before interpreting contributions or optimization results, review:

development-vs-holdout metrics;

coefficient-direction diagnostics;

model diagnostics;

the distinction between the development model and full-history historical refit.

Step 5 — Review the final allocation

The final business-facing output is generated in the final cell and saved to:

mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Result.csv
mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Summary.csv

30. Python Dependencies

The notebook imports the following primary packages/modules:

itertools
pathlib
numpy
pandas
matplotlib
seaborn
scipy
scikit-learn
statsmodels

The optimization layer additionally requires:

from scipy.optimize import minimize

The notebook does not reinstall packages; it assumes the active Python environment already contains the required dependencies.

31. Key Notebook Objects

Several objects are intentionally named so that the notebook's layers can be audited or extended.

Data objects

sales_df_clean
media_input_df_clean
media_spend_df_clean
promo_df_clean
master_mmm_df
media_spend_for_roas_df

Modeling objects

model_base_df
model_adstock_df
model_saturation_df
final_transformed_df
final_ols_df

Validation/model objects

validation_train_df
validation_holdout_df
final_ols_model
full_data_ols_model
holdout_metrics
validation_summary_df

Transformation parameters

alpha_values
saturation_parameters

Business outputs

weekly_contribution_df
weekly_revenue_df
weekly_roas_df
response_curve_df

Optimization objects

input_per_inr
spend_input_bridge_df
optimization_parameters_df
current_budget_by_family
current_total_weekly_budget
optimized_allocation
budget_allocation_df
final_budget_result_df
final_summary_df

These names make it easier to inspect intermediate values without reverse-engineering the notebook.

32. Model Diagnostics and Statistical Review

The notebook includes a broad diagnostic layer after model fitting. The imported statistical utilities include tests and diagnostics for:

heteroskedasticity;

autocorrelation;

Ljung–Box residual structure;

Breusch–Pagan testing;

Breusch–Godfrey testing;

Durbin–Watson;

Jarque–Bera normality diagnostics;

variance inflation factors (VIF);

influence and outlier diagnostics;

linear specification checks such as RESET.

These diagnostics are intended to help determine whether the fitted OLS model is structurally adequate and whether individual coefficients should be interpreted cautiously.

Statistical diagnostics should be considered alongside business knowledge and data limitations rather than treated as automatic proof of causal validity.

33. Important Modeling Assumptions

The MMM and optimization depend on several assumptions.

33.1 Historical relationships are informative for planning

The model assumes that the relationship learned from the historical data provides useful information for the planning scenario.

33.2 Media input is a valid exposure/activity measure

The model uses media inputs as the primary media-side explanatory variables. Spend is connected to these variables only through the historical spend-to-input bridge used for optimization.

33.3 Geometric adstock is an adequate carryover shape

Media persistence is represented by geometric decay. Other carryover shapes could be implemented in a future model version.

33.4 Logistic saturation captures diminishing returns sufficiently well

The response is constrained to a zero-anchored logistic shape. More flexible saturation functions could be explored in a later model iteration.

33.5 Constant planning price is sufficient for budget translation

Optimization uses a common historical weighted average price solely to convert incremental kilograms to incremental revenue. This does not model future price changes.

33.6 Steady-state adstock is suitable for planning

The optimizer converts constant weekly spend into steady-state adstock. It therefore evaluates a long-run constant-spend scenario rather than simulating a finite campaign ramp with week-by-week media pulses.

33.7 Upper spend caps improve planning robustness

The 1.25× historical maximum weekly spend cap is a guard against excessive extrapolation. It is a pragmatic planning constraint, not a learned business rule.

34. Limitations

This project is intentionally presented as a deterministic OLS MMM and should be interpreted within those limitations.

Causality

OLS association does not automatically establish causal lift. Confounding, omitted variables, reverse causality, media targeting, and measurement error can affect coefficient estimates.

Collinearity

Media channels may move together, especially when campaigns are planned in coordinated bursts. Collinearity can make individual media coefficients unstable even when overall model fit is strong.

Limited experiment-based identification

The notebook is not a randomized experiment. Where possible, future work should incorporate experimental or quasi-experimental evidence to strengthen causal interpretation.

Aggregation bias

Media are aggregated into family-level weekly inputs. Campaign-level heterogeneity is therefore compressed.

Optimization extrapolation

The optimizer is bounded, but it still makes a planning extrapolation beyond the exact historical allocation observed in some cases.

Deterministic point estimates

The current optimization is based on point estimates. It does not yet provide a posterior distribution or probabilistic range for the recommended allocation.

Full-history refit

The historical refit is intentionally useful for attribution and optimization, but it should not be confused with the untouched holdout validation model.

35. Recommended Next Enhancements

The notebook's own roadmap identifies a Bayesian extension as the next major methodological stage.

PyMC / uncertainty layer

A Bayesian MMM could be added after the deterministic pipeline is stable so that the project can report distributions rather than only point estimates for:

media contributions;

channel ROAS;

incremental sales;

incremental revenue;

response curves;

budget allocation.

Scenario planning

A future version could support several planning budgets, for example:

- Current budget
- +10%
- +20%
- -10%
- -20%

and compare the model-implied allocations and response across scenarios.

Finite-horizon budget simulation

Rather than only using steady-state adstock, a future optimizer could simulate a full 13-, 26-, or 52-week schedule and account for carryover explicitly week by week.

Additional business constraints

The optimizer could later include:

minimum spends;

maximum spend shares;

channel commitments;

production constraints;

media inventory constraints;

campaign-specific caps;

business-defined guardrails.

36. Governance and Publication Note

The notebook is a portfolio/case-study edition and explicitly notes that source data and client-specific campaign files are not included.

Before publishing the project, confirm employer/client confidentiality requirements regarding:

real data;

derived performance metrics;

brand identifiers;

campaign names;

screenshots;

spend levels;

contribution values;

ROAS values;

model coefficients.

Do not publish confidential business information merely because it appears in a generated output file.

37. Reproducibility Checklist

Before considering a run complete, verify the following:

All four required CSV files are present.

Dates and week identifiers parse correctly.

No duplicate weekly keys exist.

Week coverage is aligned across source datasets.

The development/holdout split remains chronological.

The final 20% holdout is not used during tuning.

Media scaling is learned from the development period only.

Price centering is training-referenced.

Saturation reference statistics are fold-specific during rolling CV.

The development model is evaluated on the untouched holdout without refitting.

The historical full-data refit is performed only after validation.

Media contributions reconcile with the model's predicted sales decomposition.

Spend successfully matches all MMM weeks for ROAS.

The spend-to-input bridge is positive for every optimized family.

The optimization problem is feasible under the 1.25× family caps.

The SLSQP optimizer converges successfully.

The optimized total spend equals the current total weekly planning budget.

The final optimization CSVs are generated successfully.

38. Final Business Interpretation Framework

When presenting the results to a stakeholder, the recommended narrative is:

Model credibility — start with the chronological validation design and untouched holdout rather than leading with attribution numbers.

Response understanding — explain media contribution, carryover, and diminishing returns.

Efficiency — review historical incremental revenue and ROAS by media family and fiscal year.

Planning — show the optimized allocation under the fixed-budget constraint.

Sensitivity and caveats — explain that the allocation is conditional on the fitted MMM and should be monitored against future actual performance.

A concise business statement is:

The finalized MMM converts historical weekly media, promotion, price, trend, and seasonality data into a leakage-safe OLS response model. After genuine out-of-sample validation, the locked specification is refit on all history for attribution and planning. The budget optimizer then reallocates the historical average weekly media budget across Print, TV, Meta, Instagram, and YouTube using the fitted adstock and saturation response, while limiting extrapolation to 1.25× each family's historical maximum weekly spend. The resulting allocation is a model-implied planning scenario, not a guaranteed causal outcome.

39. Final Deliverable

The primary technical deliverable is:

MMM_Leakage_Safe_Case_Study_Finalized.ipynb

The primary documentation deliverable is:

README_MMM_Leakage_Safe_Case_Study.md

The primary business-facing optimization outputs are:

mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Result.csv
mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Summary.csv

Together, these provide the reproducible modeling workflow, validation framework, business attribution layer, and final budget-planning output.

40. Version Note

This README documents the finalized notebook structure and optimization methodology as currently implemented in the supplied case-study notebook.

The notebook is designed so that the actual numerical budget allocation, modeled revenue uplift, ROAS, and channel-level recommendations are calculated from the supplied source data when the notebook is executed end to end.

