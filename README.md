📊 Marketing Mix Modeling (MMM) — Leakage-Safe OLS Case Study

<p align="center">
  <strong>End-to-End Weekly MMM • Leakage-Safe Validation • ROAS • Response Curves • Budget Optimization</strong>
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white">
  <img alt="Model" src="https://img.shields.io/badge/Model-OLS%20MMM-1f6feb">
  <img alt="Validation" src="https://img.shields.io/badge/Validation-Time%20Series-6f42c1">
  <img alt="Optimization" src="https://img.shields.io/badge/Optimization-SLSQP-2ea44f">
  <img alt="Currency" src="https://img.shields.io/badge/Currency-INR%20₹-ff9800">
  <img alt="Status" src="https://img.shields.io/badge/Status-Finalized-success">
</p>

Executive takeaway: This project converts weekly media, sales, promotion, price, trend, and seasonality data into a leakage-safe OLS Marketing Mix Model. After chronological tuning and untouched holdout validation, the locked specification is refit on all history for business attribution and planning. The final budget optimizer reallocates the historical average weekly media budget across Print, TV, Meta, Instagram, and YouTube, while accounting for media carryover, diminishing returns, and explicit extrapolation limits.

🧭 Table of Contents

1. Project at a Glance

2. Business Questions

3. Why This MMM Is Leakage-Safe

4. End-to-End Architecture

5. Data Inputs

6. Media-Family Structure

7. Data Preparation & Quality Controls

8. Media Transformation Pipeline

9. OLS MMM Specification

10. Time-Series Validation

11. Historical Attribution & Contribution

12. Incremental Sales, Revenue & ROAS

13. Response Curves

14. Budget Optimization

15. Spend-to-Input Bridge

16. Optimization Objective & Constraints

17. Finalized Budget Result

18. Diagnostics & Model Governance

19. Generated Outputs

20. Project Structure

21. How to Run

22. Dependencies

23. Key Notebook Objects

24. Assumptions

25. Limitations

26. Recommended Enhancements

27. Reproducibility Checklist

28. Business Presentation Framework

29. Final Interpretation

30. Deliverables

1. 📌 Project at a Glance

Item

Description

Project type

End-to-end weekly Marketing Mix Modeling + budget optimization

Model family

Leakage-safe OLS MMM

Time granularity

Weekly

Primary target

Sales_Volume_kgs

Revenue field

Sales_Value_INR

Price control

AVG_Price_Per_kgs

Media families

Print, TV, Meta, Instagram, YouTube

Carryover

Geometric adstock

Nonlinearity

Zero-anchored logistic saturation

Validation

Chronological rolling CV + untouched final holdout

Historical business model

Full-history refit after validation gate

Optimization

Fixed-budget constrained allocation using SLSQP

Planning currency

INR (₹)

Primary notebook

MMM_Leakage_Safe_Case_Study_Finalized.ipynb

🎯 Project outcome

The notebook is designed to move from raw weekly business data to four practical outputs:

Model credibility — leakage-safe transformations and genuine out-of-sample validation.

Business attribution — weekly/fiscal-year media contribution, incremental sales, revenue, and ROAS.

Response understanding — adstock, saturation, and modeled response curves.

Budget planning — a model-implied allocation of a fixed weekly media budget across five media families.

2. 💼 Business Questions

The project is structured to answer:

How do media, promotions, price, trend, and seasonality relate to weekly sales?

How much incremental sales volume is associated with each media family under the fitted model?

How much incremental revenue is associated with each media family?

What is historical ROAS by media family and fiscal year?

Where do modeled response curves indicate diminishing returns?

Given a fixed total weekly media budget, what allocation maximizes modeled incremental media revenue under the notebook's constraints?

Important: Budget optimization is an allocation exercise based on the fitted model. It is a planning scenario, not a guarantee of future causal lift.

3. 🔐 Why This MMM Is Leakage-Safe

The core principle

Validation leakage is a pipeline problem, not only a regression-training problem.

In a time-series MMM, leakage can occur when future observations influence learned preprocessing parameters such as scaling references, price-centering values, or saturation statistics. The notebook explicitly prevents this.

Pipeline component

Leakage-safe treatment

Media scaling

Learned from the first 80% outer-training period and frozen forward.

Trend

Deterministic origin from the first observed week.

Price centering

Training-referenced; inner CV uses fold-specific training means.

Adstock

