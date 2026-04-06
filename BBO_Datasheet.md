Datasheet: Bayesian Optimization Capstone Project Dataset

**Motivation**

**What task does this dataset help to solve?** This dataset supports the
study and evaluation of Bayesian Optimization (BO) strategies for
optimizing eight black-box functions with varying dimensionality (2D to
8D) and characteristics. The primary task is finding optimal input
configurations that maximize or minimize objective function values
through sequential query selection.

**Who created it, and why?** Created as part of an educational capstone
project to explore practical challenges in black-box optimization,
including: handling functions with different landscape characteristics,
balancing exploration vs exploitation, managing high-dimensional spaces,
and developing diagnostic validation frameworks.

**Was it funded or supported by an organization?** No external funding.
This is an individual educational project.

**Composition**

**What does the dataset contain?**

-   **Total instances:** 80 query-response pairs (10 weeks × 8
    functions)

-   **Instance types:** Each instance consists of:

    -   Input vector: continuous values in \[0,1\] range

    -   Output scalar: objective function value (varies by function)

    -   Metadata: week number, function ID, strategy used

**Structure:**

-   **F1:** 10 points in 2D space, outputs ≈0 (minimization at maximum)

-   **F2:** 10 points in 2D space, outputs \[0, 0.829\]

-   **F3:** 10 points in 3D space, outputs \[-0.4, -0.006\]

-   **F4:** 10 points in 4D space, outputs \[-31, 0.6\] (highly
    volatile)

-   **F5:** 10 points in 4D space, outputs \[5, 4518\] (exponential
    growth)

-   **F6:** 10 points in 5D space, outputs \[-2.6, -0.178\]

-   **F7:** 10 points in 6D space, outputs \[0.01, 2.3\]

-   **F8:** 10 points in 8D space, outputs \[5.5, 9.9\]

**Completeness:** Complete for the 10-week timeframe, but highly
incomplete coverage of the input space. F5 (4D) has only 10 points in a
space with infinite continuous combinations.

**Sampling representativeness:** NOT representative. Heavy sampling bias
toward:

-   Boundary regions (especially F5: 9/10 points have at least one input
    at 1.0)

-   Previously successful regions (local refinement around Week 6\'s F2
    peak)

-   Minimal interior space exploration

**Missing data:** No missing values in collected data, but vast
unexplored regions in all functions.

**Relationships:** Sequential dependency---each week\'s query depends on
all previous observations through GP modeling.

**Data splits:** Natural temporal split (Weeks 0-9 for training GP, Week
10 for validation), but no formal train/test separation.

**Subpopulations:** Can be grouped by:

-   Strategy type (random, GP-EI, local search, boundary exploration,
    diagnostic validation)

-   Dimensionality (2D, 3D, 4D, 5D, 6D, 8D)

-   Function characteristics (smooth vs volatile, exponential vs linear)

**Identifiability:** No personal data; all instances are synthetic
function evaluations.

**Collection Process**

**How was data acquired?** Sequential active learning process over 10
weeks:

-   **Week 0:** Random initialization (1 point per function)

-   **Weeks 1-5:** GP-based Expected Improvement (EI) acquisition

-   **Weeks 6-8:** Local refinement and boundary exploration

-   **Week 9:** Ensemble methods and parameter tuning (failed)

-   **Week 10:** Diagnostic validation with sensitivity analysis

**Sampling strategy:**

-   **Deterministic:** GP-EI optimization with random restarts

-   **Adaptive:** Strategy evolved based on observed patterns (boundary
    success → boundary focus)

-   **Biased:** 70% of Week 10 analysis effort on F2 and F5

**Time frame:** Approximately 10-week period (Weeks 0-10)

**Ethical considerations:** Not applicable (synthetic functions, no
human subjects)

**Impact assessment:** No direct impact on individuals; educational
exercise only

**Preprocessing/Cleaning/Labeling**

**Transformations applied:**

1.  **Log-transformation:** F5 outputs transformed via log(y+1) for GP
    modeling to handle exponential scale

2.  **Gradient computation:** Finite difference sensitivity analysis: ∇f
    ≈ (f(x+ε) - f(x-ε))/(2ε) with ε=1e-6

3.  **Normalization:** GP models use normalized outputs (sklearn\'s
    normalize_y=True)

**Raw data preservation:** No. Transformations applied in-memory; only
final query-response pairs stored.

**Labeling:** No manual labeling. Output values are direct function
evaluations.

**Uses**

**Intended uses:**

-   Comparative analysis of BO acquisition functions (EI, Thompson
    Sampling, local search)

-   Study of boundary optimization patterns in constrained spaces

-   Evaluation of diagnostic validation frameworks (sensitivity
    analysis, gradient-based validation)

-   Educational exploration of BO pitfalls (confirmation bias, sparse
    sampling, GP over-reliance)

**Inappropriate uses:**

-   Generalizing findings to other function classes without validation

-   Claiming optimal BO strategy based on 10-week sample

-   Using as benchmark for comparing BO algorithms (heavily biased,
    non-standard)

-   Drawing conclusions about interior regions (vastly undersampled)

**Risks and biases:**

-   **Confirmation bias:** Boundary success led to boundary-focused
    strategy, potentially missing interior optima

-   **Sampling bias:** 28/35 F5 test strategies were boundary-focused;
    interior space neglected

-   **GP over-trust:** Week 10 validation assumes GP gradients in sparse
    regions reflect true function structure

-   **Hindsight bias:** Week 10 diagnostic framework developed after
    seeing Week 9 failure

**Fairness considerations:** Not applicable (no demographic data, no
social impact)

**Distribution**

**Distribution plan:** Submitted via course portal; not publicly
distributed

**Access method:** Restricted to course instructors and evaluation
system

**Availability:** Course duration only

**Licensing:** Educational use only; not licensed for redistribution

**Fees:** None

**Intellectual property:** Student work submitted for educational
assessment

**Maintenance**

**Responsible parties:** Individual student (me) for course duration

**Version control:** Weekly submissions (Weeks 0-10) serve as version
history

**Updates:** No planned updates after Week 10 submission

**Archival:** Course portal maintains submissions; no long-term personal
archive planned

**Maintenance policies:** None beyond course requirements

**Additional Comments**

**Critical limitations identified:**

1.  **Sparse sampling:** 10 points in 8D space (F8) provides negligible
    coverage

2.  **Strategy evolution bias:** Each week\'s strategy influenced by
    previous weeks, creating path dependency

3.  **GP uncertainty in sparse regions:** Validation framework trusts GP
    gradients where data is sparse (e.g., quad boundary region in F5)

4.  **No ground truth:** True function optima unknown; cannot validate
    if \"best found\" equals \"true best\"

5.  **Reproducibility concerns:** Random seed selection, GP
    hyperparameter choices, and validation thresholds involve subjective
    decisions not fully documented

**Transparency gaps:**

-   Why gradient threshold -0.05 vs -0.15 for validation?

-   How were 35 F5 test strategies selected from infinite possibilities?

-   What justified log-transformation for F5 but not F3 or F7?

This dataset documents a learning process in BO strategy development
rather than a systematic benchmark. It reveals as much about human
decision-making biases as about function optimization.
