# Black-Box Bayesian Optimization: 13-Week Capstone Project

## Overview

Many real-world problems involve complex systems where we can't see inside – we can only try different inputs and observe the results. This is called "black box optimisation." In this project, I built a smart search tool that efficiently finds the best possible settings or designs without needing thousands of random trials.

Using modern techniques like Bayesian optimisation, the system learns from each test it runs. It intelligently balances exploring new possibilities with focusing on promising areas, much like a skilled researcher narrowing down options.

## DATA

### Dataset Overview
- **Source:** Imperial College Business School Executive Education - Machine Learning Module 24
- **Collection Period:** January - April 2026 (13 weeks)
- **Size:** 104 total queries (8 functions × 13 weeks)
- **Format:** `.msg` email files containing input coordinates and black-box evaluation results

### Dataset Structure
Each week, I submitted one query per function and received evaluation results:

| Function | Dimensions | Observations | Best Result | Description |
|----------|-----------|--------------|-------------|-------------|
| F1 | 2D | 13 | ≈0 (optimal) | Smooth unimodal |
| F2 | 2D | 13 | 0.866 | Bimodal with two peaks |
| F3 | 3D | 13 | -0.00571 | Narrow optimum, 2/3 dims irrelevant |
| F4 | 4D | 13 | 0.6092 | Volatile with 42% outliers |
| F5 | 4D | 13 | **2,686,080** | Exponential growth beyond bounds |
| F6 | 5D | 13 | -0.150 | Unstable, late catastrophic failures |
| F7 | 6D | 13 | 3.166 | Smooth, consistent improvement |
| F8 | 8D | 13 | 9.885 | High-dimensional, true 8D structure |

### Data Processing
- **Extraction:** Python `extract-msg` library parsed email bodies
- **Format Conversion:** Regex patterns extracted coordinates and values
- **Validation:** Cross-referenced against original submission confirmations
- **Storage:** JSON (structured, machine-readable) and CSV (annotated, human-readable)

### Data Files
- `complete_13week_data.json` - All 104 queries in structured format
- `complete_13week_results.csv` - Annotated with strategy notes and significant events
- `BBO_Function_Datasheets.md` - Detailed per-function analysis and learning

### Data Characteristics
- **Deterministic:** Most functions (F1, F3, F5, F7, F8) showed consistent outputs for repeated inputs
- **Stochastic Exception:** F2 Week 12-13 anomaly (identical input, different outputs: 0.866 vs 0.613)
- **Noise Levels:** F4 exhibited extreme volatility with 42% outliers including catastrophic -16.65
- **Missing Data:** None - all 104 queries completed successfully

### Citation
If using this dataset:
```bibtex
@dataset{bbo_capstone_2026,
  title={Black-Box Bayesian Optimization: 13-Week Sequential Dataset},
  author={BBO Capstone Project},
  year={2026},
  publisher={GitHub},
  note={104 queries across 8 functions (2D-8D)}
}
```

## MODEL

### Primary Model: Gaussian Process with Expected Improvement

I chose **Bayesian Optimization** using a Gaussian Process (GP) surrogate model with Expected Improvement (EI) acquisition function as the foundation for this challenge.

#### Why This Model?

**Sample Efficiency:** With only 1 query per function per week (13 total), I needed a method that learns from every evaluation. GP-EI is specifically designed for expensive black-box optimization where function evaluations are costly. Each query improves the GP model, making it progressively better at predicting promising regions.

**Natural Exploration-Exploitation Balance:** The Expected Improvement acquisition function automatically balances:
- **Exploitation:** Query where the GP predicts high values (mean μ is high)
- **Exploration:** Query where the GP is uncertain (standard deviation σ is high)

This balance is crucial for 13-week horizons—early exploration maps the landscape, late exploitation refines the best regions.

**Robustness Across Dimensions:** The same GP-EI framework worked across 2D-8D functions without algorithm changes, only requiring different computational resources.

#### Architecture Details

**Gaussian Process Regressor:**
```python
from sklearn.gaussian_process import GaussianProcessRegressor
from sklearn.gaussian_process.kernels import Matern, ConstantKernel as C

kernel = C(1.0, (1e-3, 1e3)) * Matern(
    length_scale=0.3,
    length_scale_bounds=(1e-2, 1e1),
    nu=2.5  # Twice differentiable (smooth functions)
)

gp = GaussianProcessRegressor(
    kernel=kernel,
    alpha=1e-6,  # Minimal noise assumption
    n_restarts_optimizer=20,  # Robust kernel optimization
    normalize_y=True  # Zero mean, unit variance
)
```

