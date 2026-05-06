# DataSheet

## Motivation

The purpose of this datasheet is to track the performance and convergence of Bayesian Optimisation (BO) on 8 unknown black-box functions as part of my capstone project for the ML/AI course at Imperial College Business School, London.

The datasheet supports the weekly sequential Black Box Optimisation process spanning 13 weeks. The running cumulative set of queries and results are fed to the Gaussian Process (GP) for surrogate model calibration, acquisition function evaluation, and convergence analysis. The dataset documents the evolution from standard GP-EI optimization (Weeks 1-8) through failed ensemble methods (Week 9) to diagnostic-hybrid validation-first methodology (Weeks 10-13).

## Composition

The data contains the input X - dimensional vector like [x1, x2, x3, ...] representing the query space for the BO function and output y - scalar output from the black-box function after submission.

**Dataset Components:**

* **Initial Baseline** - Week 1 random initialization for each function. These are the starting X and y values that seed the GP model before optimization begins.

* **Weekly Update** - X and y updates are accumulated every week. The dataset grows each week as queries and results are added to the cumulative dataset. By Week 13, each function has 13 observations (13 weeks × 1 query per function per week = 104 total queries across 8 functions).

* **Data Type** - X is floating point number (float64), y is floating point number (float64). Input dimensions vary by function: F1-F2 (2D), F3 (3D), F4-F5 (4D), F6 (5D), F7 (6D), F8 (8D).

* **Format** - The raw data are extracted from .msg email files containing weekly submission results, then stored in structured JSON format (`complete_13week_data.json`) and annotated CSV format (`complete_13week_results.csv`) for analysis.

**Function-Specific Composition:**

* F1 (2D): 13 points, outputs ≈0 (convergence to theoretical optimum)
* F2 (2D): 13 points, outputs [0.611, 0.866] (bimodal structure discovered Week 13)
* F3 (3D): 13 points, outputs [-0.34, -0.00571] (narrow optimum found Week 2)
* F4 (4D): 13 points, outputs [-16.65, 0.6092] (42% outliers detected)
* F5 (4D): 13 points, outputs [1089, 2,686,080] (exponential growth with extrapolation)
* F6 (5D): 13 points, outputs [-2.6, -0.150] (unstable with late catastrophic failures)
* F7 (6D): 13 points, outputs [0.01, 3.166] (smooth consistent improvement)
* F8 (8D): 13 points, outputs [5.5, 9.885] (high-dimensional complexity)

**Sampling Characteristics:** Heavy boundary bias observed, particularly for F5 where 9 out of 13 points have at least one input at 1.0 boundary. Interior space [0.3-0.7]^n systematically undersampled across all functions due to path dependency from Week 8 boundary success.

## Collection Process

The dataset is collected iteratively week by week using Gaussian Process Bayesian Optimisation with evolving methodology across 13 weeks.

