# Model Card

## Overview

**Model Name:** Diagnostic-Hybrid Gaussian Process Optimizer (DH-GP-BO)

**Model Type:** Gaussian Process Regressor (GPR) with validation-first methodology. It utilizes Gaussian Process as a probabilistic surrogate model to map the input X space to objective y response, applies sensitivity-based validation to candidate queries, and uses Expected Improvement acquisition function for selection.

**Version:** 1.0 (13-week competition)

## Intended Use

The model is intended for sequential optimization of expensive black-box functions where the analytical form of the target function is unknown, evaluation budget is strictly limited to 13 weekly queries, and avoiding catastrophic failures is more important than comprehensive exploration.

**Use cases:**

- **Hyperparameter Tuning:** Optimizing machine learning model parameters (e.g., learning rate, regularization strength) when training runs are expensive and query budget is limited
- **Expensive Black-Box Function Optimization:** Use when evaluating the target function is expensive in terms of cost, time, or resources, and validation of suggestions is feasible before execution
- **Experimental Design:** Search for optimal experimental conditions like temperature, pressure, chemical concentration where physical experiments are costly and failure consequences are severe
- **Boundary-Sensitive Problems:** Optimization where optima lie near parameter limits or boundaries, such as maximum material properties or extreme process conditions
- **Late-Stage Refinement:** Use in final optimization stages when refining known good regions rather than broad exploration

**Avoid using the model for:**

- High-dimensional input spaces (>8D) without trust region methods, as it suffers from curse of dimensionality
- Early exploration stages with very sparse data (<5 observations) where validation may block useful exploration
- Problems where true optimum likely lies in interior regions [0.3-0.7]^n rather than boundaries
- Scenarios requiring unbiased space-filling coverage for scientific understanding rather than optimization
- Real-time optimization where validation computation cost is comparable to evaluation cost

## Details

The optimization approach evolved through three distinct phases over 13 weeks. Initially (Weeks 1-8), I employed standard Gaussian Process regression with Matérn kernel (ν=2.5) and Expected Improvement acquisition function, focusing on exploitation with exploration parameter ξ=0.01. This phase achieved 75% success rate, with consistent week-over-week improvements across functions. I spent early weeks tuning kernel parameters including length scales and noise levels (α=1e-6 for near-deterministic assumption), while monitoring GP predictions to understand the data landscape.

Week 9 represented a failed experiment where I attempted ensemble validation combining GP-EI, Random Forest, and Thompson Sampling, resulting in catastrophic 25% success rate. This failure revealed the danger of blind model trust without structural validation, particularly when GP suggestions moved away from established patterns. The ensemble over-trusted models in unexplored regions, leading to six functions declining simultaneously.

In response, I developed the diagnostic-hybrid validation-first methodology (Weeks 10-13). This approach generates 30-290+ candidate queries per function rather than accepting a single GP suggestion. For each candidate, I apply sensitivity analysis using finite difference gradients (∇f ≈ (f(x+ε)-f(x-ε))/(2ε) with ε=1e-6) to detect structural inconsistencies. Validation thresholds (gradient <-0.15 = strong reject, <-0.05 = moderate caution) filter candidates that would move away from established good regions. Only candidates passing validation are selected for submission. Special preprocessing includes log-transformation for exponential functions (F5: log(y+1)) to maintain GP numerical stability. For late-stage optimization (Weeks 12-13), I implemented defensive locking strategies where functions with validated peaks submit identical coordinates to preserve wins, while one high-conviction bet (F5 extrapolation) explores beyond assumed constraints. I maintained weekly submission records tracking validation decisions, rejected strategies, and rationale for transparency and reproducibility.

## Performance

| Function | Dimensions | Initial Best (y) | Week 13 Best (y) | Strategy Used | Competition Rank |
|----------|-----------|------------------|------------------|---------------|------------------|
| F1: Radiation Source | 2D | 0.00360 | ≈0 (3.84×10⁻²⁶) | Standard GP-EI | 50/65 |
| F2: Bimodal Function | 2D | 0.61121 | 0.866 | GP-EI + K-means Clustering | **1/65** 🥇 |
| F3: Narrow Optimum | 3D | -0.34000 | -0.00571 | GP-EI + Thompson Sampling | 18/65 |
| F4: Volatile Function | 4D | -6.70209 | 0.6092 | Manual (GP failed, 42% outliers) | 12/65 |
| F5: Exponential Growth | 4D | 1088.86 | **2,686,080** | GP-EI + Extrapolation | **2/65** 🥈 |
| F6: Unstable Function | 5D | -0.71427 | -0.150 | Standard GP-EI | 12/65 |
| F7: 6D Smooth | 6D | 1.36497 | 3.166 | Standard GP-EI | **5/65** 🏅 |
| F8: High-Dimensional | 8D | 5.50000 | 9.885 | Standard GP-EI | 41/65 |