Recursive and causal; uses current and prior media only.

Saturation

Reference statistics learned only from relevant training rows.

Transformation tuning

Performed with chronological rolling validation inside development data.

Final holdout

Final 20% remains untouched during tuning and development-model fitting.

Historical refit

Performed only after the validation gate for attribution/planning.

Development model vs. historical refit

The notebook intentionally separates two purposes:

Development / validation model
Used to make the genuine out-of-sample claim on the final holdout.

Full-history historical refit
Used after validation for contribution decomposition, ROAS, response curves, and budget optimization.

This distinction is essential and should be preserved in any business presentation of the results.

4. 🏗️ End-to-End Architecture

                    ┌──────────────────────────────┐
                    │       Weekly Source Data     │
                    ├──────────────────────────────┤
                    │ Sales                         │
                    │ Media Inputs                  │
                    │ Media Spend                   │
                    │ Promotion / Trade Promotion  │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ Data Cleaning & Alignment    │
                    │ • dates / weeks              │
                    │ • numeric coercion            │
                    │ • duplicate checks            │
                    │ • coverage checks             │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ Leakage-Safe Modeling Base   │
                    │ • family aggregation         │
                    │ • training-only scaling      │
                    │ • price control              │
                    │ • trend / seasonality        │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                ┌─────────────────────────────────────┐
                │ Media Transformation + Tuning       │
                │ Input → Adstock → Saturation        │
                │ Chronological rolling validation    │
                └──────────────────┬──────────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ OLS MMM + Holdout Validation │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
              Contribution       ROAS       Response Curves
                    │              │              │
                    └──────────────┼──────────────┘
                                   ▼
                    ┌──────────────────────────────┐
                    │ Budget Optimization          │
                    │ Spend → Input → Adstock      │
                    │ → Saturation → Revenue       │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ FINAL BUDGET ALLOCATION      │
                    │ Weekly + 52-week annualized │
                    └──────────────────────────────┘

5. 🗂️ Data Inputs

The finalized notebook expects four source CSVs in the same directory as the notebook, or in /mnt/data in the supplied execution environment.

Required files

Consumer Brand_Soap_Sales.csv
Consumer Brand_Media_Input.csv
Consumer Brand_Media_Spends.csv
Consumer Brand_Soap_Promo.csv

5.1 Sales

Primary fields include:

Week

Week_Start

Sales_Volume_kgs

Sales_Value_INR

AVG_Price_Per_kgs

Model target: Sales_Volume_kgs
Revenue translation: Sales_Value_INR
Price control: AVG_Price_Per_kgs

5.2 Media Input

Contains campaign/activity variables that are mapped into five media families before transformation.

5.3 Media Spend

Preserved separately for spend-based business reporting, ROAS calculation, and budget optimization.

5.4 Promotion

Promotion and trade-promotion variables are carried into the modeling base alongside other non-media controls.

6. 📡 Media-Family Structure

The notebook standardizes campaign-level variables into five business-friendly media families.

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

Family aggregation logic

Campaigns inside the same family are summed weekly before the main MMM transformations.

Example:

Meta_t =
    Meta_First_Touch_Reach_t
  + Meta_Gentle_Bath_Video_t
  + Meta_Moms_Trust_Retargeting_t

The same family definitions are reused for:

modeling;

spend aggregation;

contribution decomposition;

ROAS;

response curves;

budget optimization.

This keeps the analytical layer and the business planning layer aligned.

7. 🧹 Data Preparation & Quality Controls

The data-preparation layer standardizes the weekly sources before modeling.

Date and numeric handling

Week is converted to numeric.

Week_Start is converted to date using dayfirst=True.

Numeric fields are coerced after common formatting cleanup.

Column names are stripped/standardized.

Weekly integrity checks

The notebook checks:

required time fields exist;

chronological ordering is valid;

duplicate weeks are absent;

week-to-date mappings are consistent;

source week coverage is aligned;

master data does not contain unexpected missing values;

media spend aligns to the MMM weeks used downstream.

8. 🔄 Media Transformation Pipeline

The MMM uses the following sequence:

Raw Media Input
      │
      ▼
Leakage-Safe Scaling
      │
      ▼
Geometric Adstock
      │
      ▼
Zero-Anchored Logistic Saturation
      │
      ▼
OLS Media Coefficient
      │
      ▼
Modeled Incremental Sales
      │
      ▼