**Standard GP-EI Phase (Weeks 1-8):**
* Each week, the cumulative dataset (all previous weeks' observations) is fed to GP-BO surrogate model
* GP model uses Matérn kernel (ν=2.5) with optimized length scales and noise parameter α=1e-6
* The model estimates mean μ(x) and uncertainty σ(x) across the search space
* Expected Improvement acquisition function (ξ=0.01) generates 1,000 candidate points, selecting the maximum EI as next query
* Submit the new query to black-box function, receive result, and process loop starts again for next week
* This phase achieved 75% success rate with consistent week-over-week improvements

**Failed Ensemble Phase (Week 9):**
* Attempted multi-model validation combining GP-EI, Random Forest, and Thompson Sampling
* Resulted in catastrophic 25% success rate due to blind model trust without structural validation

**Diagnostic-Hybrid Validation-First Phase (Weeks 10-13):**
* Each week, GP model generates 30-290+ candidate queries instead of single suggestion
* All candidates undergo sensitivity analysis using finite difference gradients: ∇f ≈ (f(x+ε)-f(x-ε))/(2ε) with ε=1e-6
* Validation thresholds filter candidates: gradient <-0.15 = strong reject, <-0.05 = moderate caution
* Only candidates passing structural consistency checks are selected for submission
* Additional diagnostics include k-means clustering for multimodal detection (F2) and DBSCAN for outlier identification (F4)
* Submit validated query, receive result, update cumulative dataset for next week

**Extrapolation Discovery (F5, Weeks 11-13):**
* Week 11: Tested beyond assumed [0,1] constraint with [1.05, 1.05, 1.05, 1.05] → 13,337
* Week 12: Aggressive [1.5, 1.5, 1.5, 1.5] → 263,397 validated exponential pattern
* Week 13: Final [2.0, 2.0, 2.0, 2.0] → 2,686,080 secured 2nd place ranking

## Preprocessing and Uses

**Preprocessing Applied:**

* **Logarithmic transform** - log(y+1) is applied to Function 5 which exhibits exponential range of y values (1089 to 2.69M). This allows GP to model relative change in data landscape rather than being overwhelmed with huge magnitude of absolute values. No transformation applied to other functions.

* **Input space normalization** - X is constrained to [0,1] hypercube for all functions initially, with validated extrapolation up to [2.0] for Function 5 based on exponential pattern evidence from Weeks 8-12.

* **Gradient computation** - Finite difference sensitivity analysis applied to all Week 10-13 candidates to detect structural inconsistencies before submission.

* **Output normalization** - GP models use normalized outputs (sklearn's normalize_y=True) to ensure zero mean and unit variance for numerical stability.

* **Clustering analysis** - K-means (k=2) applied to F2 Week 13 to reveal bimodal structure with two distinct peaks. DBSCAN applied to F4 to detect 42% outlier rate causing GP failures.

**Intended Use for This Datasheet:**

* **Optimization convergence tracking** - Track how each week's BO results are converging toward optimal values across 8 functions with different characteristics

* **Method evolution documentation** - Provide traceable record of strategy changes from standard GP-EI (Weeks 1-8) to validation-first diagnostic approach (Weeks 10-13)

* **Diagnostic framework evaluation** - Assess effectiveness of sensitivity-based validation and clustering analysis in preventing catastrophic failures

* **Capstone documentation** - Provide complete transparent chronological log of 104 submissions (8 functions × 13 weeks) with rationale for each decision

* **Comparative analysis** - Enable comparison of different BO strategies including standard GP-EI (75% success), Thompson Sampling (50% success), and manual methods (37.5% Week 12)

* **Boundary vs interior optimization study** - Document boundary-focused strategy emergence and sampling bias toward parameter limits

* **Competition performance analysis** - Track results leading to 9th place overall with three podium finishes (1st F2, 2nd F5, 5th F7)

## Distribution and Maintenance

**Distribution:**

* The complete dataset is hosted in GitHub repository with structured documentation
* Dataset files include:
  * `complete_13week_data.json` - All 104 queries in structured format with metadata
  * `complete_13week_results.csv` - Annotated results with strategy notes and significant events
  * `BBO_Function_Datasheets.md` - Detailed per-function analysis for all 8 functions
  * `BBO_Model_Card.md` - Complete model documentation with architecture and performance

**Maintenance:**

* Weekly queries and results are documented in function-specific datasheets ensuring readable, transparent chronological log of submissions
* Each function's datasheet contains: optimization strategy used, weekly learning outcomes, validation decisions, rejected alternatives, and rationale
* Version history preserved through 13 weekly submissions (Week 1 through Week 13)
* Final dataset locked after Week 13 competition completion - no further updates planned
* Repository maintained for educational purposes and peer learning

**Transparency and Reproducibility:**

* Complete query history with input coordinates and output values for all 104 evaluations
* Strategy evolution documented: why methods changed, what validation thresholds were used, which candidates were rejected and why
* Subjective decisions explicitly documented including validation threshold choices (gradient <-0.15 vs <-0.05) and time allocation priorities (F5/F2 received 70% attention)
* Limitations acknowledged: boundary bias (9/10 F5 points at boundaries), sparse data circularity (validation in unexplored regions), path dependency (Week 8 success locked strategy)

**Access:**

* Repository publicly available for educational reference
* Data formatted for reproducibility - JSON for machine-readable queries, CSV for human-readable analysis
* Code implementation available in `bayesian_optimizer.py` for GP-EI methodology
* Jupyter notebooks document weekly decision-making process and validation framework evolution
