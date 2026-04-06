**Model Card: Bayesian Optimization**

**1. Model Overview**

**Model name:** Diagnostic-Hybrid Sequential GP Optimizer

**Version:** Week 10

**Developer(s):** Student capstone project

**Contact information:** N/A

**License:** Educational use only

**Brief description:** Bayesian Optimization using Gaussian Process
regression with Matern kernel. Evolved from standard GP-EI exploitation
(Weeks 1-5) to diagnostic-hybrid validation-first methodology (Week 10).
Generates 30-290+ candidates per function, applies sensitivity analysis
to all, validates structural consistency, then selects best validated
option (refinement over suggestion, not blind trust).

**2. Intended Use**

**Primary task:** Sequential optimization of expensive black-box
functions (2D-8D) under limited query budgets where avoiding
catastrophic failures is more important than comprehensive exploration

**Target users:** Researchers working with expensive simulations,
hyperparameter tuning, or experimental design where query costs are high
and validation is feasible

**Recommended use cases:**

-   Boundary-sensitive optimization problems

-   Scenarios requiring validation of model suggestions before execution

-   Late-stage optimization when near budget limits

-   Educational exploration of BO pitfalls and diagnostic frameworks

-   Problems where false positives (bad queries) are more costly than
    false negatives (missed opportunities)

**Not recommended for:**

-   Comprehensive interior space coverage (suffers from boundary
    blindness)

-   Early-stage exploration with sparse data (validation may block
    useful exploration)

-   High-dimensional spaces (\>8D) without additional techniques

-   Problems where true optimum likely lies in interior regions
    \[0.3-0.7\]\^n

-   Scenarios where validation computation is comparable to function
    evaluation cost

-   Unbiased systematic space-filling (approach has strong path
    dependency from Week 8 boundary success)

**When to disable/reduce validation:** (1) Early optimization stages
with very sparse data, (2) when exploration is more valuable than
avoiding errors, (3) when rejecting too many promising candidates

**3. Training Data**

**Data sources:** Sequential query evaluations across 8 black-box
functions over 10 weeks

**Size of dataset:** 80 submitted evaluations (10 per function) + 290+
Week 10 internal validation candidates. Severe sampling bias: F5 has
9/10 points at boundaries, only 1 point in quad boundary region.

**Languages or modalities:** Continuous numerical inputs \[0,1\]\^n,
scalar outputs (varying ranges: F1≈0, F2∈\[0,0.83\], F3∈\[-0.4,-0.006\],
F4∈\[-31,0.6\], F5∈\[5,4518\], F6∈\[-2.6,-0.18\], F7∈\[0.01,2.3\],
F8∈\[5.5,9.9\])

**Preprocessing steps:** Log-transformation for F5 only via log(y+1) to
handle exponential outputs. Finite difference gradient computation ∇f ≈
(f(x+ε)-f(x-ε))/(2ε) with ε=1e-6 for sensitivity analysis. GP output
normalization (sklearn normalize_y=True).

**4. Evaluation Metrics**

**Metrics used:** Week-over-week improvement, success rate (% functions
improving/maintaining best), absolute improvement over baseline, gap to
known better solutions where available

**Performance results:**

-   Weeks 1-8: 75% average success rate

-   Week 9: 25% (catastrophic failure from blind GP trust)

-   Week 10: 75-88% expected (validation-first approach, actual results
    pending)

-   F5: predicted 4518 (+1.5% over Week 8 best 4450)

-   F2: predicted 0.834 (+0.6% over Week 6 best 0.829)

**Fairness or bias checks:**

-   **Boundary blindness:** F5 has 9/10 points with at least one input
    at 1.0; interior space systematically undersampled

-   **Path dependency:** Week 8 boundary success led to 28/35 Week 10 F5
    strategies being boundary-focused

-   **Time allocation bias:** F5/F2 received 70% of Week 10 analysis
    effort

-   **Sparse data circularity:** Validation relies on GP gradients in
    regions with only 1-2 nearby data points

**5. Ethical Considerations**

**Potential biases or risks:**

-   Confirmation bias (boundary-focused after Week 8 success)

-   **Sparse data circularity:** GP over-trust in unexplored regions -
    validation uses GP gradients where GP itself is unreliable (e.g., 29
    points in 4D F5 space)

-   Rejected quad boundary \[1.0,1.0,1.0,1.0\] based on uncertain
    predictions in barely-explored region

-   Time allocation bias (F5/F2 focus left F4/F6 underexplored)

**Mitigation strategies:**

-   Sensitivity analysis on all 290+ candidates before selection

