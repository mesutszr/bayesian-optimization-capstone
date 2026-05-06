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

**Overall:** GP-EI as foundation with function-specific adaptations achieved competitive results across all eight functions.

## HYPERPARAMETER OPTIMISATION

The Gaussian Process model uses several key hyperparameters that define its optimization behavior. The kernel length scale (ℓ) controls how quickly the function varies and was initialized at 0.3 for all dimensions, then optimized via maximum likelihood estimation with 20 random restarts and bounds [0.01, 10.0] to prevent over/underfitting. The Matérn kernel smoothness parameter (ν=2.5) assumes twice-differentiable functions. The noise parameter (α=1e-6) assumes near-deterministic evaluations for most functions, though F4 required increased noise (α=1e-5) due to high volatility.

The Expected Improvement acquisition function uses exploration parameter ξ=0.01 for exploitation-biased search, and evaluates 1,000 random candidate points per iteration for computational efficiency. Method selection hyperparameters include dimension freezing threshold (length scale >5.0 indicates irrelevant dimensions), outlier detection threshold (>30% outliers triggers method switch from GP to manual approaches), and stagnation detection (3 consecutive weeks without improvement triggers switch to Thompson Sampling for exploration).

Hyperparameter choices were based on empirical week-over-week success rates rather than formal cross-validation, with strategy adapting from exploration-focused (Weeks 1-3) to exploitation-focused (Weeks 12-13) as optimization progressed. Missed opportunities include not implementing trust region methods for high-dimensional functions and not using length scale analysis earlier to identify freezable dimensions.

## RESULTS

### Competition Performance

**Individual Function Rankings:**
- 🥇 **F2: 1st place** (0.866) - K-means clustering revealed bimodal structure
- 🥈 **F5: 2nd place** (2,686,080) - Extrapolation beyond assumed [0,1] bounds
- 🏅 **F7: 5th place** (3.166) - Standard GP-EI worked perfectly
- **F4: 12th place** (0.6092) - Volatile function with 42% outliers
- **F6: 12th place** (-0.150) - Unstable, late failures
- **F3: 18th place** (-0.00571) - Early lucky discovery, never matched
- **F8: 41st place** (9.885) - 8D needed TuRBO, didn't implement
- **F1: 50th place** (≈0) - Reached optimum but slow convergence

### Learning Outcomes

The optimization journey revealed several key insights. Successful strategies included using GP-EI as a reliable foundation achieving 75% success on smooth functions, questioning assumed constraints which led to F5's 2nd place finish through extrapolation, applying clustering analysis before optimization which secured F2's 1st place by revealing bimodal structure, implementing defensive final-week strategies that locked validated wins while taking one calculated risk, and using Thompson Sampling as an escape mechanism for local optima.

Strategies that failed included manual overrides without empirical evidence (Week 12's 37.5% success rate), bundling multiple untested changes simultaneously which made failure diagnosis impossible, discovering dimensional structure too late (Week 13 ARD analysis should have been Week 4), not implementing trust regions for high-dimensional functions resulting in poor F8 performance, and continuing standard GP-EI on volatile functions despite clear outlier patterns. Future improvements would include implementing TuRBO trust regions from Week 4 for high-dimensional functions, running ARD analysis weekly to identify freezable dimensions early, using DBSCAN outlier detection from Week 6 to flag volatility, employing Latin Hypercube Sampling for initial queries to ensure better space coverage, and testing one methodological change per week with validation rather than bundling changes.

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

**Author:** Mesut Sezer  
Imperial College London ML/AI Capstone 2026  
**License:** MIT