Revenue Translation

8.1 Leakage-Safe Media Scaling

Media variables may live on very different numerical scales. The notebook learns scaling references using only the first 80% chronological outer-training period, then freezes those references.

This ensures future holdout observations cannot alter the transformation used by the model.

8.2 Geometric Adstock

Media effects can persist after the original media activity occurs. The notebook applies geometric carryover:

$$
\text{Adstock}t = x_t + \alpha , \text{Adstock}{t-1}
$$

where:

$x_t$ = current-week media input;

$\alpha$ = retention/decay parameter;

$Adstock_t$ = carryover-adjusted media level.

Baseline family-level alpha values are defined as:

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

The half-life relationship is:

$$
\text{Half-Life} = \frac{\ln(0.5)}{\ln(\alpha)}
$$

Adstock is causal: week t uses current and prior media activity only.

8.3 Zero-Anchored Logistic Saturation

The model uses logistic saturation to represent diminishing returns.

A standard logistic can imply a positive response at zero input. The notebook therefore uses a normalized, zero-anchored version so that:

$$
S(0)=0
$$

Conceptually:

$$
L(x) = \frac{1}{1 + e^{-k(x-x_0)}}
$$

$$
S(x) = \frac{L(x)-L(0)}{1-L(0)}
$$

where $L(x)$ is the ordinary logistic function and $S(0)=0$.

The transformed value is clipped to [0, 1].

Tuning grid

The transformation tuning explores combinations of:

alpha: 0.00, 0.20, 0.40, 0.60, 0.80

saturation midpoint multiplier: 0.75, 1.00, 1.25

saturation slope multiplier: 0.50, 1.00, 2.00, 4.00

Reference statistics used by saturation are recomputed inside each training fold during rolling validation.

8.4 Price Control

AVG_Price_Per_kgs is retained as a demand control.

The modeled price variable is centered using a training-referenced mean:

$$
Price_{Centered} = Price - \overline{Price}_{training}
$$

Inner validation folds use fold-specific training means.

8.5 Trend and Seasonality

The trend is deterministic:

$$
Trend_t = Week_t - Week_{first}
$$

The notebook can include Fourier seasonal controls such as:

Fourier_Sin_1

Fourier_Cos_1

Legacy names such as Sin_52 and Cos_52 are also accommodated when available.

9. 📐 OLS MMM Specification

The model uses an intercept plus the five saturated media-family variables and available controls.

Media block

Print_Saturated
TV_Saturated
Meta_Saturated
Instagram_Saturated
YouTube_Saturated

Core OLS equation

The fitted MMM can be written generically as:

\beta_0
+
\sum_{j=1}^{J} \beta_j X_{j,t}^{sat}
+
\sum_{m=1}^{M} \gamma_m Z_{m,t}
+
\varepsilon_t
$$

where:

$Y_t$ = weekly sales volume (Sales_Volume_kgs);

$X_{j,t}^{sat}$ = saturated media-family input for family $j$;

$Z_{m,t}$ = control variables such as price, promotion, trend, holidays, and Fourier terms;

$\beta_j$ = media-family coefficient;

$\gamma_m$ = control coefficient;

$\varepsilon_t$ = model error.



Control block

Depending on available columns, controls can include:

Price_Mean_Centered

Holiday_*

Promotion_*

TradePromo_*

Trend_Centered or Trend

Fourier_Sin_1

Fourier_Cos_1

legacy Fourier names when present

The model is fitted with statsmodels OLS and an intercept.

10. 🧪 Time-Series Validation

10.1 Outer split

The notebook uses a chronological split:

First 80%  → Development / Training
Final 20%  → Untouched Holdout

The final holdout is not used for transformation tuning or development-model fitting.

10.2 Inner rolling cross-validation

Within the development period, chronological validation folds are created around:

55% of development data
70% of development data
85% of development data

with an approximately 10% validation length, while never crossing into the final holdout.

10.3 What happens inside each fold

For every candidate transformation and fold:

Apply candidate adstock.

Learn saturation statistics from training rows only.

Freeze saturation parameters.

Center price using the fold's training mean.

Fit OLS on the fold's training rows.

Predict the chronological validation block.

Record validation RMSE.

The selected transformation is determined from this chronological development process.

10.4 Final holdout metrics

The notebook reports:

RMSE — root mean squared error;

MAE — mean absolute error;

MAPE — mean absolute percentage error;