**Expected Improvement Acquisition:**
```python
def expected_improvement(X, gp, best_y, xi=0.01):
    mu, sigma = gp.predict(X, return_std=True)
    improvement = mu - best_y - xi
    Z = improvement / sigma
    ei = improvement * norm.cdf(Z) + sigma * norm.pdf(Z)
    return ei
```

#### Alternative Methods Tested

**Thompson Sampling** (Weeks 10, 13):
- Used when stuck in local optima (F3)
- Samples from GP posterior instead of maximizing EI
- More exploratory, useful for escape scenarios
- **Performance:** 50% success rate vs 75% for GP-EI

**Manual Selection** (Week 12-13):
- Used for volatile functions where GP failed (F4)
- Conservative single-dimension nudges
- **Performance:** 37.5% success (Week 12) vs 75% for GP-EI
- **Learning:** Sophisticated manual strategies underperformed simple GP-EI

**Clustering Analysis** (Week 13):
- K-means (k=2) for multimodal detection (F2)
- DBSCAN for outlier identification (F4)
- Not a replacement for GP-EI but diagnostic tool before method selection

#### Model Performance

**Success Rates by Phase:**
- Weeks 1-3: 67% (exploration)
- Weeks 4-8: **75%** (GP-EI golden era) ⭐
- Week 9: 25% (over-trusted GP)
- Weeks 10-11: 62% (Thompson recovery)
- Week 12: 37.5% (failed manual experiments)
- Week 13: 62.5% (clustering + defensive)

**Overall:** GP-EI as foundation with function-specific adaptations achieved **9th place out of 65 competitors** with three podium finishes.

## HYPERPARAMETER OPTIMISATION

The model itself is a Bayesian Optimization system, so discussing "hyperparameter optimization" involves the meta-level decisions about how to configure the optimizer.

### GP Kernel Hyperparameters

**Length Scale (ℓ):**
- **Initial Value:** 0.3 for all dimensions
- **Optimization:** Maximum likelihood estimation (MLE) with 20 random restarts
- **Bounds:** [0.01, 10.0] to prevent overfitting or underfitting
- **Learning:** Week 13 ARD (Automatic Relevance Determination) analysis revealed:
  - F3: Only Dim 0 matters (ℓ=0.019), others irrelevant (ℓ=10.0)
  - F5: Only Dim 3 varies (ℓ=0.31), others frozen (ℓ=5-10)
  - F8: 6/8 dimensions critical (ℓ<0.03)

**Smoothness Parameter (ν):**
- **Fixed Value:** ν=2.5 (Matérn kernel twice differentiable)
- **Rationale:** Assumes smooth functions where gradients exist
- **Alternatives Considered:**
  - ν=1.5: Less smooth, more flexible (not tested)
  - ν=∞: RBF kernel, infinitely differentiable (too smooth, would overfit)

**Noise Parameter (α):**
- **Standard Value:** α=1e-6 (near-deterministic assumption)
- **Increased for F4:** α=1e-5 due to observed volatility
- **Learning:** Should have used robust GP with Student-t likelihood for F4's 42% outlier rate

### Acquisition Function Hyperparameters

**Exploration Parameter (ξ):**
- **Value:** ξ=0.01 (minimal exploration bonus)
- **Effect:** Heavily exploitation-biased
- **Trade-off:** 
  - Converges quickly on smooth functions (F7: 5th place)
  - Gets stuck in local optima on multimodal functions (F2, F3)
- **Alternative:** ξ=0.1 would increase exploration, tested implicitly through Thompson Sampling

**Candidate Sample Size (n):**
- **Value:** n=1,000 random candidates per iteration
- **Trade-off:**
  - More candidates: Better EI maximum found, slower computation
  - Fewer candidates: Faster but may miss optimal points
- **Optimization Method:** Random search vs gradient-based (L-BFGS-B)
  - Chose random for speed (<1 second per iteration)
  - Gradient-based would guarantee local EI optimum but 10× slower

### Method Selection Hyperparameters

**Dimension Freezing Threshold:**
- **Value:** Length scale > 5.0 indicates low sensitivity
- **Discovered:** Week 13 (too late to apply)
- **Impact:** F3 and F5 had 2-3 freezable dimensions—missed opportunity

**Outlier Detection Threshold:**
- **Value:** DBSCAN flagging >30% as outliers triggers method switch
- **Applied:** F4 had 42% outliers → switched from GP-EI to manual
- **Timing:** Week 13 discovery, should have been Week 6

**Stagnation Detection:**
- **Criterion:** 3 consecutive weeks without improvement
- **Action:** Switch from GP-EI to Thompson Sampling (exploration emphasis)
- **Applied:** F3 Week 10 successfully, F3 Week 13 unsuccessfully