-   Multiple candidate generation per function (30-290+ options tested)

-   Validation thresholds: gradient \<-0.15 = strong reject, \<-0.05 =
    moderate caution

-   Documented all tested strategies for transparency

**Critical limitation - Reproducibility caveat:** Validation thresholds
(gradient \<-0.15 = reject) are not derived from theory or empirical
analysis. They are judgment calls. A different researcher using the same
data might reasonably choose different thresholds (e.g., -0.10 instead
of -0.15) and reach different conclusions about which candidates to
select. This limits reproducibility despite transparent documentation.
The validation rules reflect personal interpretation of \"don\'t move
away from boundaries\" rather than objective mathematical criteria.

**When to adjust validation rigor:**

-   **Reduce validation strictness when:** (1) exploration is more
    valuable than error avoidance, (2) early in optimization with sparse
    data, (3) validation is rejecting most candidates

-   **Increase validation strictness when:** (1) function evaluations
    are extremely expensive, (2) near end of query budget, (3)
    consequences of bad queries are severe

-   **Current approach may be over-validating** in data-sparse regions
    like F5 quad boundary, potentially blocking useful exploration paths

**Privacy concerns:** None (synthetic functions, no personal data)

**6. Model Life Cycle**

**Date of last update:** Week 10 (final submission)

**Version control or repository:** Weekly iterations (Weeks 0-10)
document strategy evolution: random initialization (Week 0) → GP-EI
exploitation (Weeks 1-5) → local refinement (Weeks 6-8) → failed
ensemble (Week 9) → diagnostic-hybrid validation-first (Week 10)

**Monitoring plan:** Week 11 results will validate whether diagnostic
approach represents genuine improvement or overfitting to GP noise in
sparse regions.

**CRITICAL TEST - Validation of Entire Methodology**

**The F5 boundary strategy serves as the definitive test** of whether
sensitivity-based validation provides genuine insight or creates false
confidence in data-sparse regions.

**Week 10 decision under scrutiny:**

-   Selected: \[0.200, 1.0, 0.995, 1.0\] (Input 3 at 0.995, not 1.0)

-   Rationale: Sensitivity analysis showed gradient -0.26 at Input
    3=1.0, suggesting move away from exact boundary

-   Rejected: \[1.0, 1.0, 1.0, 1.0\] (quad boundary) - predicted only
    580 vs 4518 for selected option

-   Data context: Only 1 point near quad boundary; validation relied on
    GP extrapolation in nearly unexplored region

**Interpretation of Week 11 F5 results:**

**If F5 ≥ 4450 (maintains or improves):**

-   **Confirms:** Sensitivity analysis successfully detected real
    function structure (Input 3 at 0.995 relieves gradient tension)

-   **Validates:** Diagnostic-hybrid methodology prevents disasters
    while enabling refinement

-   **Supports:** Week 10 approach was genuine improvement over Week
    9\'s blind GP trust

**If F5 \< 4400 (significant decline):**

-   **Confirms:** Model overfitted to GP noise in sparse boundary
    regions

-   **Indicates:** The -0.26 gradient at Input 3=1.0 was GP artifact,
    not real structure

-   **Reveals:** Validation framework created false confidence -
    rejected quad boundary \[1.0,1.0,1.0,1.0\] which may have been the
    true path to higher values (6000+)

-   **Demonstrates:** Sparse data circularity is fatal flaw - cannot
    reliably validate using unreliable model

**If F5 ∈ \[4400-4449\] (marginal decline):**

-   **Suggests:** Refinement from 1.0 to 0.995 was noise, not signal

-   **Highlights:** Precision of gradient-based boundary adjustment
    exceeds GP reliability in sparse regions

-   **Questions:** Whether validation adds value beyond simpler
    approaches (e.g., small random perturbations)

**This test matters because:** F5 represents the core innovation of Week
10 (sensitivity-guided boundary refinement). If this fails, the entire
diagnostic validation framework is called into question. Week 11 results
will definitively answer whether validation-first methodology is
sophisticated improvement or elaborate rationalization of GP noise.

**Key Takeaway for Users**

This is a diagnostic-hybrid approach that prioritizes error prevention
over comprehensive exploration. It excels at refining known good regions
but suffers from boundary blindness (assumes optima near boundaries),
sparse data circularity (validates using unreliable model), and path
dependency (constrained by Week 8 success).

**Use when:** Query costs are high and you\'re refining near-optimal
solutions.\
**Avoid when:** You need unbiased space coverage or suspect interior
optima.\
**Remember:** Different validation thresholds yield different results -
this is documented methodology, not deterministic algorithm.