WAPE — weighted absolute percentage error.

The holdout is the notebook's genuine evidence of out-of-sample predictive performance.

11. 🧩 Historical Attribution & Contribution

After the validation gate, the notebook refits the locked specification using all available history.

The resulting object is:

full_data_ols_model

This historical model supports:

weekly contribution decomposition;

fiscal-year contribution summaries;

incremental sales;

incremental revenue;

ROAS;

response curves;

budget optimization.

Media contribution

For media family $j$:

\beta_j \times X^{\mathrm{sat}}_{j,t}
$$

The decomposition is reconciled against the model's fitted sales prediction.

Governance note: This full-history refit is a historical/business-analysis model. It must not be presented as the source of the notebook's holdout-validation claim.

12. 💰 Incremental Sales, Revenue & ROAS

Incremental sales

The MMM reports media-attributed incremental sales in kilograms.

Revenue translation

Fiscal-year average realized price is calculated as:

\frac{\text{FY Sales Value}}{\text{FY Sales Volume}}
$$

Incremental media revenue is:

\text{Incremental Media Sales}
\times
\text{FY Average Price}
$$

ROAS

For media family $j$:

\frac{
\text{Incremental Media Revenue}_j
}{
\text{Media Spend}_j
}
$$

When spend is zero, ROAS is left as NaN rather than forcing a value.

ROAS is available at weekly, fiscal-year, family, total-media, and overall FY23–FY25 levels.

13. 📈 Response Curves

The response-curve layer makes the model's nonlinear media response visible.

Media Input
    ↓
Geometric Adstock
    ↓
Zero-Anchored Logistic Saturation
    ↓
OLS Media Coefficient
    ↓
Modeled Incremental Sales
    ↓
Revenue Translation

Interpretation

Response curves are primarily used to understand:

carryover effects;

diminishing returns;

response shape;

relative behavior of the five media families.

They should not be interpreted as equally reliable at every spend level; areas far outside the observed historical range are more extrapolative.

14. 🚀 Budget Optimization

Budget optimization is the final analytical layer of the notebook.

Optimization goal

Reallocate a fixed weekly media budget across the five media families to maximize modeled incremental media revenue.

Media families optimized:

Print

TV

Meta

Instagram

YouTube

Why optimization is not a simple ROAS ranking

Historical ROAS is an average over observed spend.

Optimization asks a different question:

What is the modeled incremental value of the next unit of spend at the proposed allocation?

Because the model contains saturation, the marginal return changes as spend increases. Therefore, the optimizer evaluates the nonlinear response rather than mechanically assigning all budget to the historically highest-average-ROAS family.

15. 🔗 Spend-to-Input Bridge

The MMM is trained on transformed media inputs, while the budget decision is expressed in INR spend. The notebook therefore estimates a historical bridge:

$$
\text{Model Input}
\approx
\text{Spend}_{INR} \times \text{InputPerINR}
$$

For each family, the optimizer records:

historical weeks with spend;

total historical spend;

total historical model input;

Input_Per_INR;

through-origin bridge R².

A non-positive bridge slope causes the optimization stage to stop rather than produce an unreliable allocation.

Planning assumption

The bridge assumes the historical spend-to-input relationship remains informative for the planning scenario.

16. 🎯 Optimization Objective & Constraints

16.1 Spend → Input

For media family $j$:

\text{Spend}_j \times \text{InputPerINR}_j
$$

16.2 Input → Steady-State Adstock

For constant weekly planning spend:

\frac{\text{Weekly Input}_j}{1-\alpha_j}
$$

16.3 Adstock → Saturation

The optimizer applies the same zero-anchored logistic saturation function used by the MMM:

$$
L_j(a) = \frac{1}{1 + e^{-k_j(a-x_{0,j})}}
$$

$$
S_j(a) = \frac{L_j(a)-L_j(0)}{1-L_j(0)}
$$

where $a$ is the steady-state adstock, $k_j$ is the fitted slope parameter, and $x_{0,j}$ is the fitted midpoint parameter. By construction, $S_j(0)=0$.

16.4 Saturation → Incremental Sales

\beta_j \times S_j
$$

16.5 Sales → Revenue

A common historical weighted-average planning price is used:

\frac{
\text{Total Sales Value}
}{
\text{Total Sales Volume}
}
$$

Then:

\text{Incremental Sales}_j \times \text{Planning Price}
$$