### Meta-Hyperparameter: Strategy Adaptation Schedule

**Exploration → Exploitation Timeline:**
- Weeks 1-3: 60% exploration budget (broad GP-EI sampling)
- Weeks 4-8: 30% exploration budget (natural GP-EI balance)
- Weeks 9-11: 50% exploration (recovery mode—Thompson, clustering)
- Weeks 12-13: 5% exploration (defensive locks + one moonshot)

**Empirical Tuning:** No formal hyperparameter optimization (no validation set). All choices based on:
1. Week-over-week success rates
2. Peer strategy discussions
3. GP diagnostics (variance, length scales)
4. Competition rankings

### Hyperparameters Not Optimized (Missed Opportunities)

**Trust Region Radius:**
- Not implemented despite being standard for high-D optimization
- Peers used ±5% radius around best
- Would have improved F7, F8 performance

**Multi-start Restarts:**
- Used 20 kernel optimization restarts (good)
- Could have used 5-10 GP-EI restarts from different initializations to avoid local optima

**Ensemble Size:**
- Single GP model used
- Ensemble of 3-5 GPs would quantify epistemic uncertainty better
- Especially useful for detecting when GP is unreliable (F6 catastrophic failures)

## RESULTS

### Competition Performance

**Overall Ranking: 9th out of 65 competitors (Top 14%)**

**Individual Function Rankings:**
- 🥇 **F2: 1st place** (0.866) - K-means clustering revealed bimodal structure
- 🥈 **F5: 2nd place** (2,686,080) - Extrapolation beyond assumed [0,1] bounds
- 🏅 **F7: 5th place** (3.166) - Standard GP-EI worked perfectly
- **F4: 12th place** (0.6092) - Volatile function with 42% outliers
- **F6: 12th place** (-0.150) - Unstable, late failures
- **F3: 18th place** (-0.00571) - Early lucky discovery, never matched
- **F8: 41st place** (9.885) - 8D needed TuRBO, didn't implement
- **F1: 50th place** (≈0) - Reached optimum but slow convergence

**Key Achievement:** Three podium finishes (1st, 2nd, 5th) validates high-risk/high-reward strategy.

### Method Performance Analysis

**GP-EI Success Rates:**
- Smooth functions (F1, F7): 75% success
- Multimodal functions (F2): 50% success until clustering Week 13
- Volatile functions (F4): 20% success, needs robust methods
- High-dimensional (F8): 60% success, needs trust regions

**Key Insight:** Simple GP-EI (75% success Weeks 4-8) significantly outperformed sophisticated manual strategies (37.5% success Week 12) when data was sparse.

### Breakthrough Results

#### Function 5: Exponential Growth Discovery

**Progression:**
```
Week 1:  [0.282, 0.850, 0.994, 0.935] → 2,266 (interior)
Week 8:  [0.212, 1.0, 1.0, 1.0]     → 4,450 (triple boundary +96%)
Week 11: [1.05, 1.05, 1.05, 1.05]   → 13,337 (extrapolation +200%)
Week 12: [1.5, 1.5, 1.5, 1.5]       → 263,397 (aggressive +1,875%)
Week 13: [2.0, 2.0, 2.0, 2.0]       → 2,686,080 (moonshot +920%)
```

**Strategy:** Questioned the assumed [0,1] constraint when three consecutive boundary tests showed exponential gains. Fitted exponential model: y = 54.06 × exp(5.68x) - 7,704, which predicted 4.6M at [2.0]. Achieved 2.69M (58% of prediction)—enough for 2nd place.

**Only beaten by:** Santosh Sougrakpam (1 person out of 65 competitors)

#### Function 2: Bimodal Structure Discovery

**Week 13 Clustering Analysis:**
- K-means (k=2) revealed two distinct peaks:
  - **Peak 1:** [0.50, 0.49] → 0.866 (best)
  - **Peak 2:** [0.71, 0.93] → 0.55-0.62 (suboptimal)
- Week 12 query [0.550, 0.481] landed in valley between peaks
- Returned to Peak 1 for Week 13, achieved 1st place

**Learning:** GP treats multimodal functions as noisy unimodal. Clustering reveals true structure invisible to GP alone.

### Failure Analysis

**Week 12 Catastrophe (37.5% success):**

Bundled four untested strategies simultaneously:
1. Ultra-fine precision search on F2 (±0.003 grid) → overfit to GP noise
2. Averaging method on F3 (mean of top 3 historical) → worse than Thompson
3. Feature freezing on F7 (lock 3/6 dims) → no improvement
4. Sensitivity nudges on F6 → catastrophic failure