**Note on F2:** K-means clustering (k=2) in Week 13 revealed bimodal structure with two distinct peaks at [0.50, 0.49] → 0.866 (Peak 1) and [0.71, 0.93] → 0.55 (Peak 2). Standard GP alone would have suggested valley points between peaks. Clustering diagnostic prevented this trap, leading to 1st place.

**Note on F5:** Exponential growth discovered through boundary sensitivity analysis. Week 8 triple boundary [0.212, 1.0, 1.0, 1.0] → 4,450 validated pattern. Weeks 11-13 extrapolated beyond assumed [0,1] constraint: [1.05] → 13,337 (+200%), [1.5] → 263,397 (+1,875%), [2.0] → 2,686,080 (+920%). Validation-first methodology enabled confident aggressive extrapolation, achieving 2nd place (only beaten by 1 competitor).

**Note on F4:** DBSCAN outlier detection revealed 42% of observations were outliers, including catastrophic -16.65 value (Week 6). Standard GP assumptions (Gaussian noise, α=1e-6) fundamentally violated. Switched to manual methods Week 13, recovering to Week 7 best value.

## Assumptions and Limitations

The model assumes query search space is continuous and relatively smooth, with the expectation that points close to each other yield similar output results. A stationary Matérn kernel (ν=2.5) is used assuming consistent roughness across the search space. The model assumes near-deterministic evaluations (noise parameter α=1e-6), which proved accurate for most functions but failed catastrophically on F4 with 42% outliers. 

Key limitations include boundary blindness—9 out of 10 F5 observations had at least one input at 1.0 boundary, systematically undersampling interior regions [0.3-0.7]^n. The approach exhibits path dependency where Week 8 boundary success locked subsequent strategy into boundary-focused optimization, with 70% of Week 10 analysis concentrated on boundary-sensitive functions. Sparse data circularity creates a validation paradox: using GP gradients to validate GP suggestions in regions with limited data (e.g., only 29 observations in 4D F5 space) can create false confidence.

Validation thresholds (gradient <-0.15 = reject) are judgment calls rather than theoretically derived values, limiting reproducibility—different researchers using identical data might select different candidates with different threshold choices. The model performs poorly beyond 6-7 dimensions without specialized methods like trust regions (F8: 41st place in 8D), and standard GP fails on multimodal functions without diagnostic clustering (F2 bimodal structure invisible for 12 weeks).

## Constraints

- Query search space is constrained to [0,1] hypercube with validated scope for extrapolation (F5 extended to [2.0] based on exponential pattern evidence)
- Query budget strictly limited to 13 weekly submissions (1 query per function per week, no recovery rounds)
- Validation computation must complete within week between submissions
- Dimensionality effectively limited to 2-8D (optimal performance 2-6D)
- Assumes access to all historical query results for GP fitting

## Ethical Considerations

**Transparency:** The project maintains complete documentation of all 104 queries (8 functions × 13 weeks) with detailed rationale for each decision including validation thresholds applied, rejected strategies, and alternative candidates considered. Weekly submission records and function-specific datasheets enable full traceability and reproducibility of the optimization journey. However, validation thresholds remain subjective judgment calls rather than objective mathematical criteria, and this subjectivity is explicitly documented rather than hidden.

**Validation Subjectivity and Reproducibility:** While the validation-first methodology is transparently documented, the specific thresholds (gradient <-0.15 = reject, <-0.05 = caution) reflect personal interpretation of "don't move away from boundaries" rather than empirical derivation. Different researchers using identical data but different thresholds would reach different conclusions about which candidates to select. This limits reproducibility despite transparent methodology and represents an ethical consideration for scientific applications requiring objective reproducibility.

**Safety and Real-World Adaptation:** In real-world scenarios, exponential growth can lead to equipment failure or safety hazards (e.g., runaway chemical reactions, resource exhaustion). The model applies log-transformation for exponential functions (F5: log(y+1)) to maintain GP numerical stability and prevent "explosive" suggestions in high-magnitude regions. However, the aggressive extrapolation strategy (F5: [2.0] producing 2.69M) that succeeded in competition could be dangerous in physical systems with genuine hard constraints. Users must validate that extrapolation beyond assumed bounds is physically safe before applying this strategy to real experiments.

**Bias Acknowledgment:** The optimization exhibits confirmed biases including boundary focus (9/10 F5 points at boundaries), path dependency from Week 8 success (locked into boundary strategies), and time allocation bias (F5/F2 received 70% attention leaving F4/F6/F8 underexplored). These biases are documented but not fully mitigated, representing inherent trade-offs in the validation-first approach that prioritizes error prevention over comprehensive exploration.