16.6 Optimization problem

$$
\boxed{
\max_{s_1,\ldots,s_J}
\sum_{j=1}^{J}
\text{Incremental Revenue}_j(s_j)
}
$$

subject to:

$$
\boxed{
\sum_{j=1}^{J} s_j = B
}
$$

and:

$$
\boxed{
0 \le s_j
\le
1.25 \times \text{Historical Max Weekly Spend}_j
}
$$

where B is the current historical average total weekly media budget.

16.7 Numerical optimizer

The notebook uses SciPy's constrained optimizer with SLSQP.

The optimizer:

starts from the current historical family-level average spend;

enforces non-negative spending;

enforces the 1.25× historical-maximum weekly spend cap;

preserves the fixed total weekly budget;

checks feasibility;

requires successful convergence before accepting the result.

17. ✅ Finalized Budget Result

The final notebook section is explicitly designed to produce a business-ready output:

FINALIZED MMM BUDGET OPTIMIZATION RESULT

Top-line outputs

The result reports:

current average weekly media budget;

optimized weekly media budget;

52-week annualized planning budget;

current-budget modeled incremental sales;

optimized modeled incremental sales;

modeled sales uplift per week and year;

current-budget modeled incremental revenue;

optimized modeled incremental revenue;

modeled revenue uplift per week and year;

current-budget scenario ROAS;

optimized scenario ROAS.

Family-level allocation table

The final allocation includes:

Field

Purpose

Media_Family

Business media family

Current_Avg_Weekly_Spend_INR

Historical average weekly spend

Current_Budget_Share_%

Historical budget share

Optimized_Weekly_Spend_INR

Model-implied optimized weekly spend

Optimized_Budget_Share_%

Optimized budget share

Spend_Change_INR_per_Week

Weekly rupee change vs. current

Spend_Change_%

Percentage change vs. current

Optimized_Annual_Spend_INR

52-week annualized spend

Marginal_Revenue_per_Additional_INR

Local modeled marginal revenue

OLS_Beta

Fitted media coefficient

Model_Response_Flag

Indicates modeled response eligibility/status

How to read the allocation

The final allocation should be interpreted as:

The media-family spend mix that maximizes modeled incremental media revenue under the fixed-budget, response-function, and extrapolation constraints implemented in the notebook.

It should not be interpreted as:

a guaranteed future sales result;

proof of causal lift independent of modeling assumptions;

a replacement for media buying judgment;

evidence that historical ROAS will remain constant as spend changes;

permission to exceed historical planning bounds without additional evidence.

18. 🩺 Diagnostics & Model Governance

The notebook includes a broad diagnostic layer covering:

heteroskedasticity;

autocorrelation;

Ljung–Box residual structure;

Breusch–Pagan testing;

Breusch–Godfrey testing;

Durbin–Watson;

Jarque–Bera normality diagnostics;

VIF / multicollinearity;

influence and outlier diagnostics;

RESET-style linear specification checks.

Coefficient-direction audit

The notebook reviews:

negative media coefficients;

unexpected price direction;

other coefficient-direction signals requiring investigation.

It does not force coefficients to expected business signs simply because a sign is inconvenient.

A coefficient sign should be evaluated in context of:

data coverage;

collinearity;

model specification;

transformation choices;

business knowledge.

19. 📦 Generated Outputs

All major artifacts are written to the mmm_outputs directory.

Core modeling

Consumer Brand_MMM_Master_Numeric.csv
Consumer Brand_MMM_Modeling_Base.csv
Consumer Brand_MMM_Model_Base_Adstocked.csv
Consumer Brand_MMM_Model_Base_Adstock_Saturated.csv
Consumer Brand_MMM_Final_Adstock_Saturated_Tuned.csv

Contribution & incremental sales

Consumer Brand_MMM_Weekly_Contribution.csv
Consumer Brand_MMM_FY23_FY24_FY25_Contribution.csv
Consumer Brand_MMM_Total_Contribution_FY23_FY25.csv
Consumer Brand_MMM_Weekly_Incremental_Sales_Revenue.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_Incremental_Sales.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_Incremental_Revenue.csv

Sales & ROAS

