# Black-Box Bayesian Optimisation (BBO) Capstone

## Overview

This repository contains my Black-Box Bayesian Optimisation capstone work. The task is to optimise eight unknown black-box functions (2D-8D) where I cannot see the function equations. I only learn by submitting an input vector and receiving a score back from the course portal. The practical constraint is one query per function per week, so each query must be chosen carefully.

The workflow is built to be repeatable, traceable, and defensible. The notebooks handle the modelling logic. The folder structure and helper scripts keep the weekly process organised.

---

## Repository Structure

```
bayesian-optimization-capstone/
├── .gitignore
├── LICENSE
├── README.md
├── requirements.txt
├── notebooks/
│   ├── week00_random_initialization.ipynb
│   ├── week01-05_gp_ei_exploitation.ipynb
│   ├── week06-08_boundary_exploration.ipynb
│   ├── week09_catastrophic_failure.ipynb
│   └── week10_diagnostic_validation.ipynb
├── scripts/
│   ├── bayesian_optimizer.py
│   ├── diagnostic_validator.py
│   └── utils.py
├── docs/
│   ├── BBO_Datasheet.md
│   ├── BBO_Model_Card.md
│   └── Week10_Reflection.md
└── data/
    ├── week_00/
    ├── week_01/
    ├── ...
    ├── week_10/
    ├── cumulative/
    └── analysis/
```

---

## What the Notebooks Use

The notebooks work from the repository root and use these main folders:

* `data/week_XX/` for weekly portal submissions (inputs.txt) and returns (outputs.txt)
* `data/cumulative/` for combined historical data across all weeks
* `data/analysis/` for performance visualizations and progress tracking
* `scripts/` for core optimization and validation logic

Each week folder follows this convention:
* `data/week_XX/inputs.txt` - portal-ready query format
* `data/week_XX/outputs.txt` - returned function evaluations

---

## Weekly Workflow

1. Load cumulative data from previous weeks
2. Run appropriate weekly notebook (e.g., `week10_diagnostic_validation.ipynb`)
3. Generate next query using Gaussian Process surrogate
4. Format as portal-ready `inputs.txt`
5. Submit to portal and receive `outputs.txt`
6. Update cumulative dataset for next cycle

---

## Technical Approach

The workflow uses Gaussian Process surrogate models per function with acquisition-based candidate selection. In practical terms, the notebooks use:

* **scikit-learn** Gaussian Process regression
* **Matérn kernel** with adaptive length_scale tuning
* **Acquisition functions** such as Expected Improvement and Thompson Sampling
* **Validation framework** (Week 10) using sensitivity analysis to check model suggestions

This is a good fit for the capstone because the data is sparse and the evaluation budget is tight.

---

## Functions Overview

| Function | Dimensions | Characteristics |
|----------|-----------|-----------------|
| F1 | 2D | Minimization at maximum |
| F2 | 2D | Ultra-sharp peak |
| F3 | 3D | Prone to local optima |
| F4 | 4D | Highly volatile |
| F5 | 4D | Exponential growth, boundary-sensitive |
| F6 | 5D | Moderate complexity |
| F7 | 6D | Steady improvement pattern |
| F8 | 8D | High-dimensional sparse data |

---

## Installation & Usage

```bash
# Clone repository
git clone https://github.com/mesutszr/bayesian-optimization-capstone.git
cd bayesian-optimization-capstone

# Install dependencies
pip install -r requirements.txt

# Run optimization notebooks
jupyter notebook notebooks/

# Run diagnostic validation
python scripts/diagnostic_validator.py
```

---

## Project Documentation

Detailed methodology and dataset documentation:

* **[Datasheet for the BBO Dataset](BBO_Datasheet.md)** - Complete dataset documentation including data characteristics, sampling biases, collection process, and known limitations

* **[Model Card for BBO Optimization Approach](BBO_Model_Card.md)** - Methodology overview, intended use cases, performance evaluation, ethical considerations, and critical limitations


---