**Critical Lesson:** Test one change per week with validation. Bundled changes make failure diagnosis impossible.

**F4 Outlier Problem:**

DBSCAN detected 42% of observations as outliers:
- Week 6: -16.65 catastrophic failure
- Week 2: -0.767 failure
- Only 4 of 13 observations near optimal 0.6 range

Standard GP assumes Gaussian noise—failed completely. Should have switched to robust GP or manual methods Week 7, took until Week 13.

### Learning Outcomes

**What Worked:**
1. GP-EI as reliable foundation (75% success on smooth functions)
2. Questioning assumptions (F5 extrapolation → 2nd place)
3. Clustering before optimization (F2 → 1st place)
4. Defensive final week (locked wins, one calculated risk)
5. Thompson Sampling as escape tool (F3 Week 10 recovery)

**What Didn't Work:**
1. Manual overrides without evidence (Week 12: 37.5%)
2. Bundled untested changes (impossible to diagnose failures)
3. Late discovery of dimension structure (Week 13, too late)
4. No trust regions on high-D functions (F8: 41st place)
5. Continued GP-EI on volatile functions (F4 outliers)

**What I'd Do Differently:**
1. Implement TuRBO trust regions Week 4+ (for F7, F8)
2. Run ARD analysis every week from Week 4 (not just Week 13)
3. Use DBSCAN outlier detection Week 6+ (flag F4 volatility early)
4. Latin Hypercube Sampling for initial 3 queries (better coverage)
5. Test one change per week with validation (avoid Week 12 bundling)

### Real-World Applications

**Hyperparameter Tuning:** For expensive models ($10K+ per training run), this GP-EI approach reduces evaluations by 3-5× vs random search while providing clustering diagnostics for multimodal parameter spaces.

**A/B Testing:** Phased rollout strategy (explore Weeks 1-3, exploit Weeks 4-8, defensive final week) directly applies to product development where each test takes weeks and affects real users.

**Neural Architecture Search:** GP-EI on architecture embeddings with clustering for motif discovery and Thompson Sampling for escaping ResNet-variants toward Transformers reduces training runs by 10× vs random NAS.

**Production Deployment:** Defensive Week 13 strategy (lock working models, shadow mode for candidates, one experimental 5% rollout) maps to enterprise ML where rollback is expensive.

## DOCUMENTATION

### Complete Project Documentation

📄 **[Function-Specific Datasheets](BBO_Function_Datasheets.md)** - Detailed analysis of all 8 functions including optimization decisions, weekly learning, and performance for each function (F1-F8).

📄 **[Model Card](BBO_Model_Card.md)** - Complete documentation of the GP-EI model including architecture, performance metrics, limitations, and trade-offs.

📄 **[Dataset Documentation](BBO_Datasheet.md)** - Comprehensive dataset description covering motivation, composition, collection process, and intended uses (if available).

---

## CONTACT DETAILS

**Project Repository:** https://github.com/yourusername/bbo-capstone

**Questions & Discussion:** Open an issue in the repository

**Collaboration:** Pull requests welcome for:
- Jupyter notebooks with analysis/visualizations
- TuRBO implementation for high-dimensional functions
- Additional benchmark comparisons

**Academic Context:** Imperial College Business School Executive Education - Machine Learning Module 24

---

## Repository Structure

```
bbo-capstone/
├── README.md (this file)
├── requirements.txt
├── LICENSE
├── .gitignore
│
├── data/
│   ├── complete_13week_data.json
│   └── complete_13week_results.csv
│
├── docs/
│   ├── BBO_Function_Datasheets.md (8 function-specific analyses)
│   ├── BBO_Model_Card.md (GP-EI model documentation)
│   ├── BBO_Datasheet.md (complete dataset documentation)
│   └── final_reflection.md (PCA/RL discussion, peer analysis)
│
├── scripts/
│   ├── bayesian_optimizer.py (core GP-EI implementation)
│   ├── extract_complete_data.py (.msg file parser)
│   └── analysis_scripts/ (clustering, length scales, etc.)
│
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_method_comparison.ipynb
│   ├── 03_f5_extrapolation_analysis.ipynb
│   └── 04_clustering_insights.ipynb
```

## Quick Start

```bash
# Clone repository
git clone https://github.com/yourusername/bbo-capstone.git
cd bbo-capstone

# Install dependencies
pip install -r requirements.txt

# Run example
python scripts/bayesian_optimizer.py

# Explore notebooks
jupyter notebook notebooks/
```

---

**License:** MIT  
**Version:** 1.0 (13-week competition completed)  
**Last Updated:** April 2026