Consumer Brand_MMM_FY23_FY24_FY25_Sales_Revenue_Summary.csv
Consumer Brand_MMM_Weekly_Media_ROAS.csv
Consumer Brand_MMM_Weekly_ROAS_Summary.csv
Consumer Brand_MMM_FY_Media_ROAS.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_ROAS.csv
Consumer Brand_MMM_Overall_FY23_FY25_Media_ROAS.csv
Consumer Brand_MMM_FY23_FY24_FY25_Media_Spend.csv

Response curves

Consumer Brand_MMM_Response_Curves.csv
Consumer Brand_MMM_Response_Curve_Summary.csv

Diagnostics

Consumer Brand_MMM_Actual_Predicted_Residuals.csv

Budget optimization

Consumer Brand_MMM_Budget_Optimization_Parameters.csv
Consumer Brand_MMM_Optimized_Budget_Allocation.csv
Consumer Brand_MMM_Spend_to_Input_Bridge.csv
Consumer Brand_MMM_FINAL_Budget_Optimization_Result.csv
Consumer Brand_MMM_FINAL_Budget_Optimization_Summary.csv

⭐ Business users: Start with the two files beginning with FINAL_.

20. 📁 Project Structure

Recommended structure:

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

📐 Formula Rendering

All equations in this README use display-math blocks with $$ ... $$ and inline math with $ ... $, which are supported by modern GitHub Markdown renderers. The notation follows the formulas implemented in the finalized notebook.

21. ▶️ How to Run

Step 1 — Place the files together

Keep the notebook and the four source CSVs in the same directory.

The notebook checks the current working directory and then /mnt/data when searching for inputs.

Step 2 — Open the notebook

Compatible environments include:

Jupyter Notebook;

JupyterLab;

VS Code;

other standard Python/Jupyter environments.

Step 3 — Run sequentially

Run the notebook from top to bottom. Later business-output sections depend on objects created earlier.

Step 4 — Review the validation gate

Before using attribution or optimization outputs, review:

development-vs-holdout performance;

coefficient-direction diagnostics;

model diagnostics;

the distinction between development model and full-history refit.

Step 5 — Review final planning outputs

The final business-facing results are saved to:

mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Result.csv
mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Summary.csv

22. 🧰 Dependencies

Primary packages/modules include:

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

The notebook assumes the active environment already contains the required dependencies.

23. 🔍 Key Notebook Objects

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

Validation / model objects

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

24. ⚙️ Assumptions

24.1 Historical relationships remain useful for planning

The model assumes historical relationships provide useful information for the planning scenario.

24.2 Media input is a valid exposure/activity measure

The MMM is fit on media inputs. Spend is connected to those inputs through the historical spend-to-input bridge used for optimization.

24.3 Geometric adstock adequately represents carryover

Carryover is modeled through geometric decay.

24.4 Logistic saturation adequately represents diminishing returns

The response curve is constrained to the chosen zero-anchored logistic form.

24.5 Historical weighted-average price is sufficient for revenue translation

Optimization uses a common historical price only to translate incremental kilograms into revenue.

24.6 Steady-state adstock is suitable for planning

The optimizer evaluates a constant weekly spending scenario through steady-state adstock rather than a finite campaign schedule.

24.7 1.25× cap is an extrapolation guard

The family-level upper bound is a practical planning constraint designed to reduce excessive extrapolation. It is not a learned causal/business rule.

25. ⚠️ Limitations

Causality

OLS association does not by itself establish causal lift. Confounding, omitted variables, targeting effects, reverse causality, and measurement error may influence coefficients.

Collinearity

Coordinated media activity can create correlation among channels, making individual coefficients unstable.

Limited experimental identification

This is not a randomized experiment. Experimental or quasi-experimental evidence can strengthen causal interpretation.

Aggregation bias

Campaign-level heterogeneity is compressed into family-level weekly variables.

Optimization extrapolation

The optimizer is bounded, but an optimized mix can still differ materially from an exact historical weekly allocation.

Deterministic point estimates

The current implementation produces point-estimate planning outputs rather than posterior distributions or probabilistic uncertainty intervals.

Steady-state planning simplification

The current budget optimization is based on steady-state adstock. A finite-horizon stateful simulation would better represent ramp-up and carryover dynamics for campaign schedules.

26. 🛠️ Recommended Enhancements

Bayesian uncertainty layer

A PyMC-based extension could provide distributions for:

media contributions;

ROAS;

incremental sales;

incremental revenue;

response curves;

optimized budget allocation.

Scenario planning

Add multiple planning budgets, for example:

-20%   Current   +10%   +20%

and compare allocation and modeled response.

Finite-horizon optimization

Simulate 13-, 26-, or 52-week spend paths explicitly, carrying adstock state week by week.

Additional business constraints

Potential future controls include:

minimum spends;

maximum budget shares;

channel commitments;

inventory constraints;

production constraints;

campaign-level caps;

business-defined guardrails.

27. ✅ Reproducibility Checklist

Use this checklist before publishing or presenting the model.

All four required CSVs are present.

Dates and weeks parse correctly.

No duplicate weekly keys exist.

Source week coverage is aligned.

The development/holdout split remains chronological.

The final 20% holdout is untouched during tuning.

Media scaling is learned from the development period only.

Price centering is training-referenced.

Saturation statistics are fold-specific during rolling CV.

Holdout predictions are generated without refitting on holdout rows.

Historical refit occurs only after the validation gate.

Contribution decomposition reconciles to fitted sales.

Spend aligns to the MMM weeks.

Spend-to-input bridge slopes are positive for optimized families.

Optimization is feasible under the family caps.

SLSQP converges successfully.

Optimized spend equals the fixed planning budget.

Final optimization CSVs are generated.

28. 🗣️ Business Presentation Framework

A stakeholder presentation should follow this order:

1. Model credibility

Start with the chronological validation design and untouched holdout.

2. Response understanding

Explain adstock, carryover, saturation, and diminishing returns.

3. Historical efficiency

Review incremental revenue and ROAS by family and fiscal year.

4. Planning implication

Present the optimized fixed-budget allocation.

5. Sensitivity and caveats

Explain that the optimization is conditional on the MMM and should be monitored against future actual performance.

Recommended stakeholder statement

The finalized MMM converts historical weekly media, promotion, price, trend, and seasonality data into a leakage-safe OLS response model. After genuine out-of-sample validation, the locked specification is refit on all history for attribution and planning. The budget optimizer reallocates the historical average weekly media budget across Print, TV, Meta, Instagram, and YouTube using the fitted adstock and saturation response, while limiting extrapolation to 1.25× each family's historical maximum weekly spend. The resulting allocation is a model-implied planning scenario, not a guaranteed causal outcome.

29. 🧾 Final Interpretation

The final optimization answer should be summarized in three layers:

A. Current state

What is the existing weekly spend by family and how is the current budget distributed?

B. Optimized state

How does the model reallocate that same total weekly budget across Print, TV, Meta, Instagram, and YouTube?

C. Modeled business impact

What does the fitted response function imply for:

incremental sales;

incremental revenue;

budget share change;

marginal revenue;

scenario ROAS?

⚠️ Final decision rule

The optimization result is a model-implied planning scenario. It should be used together with:

validation performance;

coefficient stability;

diagnostics;

spend-to-input bridge quality;

historical support for the optimized spend range;

business constraints;

future test-and-learn evidence.

30. 📦 Deliverables

Primary notebook

MMM_Leakage_Safe_Case_Study_Finalized.ipynb

Primary README

README_MMM_Leakage_Safe_Case_Study.md

Primary optimization outputs

mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Result.csv
mmm_outputs/Consumer Brand_MMM_FINAL_Budget_Optimization_Summary.csv

Supporting optimization outputs

mmm_outputs/Consumer Brand_MMM_Budget_Optimization_Parameters.csv
mmm_outputs/Consumer Brand_MMM_Optimized_Budget_Allocation.csv
mmm_outputs/Consumer Brand_MMM_Spend_to_Input_Bridge.csv

🏁 Final Project Statement

This case study demonstrates a complete leakage-safe weekly MMM workflow: structured data preparation → family-level media modeling → causal-safe transformation handling → chronological rolling validation → untouched holdout evaluation → historical contribution and ROAS → nonlinear response curves → constrained fixed-budget optimization.

The key methodological principle is simple:

Do not let future information influence model choices, transformation parameters, or validation.

The key business principle is equally important:

Optimize on modeled marginal response and diminishing returns — not on historical average ROAS alone.

🔖 Version Note

This README documents the finalized notebook structure and optimization methodology supplied with the case study. Numerical allocation, modeled revenue uplift, ROAS, and channel-level planning values are generated from the source data when the notebook is executed end to end.

<p align="center">
  <strong>Marketing Mix Modeling • Leakage-Safe Validation • Response Curves • Budget Optimization</strong>
</p>
