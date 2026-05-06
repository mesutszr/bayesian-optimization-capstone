# BBO Capstone: Function-Specific Datasheets

This document provides detailed optimization decisions, learning, and reasoning for all eight functions in the Black-Box Optimization capstone project.

---

# Function 1 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 1 (F1)

**2. What real-world scenario does this function simulate?**
A 2-dimensional optimization problem representing scenarios like material property optimization where two parameters must be simultaneously tuned to achieve an optimal outcome (e.g., temperature and pressure in a manufacturing process).

**3. What is the dimensionality of the input?**
2D (two input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
A scalar performance metric to maximize, with optimal value near zero (≈3.84×10⁻²⁶).

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (2D input, 1D output)
- Final dataset: 13 observations accumulated over 13 weeks

**2. How does the dataset evolve as you add new queries weekly?**
Dataset grew linearly (1 point per week). Early weeks (1-5) explored broad regions. Weeks 6-9 showed convergence toward optimal region. Weeks 10-13 locked at converged value with minimal variation.

**3. Does the function include noise or randomness?**
No significant noise detected. Repeated queries at identical coordinates (Weeks 10-13) produced identical outputs (≈3.84×10⁻²⁶), confirming deterministic behavior.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
Unimodal and smooth. GP predictions showed single convergence basin. No local optima encountered. Smooth gradient indicated by consistent improvement toward boundary region [0.54, 0.28].

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Primary: Gaussian Process with Expected Improvement (GP-EI)
- Kernel: Matérn (ν=2.5)
- Acquisition: Expected Improvement with ξ=0.01
- Candidate search: Random sampling (n=1,000)

**2. Why did you choose this method for this particular function?**
2D dimensionality suits GP well. Initial exploration suggested smooth landscape. GP-EI naturally balances exploration/exploitation without manual tuning required for low-dimensional smooth functions.

**3. How did you balance exploration and exploitation?**
GP-EI acquisition function automatically balanced:
- Weeks 1-3: High exploration (broad sampling)
- Weeks 4-9: Mixed (GP uncertainty decreased)
- Weeks 10-13: Pure exploitation (locked at best)

**4. Did your strategy change over the weeks? Why?**
Week 10: Switched from exploration to pure exploitation after reaching ≈0 (theoretical optimum). Weeks 10-13 submitted identical coordinates as defensive lock strategy.

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No rescaling. Inputs already in [0,1] range, well-suited for GP without normalization.

**2. Did you train any surrogate models?**
Yes, Gaussian Process surrogate fitted weekly on all historical observations.

**3. If yes, what preprocessing did the surrogate require?**
- Kernel: Matérn(ν=2.5) with length_scale=0.3 initial
- Output normalization: `normalize_y=True` (zero mean, unit variance)
- Noise parameter: `alpha=1e-6`
- Kernel optimization: 20 restarts

**4. Did you handle outliers or unusual data points?**
No outliers detected in F1. All 13 observations formed consistent convergence pattern toward optimal region.

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
- Weeks 1-3: Confirmed smooth, unimodal structure
- Weeks 4-9: Identified optimal region near [0.54, 0.28]
- Week 10: Reached theoretical optimum (≈0)
- Weeks 11-13: Confirmed convergence stability

**2. Did you encounter local optima? How did you detect them?**
No local optima detected. GP variance remained low across search space after Week 6, indicating confident single-basin structure.

**3. Which queried inputs were most informative and why?**
Week 10's query at [0.543405, 0.278369] → 3.84×10⁻²⁶ was most informative, revealing theoretical optimum location and confirming unimodal hypothesis.

**4. If you restarted, what would you do differently?**
- Use Latin Hypercube Sampling for initial 2-3 queries (better space coverage)
- Implement trust regions after Week 6 (focus search ±5% around best)
- More aggressive exploitation after Week 8 (reached near-optimal earlier than Week 10)

## Performance and Results

**1. What is the best output value you achieved?**
3.837488851105284×10⁻²⁶ (effectively zero, theoretical optimum)

**2. Which input vector produced this value?**
[0.543405, 0.278369]

**3. How confident are you that this is near the global maximum? Why?**
Very high confidence (95%+). Value is ≈0 (theoretical optimum for this function type). Repeated queries Weeks 10-13 produced identical results. GP variance extremely low across entire search space.

**4. Did your results align with expectations for this function?**
Yes. 2D smooth optimization expected to converge within 10-13 queries. Achieved theoretical optimum Week 10.

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Similar to industrial process optimization (temperature/pressure tuning), material science (composition optimization), or experimental design where only 2 key parameters matter.

**2. What limitations arise from the synthetic nature of the function?**
Real processes have noise, constraints, and time-varying behavior. F1's deterministic smoothness is ideal but unrealistic.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Yes for 2-5D smooth problems. GP-EI achieved optimum in 10 queries (very sample-efficient). For >10D or noisy problems, would need trust regions or robust GP methods.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- Don't over-explore after reaching ≈0 (wasted queries)
- 2D simplicity not representative of high-D complexity
- Deterministic behavior masks real-world noise handling needs

---

# Function 2 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 2 (F2)

**2. What real-world scenario does this function simulate?**
Bimodal optimization representing scenarios with multiple local optima, such as drug formulation with two viable compounds or materials with distinct stable configurations.

**3. What is the dimensionality of the input?**
2D (two input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
Performance score to maximize, with best achieved value of 0.866 (Week 12).

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (2D input, 1D output)
- Final dataset: 13 observations

**2. How does the dataset evolve as you add new queries weekly?**
Early exploration (Weeks 1-5) discovered high-value region. Week 6 found Peak 1 (0.829). Weeks 7-11 explored alternative regions discovering Peak 2. Week 12 achieved best (0.866). Week 13 anomaly: identical coordinates to Week 12 produced 0.613 (possible stochastic behavior).

**3. Does the function include noise or randomness?**
Likely yes. Week 12-13 anomaly: identical input [0.497, 0.486] produced different outputs (0.866 vs 0.613). Only instance across all functions showing this behavior.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
Bimodal with potential stochasticity. Week 13 k-means clustering (k=2) revealed:
- Peak 1: [0.50, 0.49] → 0.829-0.866
- Peak 2: [0.71, 0.93] → 0.55-0.62
- Valley between peaks caught some queries

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Weeks 1-12: GP-EI
- Week 13: K-means clustering analysis + return to validated peak

**2. Why did you choose this method for this particular function?**
GP-EI for smooth local regions. Week 13 clustering because GP missed multimodal structure—k-means revealed two distinct peaks that GP treated as single noisy region.

**3. How did you balance exploration and exploitation?**
- Weeks 1-6: Broad exploration found Peak 1
- Weeks 7-11: Continued exploration found Peak 2
- Week 12: Exploitation refined Peak 1
- Week 13: Diagnostic clustering, then defensive lock at Peak 1

**4. Did your strategy change over the weeks? Why?**
Major change Week 13: Added clustering analysis after noticing inconsistent GP predictions. K-means revealed bimodal structure invisible to GP alone. Returned to Peak 1 for final submission.

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No rescaling needed. [0,1] range appropriate for GP.

**2. Did you train any surrogate models?**
Yes, GP surrogate weekly. Week 13 added k-means clustering (k=2) for structure diagnosis.

**3. If yes, what preprocessing did the surrogate require?**
- GP: Matérn kernel, normalize_y=True, alpha=1e-6
- K-means: No preprocessing, fitted on historical (X, y) pairs

**4. Did you handle outliers or unusual data points?**
Week 13 clustering revealed no outliers but distinct clusters. Week 12-13 discrepancy flagged as potential stochastic noise or data anomaly.

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
- Weeks 1-6: Appeared unimodal smooth
- Weeks 7-11: Inconsistent results suggested complexity
- Week 13: K-means revealed true bimodal structure

**2. Did you encounter local optima? How did you detect them?**
Yes, two peaks. Detected through k-means clustering showing distinct high-value regions at [0.50, 0.49] and [0.71, 0.93].

**3. Which queried inputs were most informative and why?**
Week 12: [0.497, 0.486] → 0.866 most informative, achieving best value and confirming Peak 1 location. Week 6: [0.497, 0.486] → 0.829 initially discovered Peak 1.

**4. If you restarted, what would you do differently?**
- Run k-means clustering after Week 6 (8 observations sufficient)
- Explore both peaks systematically rather than GP random walk
- Test stochasticity hypothesis by repeating queries earlier

## Performance and Results

**1. What is the best output value you achieved?**
0.866 (Week 12)

**2. Which input vector produced this value?**
[0.497000, 0.486000]

**3. How confident are you that this is near the global maximum? Why?**
Moderately confident (70%). This is Peak 1 maximum. Peak 2 might have higher undiscovered regions. Week 13's decline (0.613 at identical coords) suggests either stochasticity or Peak 1 instability.

**4. Did your results align with expectations for this function?**
Partially. Expected smooth function, discovered bimodal. Week 12-13 stochastic behavior unexpected and concerning.

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Drug discovery with multiple viable compounds, materials with distinct stable phases, or investment portfolios with different risk-return profiles.

**2. What limitations arise from the synthetic nature of the function?**
Real multimodal problems often have >2 modes and higher noise. F2's clean bimodal structure is instructive but simplified.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Clustering-first approach scales well. K-means computational cost is low. For expensive evaluations, diagnostic clustering after 8-10 points prevents wasted queries exploring valleys.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- Don't trust GP alone on multimodal functions
- Cluster early (Week 6+) to reveal structure
- Stochastic behavior (Week 12-13) requires multiple validation queries
- Valley between peaks can trap GP-based search

## Competition Result

**Rank: 1st place out of 65 competitors** 🥇

---

# Function 3 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 3 (F3)

**2. What real-world scenario does this function simulate?**
3-dimensional optimization representing problems like chemical reaction optimization with three parameters (temperature, pressure, catalyst concentration) or sensor placement with three spatial coordinates.

**3. What is the dimensionality of the input?**
3D (three input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
Performance metric to maximize (minimize negative values), with best achieved value of -0.00571 (Week 2).

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (3D input, 1D output)
- Final dataset: 13 observations

**2. How does the dataset evolve as you add new queries weekly?**
Week 2 found best value (-0.00571), then performance declined consistently. Weeks 3-13 never matched Week 2. Multiple recovery attempts (Thompson Sampling, averaging) failed until Week 13 partial recovery to -0.0185.

**3. Does the function include noise or randomness?**
Minimal noise detected. Deterministic behavior observed with consistent decline pattern suggesting narrow optimum found Week 2 then lost.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
Unimodal with very narrow optimum. Week 13 ARD analysis revealed only Dim 0 matters (length_scale=0.019 vs Dims 1-2: 10.0). Function essentially 1D with sharp peak.

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Weeks 1-9: GP-EI
- Week 10, 13: Thompson Sampling (escape attempts)
- Weeks 11-12: Averaging method
- Week 13 final: Return to Week 2 coordinates

**2. Why did you choose this method for this particular function?**
GP-EI initially seemed appropriate for smooth 3D. After plateau Weeks 3-9, switched to Thompson Sampling (exploration emphasis). Week 13 analysis revealed 2/3 dimensions irrelevant—should have frozen them and searched 1D.

**3. How did you balance exploration and exploitation?**
- Weeks 1-2: Pure exploration (found best)
- Weeks 3-9: Exploitation attempts (failed to match)
- Week 10: Thompson exploration (improved to -0.097)
- Week 13: Return to historical best (recovery)

**4. Did your strategy change over the weeks? Why?**
Multiple changes due to persistent failure to match Week 2. Week 10 Thompson, Weeks 11-12 averaging, Week 13 recovery. Should have analyzed dimension sensitivity Week 4, frozen irrelevant dimensions.

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No rescaling. [0,1] range appropriate.

**2. Did you train any surrogate models?**
Yes, GP surrogate. Week 13 added ARD kernel analysis revealing dimension structure.

**3. If yes, what preprocessing did the surrogate require?**
- Standard GP: Matérn kernel, normalize_y=True
- Week 13 ARD: Different length scale per dimension to identify sensitivity

**4. Did you handle outliers or unusual data points?**
No outliers. All observations showed consistent pattern: Week 2 best, rest worse.

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
- Week 2: Found -0.00571, appeared to discover optimum
- Weeks 3-9: Couldn't replicate, suggested narrow peak
- Week 13: ARD revealed 2/3 dims irrelevant—true structure is 1D sharp peak

**2. Did you encounter local optima? How did you detect them?**
Weeks 3-12 likely stuck in local optima (-0.097 to -0.121 range). Week 2's -0.00571 was likely global optimum, never relocated.

**3. Which queried inputs were most informative and why?**
Week 2: [0.269822, 0.373539, 0.449399] → -0.00571 most informative—discovered global optimum but couldn't explain why. Week 13 ARD analysis showed only first coordinate mattered.

**4. If you restarted, what would you do differently?**
- Run ARD analysis Week 4 (after 3-4 observations)
- Freeze Dims 1-2 at Week 2 values
- Search only Dim 0 in tight range around 0.270
- Use finer grid search on single dimension instead of 3D GP

## Performance and Results

**1. What is the best output value you achieved?**
-0.005714 (Week 2)

**2. Which input vector produced this value?**
[0.269822, 0.373539, 0.449399]

**3. How confident are you that this is near the global maximum? Why?**
High confidence (85%). Week 2 result 30× better than next-best. ARD analysis confirms narrow peak in Dim 0. Likely within 0.01 of global optimum.

**4. Did your results align with expectations for this function?**
No. Expected 3D smooth function. Discovered effective 1D function with 2/3 dimensions irrelevant and extremely narrow optimum. Misalignment caused 11 weeks of failed recovery attempts.

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Many engineering problems have irrelevant parameters. Chemical reactions where some variables don't affect yield, or sensor networks where some positions don't matter for coverage.

**2. What limitations arise from the synthetic nature of the function?**
Real problems rarely have 2/3 dimensions perfectly irrelevant. Synthetic clarity helpful for learning but unrealistic.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Critical lesson: Always run sensitivity analysis early. For expensive evaluations, identifying irrelevant dimensions saves enormous resources. ARD after 3-4 evaluations prevents wasting 8+ queries on irrelevant dimensions.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- Don't assume all stated dimensions matter
- Run ARD/sensitivity analysis early (after 3-5 observations)
- Narrow optima require fine resolution once located
- Lucky early discovery can be impossible to replicate without understanding structure

## Competition Result

**Rank: 18th out of 65 competitors**

---

# Function 4 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 4 (F4)

**2. What real-world scenario does this function simulate?**
4-dimensional volatile optimization representing high-noise scenarios like biological systems, market predictions, or experimental processes with measurement error and inherent variability.

**3. What is the dimensionality of the input?**
4D (four input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
Performance score to maximize, with extreme volatility: best 0.6092, worst -16.65.

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (4D input, 1D output)
- Final dataset: 13 observations with extreme variance

**2. How does the dataset evolve as you add new queries weekly?**
Extreme volatility throughout. Week 2: -0.767 (bad). Weeks 3-5: improved to 0.4-0.5 range. Week 6: catastrophic -16.65. Week 7: recovery to 0.609 (best). Weeks 8-13: oscillated, final Week 13 perfect recovery to 0.6092.

**3. Does the function include noise or randomness?**
Extreme noise/outliers. Week 13 DBSCAN analysis: 42% of observations flagged as outliers. Includes the -16.65 catastrophe and several -0.5 to -1.0 failures.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
Extremely noisy with outlier dominance. Not smooth—DBSCAN shows only 58% of points form coherent cluster. GP assumptions violated by outlier structure.

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Weeks 1-12: GP-EI (failed repeatedly)
- Week 13: Manual return to Week 7 best coordinates

**2. Why did you choose this method for this particular function?**
Initially chose GP-EI by default. Should have switched to manual/robust methods after Week 6 catastrophe. Week 13 recognized GP fundamentally unsuitable for 42% outlier rate.

**3. How did you balance exploration and exploitation?**
Poorly. GP-EI assumes smooth landscape, kept exploring despite evidence of extreme noise. Should have exploited Week 7 best more conservatively.

**4. Did your strategy change over the weeks? Why?**
Minimal change until Week 13. Persisted with GP-EI too long. Mariana (peer) banned EI on F4 after Weeks 6-7 failures. I didn't ban until Week 13 (6-week delay).

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No rescaling. [0,1] range standard.

**2. Did you train any surrogate models?**
Yes, GP surrogate through Week 12. Week 13 abandoned GP for outlier-aware analysis (DBSCAN).

**3. If yes, what preprocessing did the surrogate require?**
Standard GP with increased noise parameter (alpha=1e-5 vs standard 1e-6) attempting to handle volatility. Insufficient for 42% outlier rate.

**4. Did you handle outliers or unusual data points?**
No until Week 13. DBSCAN (eps=0.3, min_samples=3) flagged 42% outliers including -16.65, -0.767, and several -0.5 range failures. Should have used robust GP or outlier rejection after Week 6.

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
- Weeks 1-5: Appeared noisy but manageable
- Week 6: -16.65 catastrophe revealed extreme volatility
- Week 7: 0.609 showed good region exists
- Week 13: DBSCAN revealed only 4 points near 0.6 form cluster, rest outliers

**2. Did you encounter local optima? How did you detect them?**
Can't distinguish local optima from noise. Volatility dominates. Week 7's 0.609 might be local or global—impossible to determine with 42% outlier rate.

**3. Which queried inputs were most informative and why?**
Week 7: [0.431522, 0.423071, 0.365186, 0.419712] → 0.6092 most informative—only reliably good region identified. Week 6: -16.65 informative negatively—revealed catastrophic failure mode exists.

**4. If you restarted, what would you do differently?**
- After Week 6 catastrophe, switch to manual methods immediately
- Use robust GP with outlier rejection
- Never use standard GP-EI on functions with >20% outlier rate
- Replicate Week 7 best 3-5 times to confirm stability

## Performance and Results

**1. What is the best output value you achieved?**
0.6092 (Week 7, matched Week 13)

**2. Which input vector produced this value?**
[0.431522, 0.423071, 0.365186, 0.419712]

**3. How confident are you that this is near the global maximum? Why?**
Low confidence (40%). Only 4 of 13 queries achieved >0.5. Don't know if better regions exist or if 0.6 is noise ceiling. 42% outlier rate prevents reliable landscape modeling.

**4. Did your results align with expectations for this function?**
No. Expected 4D optimization, discovered noise-dominated system where GP fundamentally fails. Expectations completely violated.

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Biological experiments with inherent variability, financial markets with rare catastrophic events, or physical systems with measurement errors and equipment failures.

**2. What limitations arise from the synthetic nature of the function?**
Real noisy systems might have identifiable noise sources to model or avoid. F4's synthetic volatility is educational but doesn't teach noise source diagnosis.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Current strategy (GP-EI) absolutely would NOT scale. For expensive noisy evaluations, must use robust methods: replicated observations, outlier-aware GP, or Bayesian methods with heavy-tailed noise models. Single expensive evaluation on F4-like system could waste $100K on catastrophic outlier.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- Standard GP fails on >30% outlier rates
- Catastrophic failures can occur (Week 6: -16.65)
- Must implement outlier detection early (DBSCAN after 5-6 points)
- Consider robust alternatives: replicated queries, median regression, or manual conservative approach

## Competition Result

**Rank: 12th out of 65 competitors**

---

# Function 5 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 5 (F5)

**2. What real-world scenario does this function simulate?**
4-dimensional exponential growth optimization representing scenarios where parameters interact multiplicatively, such as compound returns in finance, viral spread with multiple factors, or chemical reactions with catalytic amplification.

**3. What is the dimensionality of the input?**
4D (four input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
Performance score to maximize, achieving final value of 2,686,080 (Week 13) through boundary extrapolation.

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (4D input, 1D output)
- Final dataset: 13 observations showing exponential growth pattern

**2. How does the dataset evolve as you add new queries weekly?**
Exponential progression through boundary pushing:
- Weeks 1-7: Interior/boundary exploration (2,266 → 4,450)
- Week 8: Triple boundary [0.212, 1.0, 1.0, 1.0] → 4,450 (+96%)
- Weeks 11-12: Extrapolation beyond [0,1] → 13,337 → 263,397
- Week 13: Aggressive [2.0] → 2,686,080 (+920%)

**3. Does the function include noise or randomness?**
No noise detected. Exponential pattern highly consistent. Repeated boundary tests showed reliable multiplicative gains.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
Monotonically increasing toward boundaries with exponential growth. Week 13 ARD analysis revealed only Dim 3 varies meaningfully (length_scale=0.31), Dims 0-2 frozen (length_scale 5-10). Effectively 1-2D exponential function in 4D space.

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Weeks 1-7: GP-EI standard
- Week 8-10: Boundary pushing within [0,1]
- Weeks 11-13: Extrapolation beyond assumed [0,1] constraint

**2. Why did you choose this method for this particular function?**
Week 8's triple boundary success suggested testing beyond [0,1]. Three consecutive boundary queries (Weeks 8, 11, 12) showed multiplicative not additive gains, validating exponential hypothesis. Week 13 extrapolation was calculated risk based on strong pattern evidence.

**3. How did you balance exploration and exploitation?**
- Weeks 1-10: Standard GP-EI balance
- Weeks 11-12: Structured exploration beyond assumed bounds
- Week 13: Pure exploitation of validated exponential pattern

**4. Did your strategy change over the weeks? Why?**
Major strategic shift Week 11: Questioned [0,1] constraint assumption. When Week 11's [1.05] achieved 200% gain and Week 12's [1.5] achieved 1,875% gain, exponential pattern confirmed. Week 13 went aggressive [2.0] based on validated trend.

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No initial rescaling. After Week 11, allowed extrapolation beyond [0,1], effectively removing upper bound constraint when data supported it.

**2. Did you train any surrogate models?**
Yes, GP surrogate weekly. Week 13 added exponential curve fitting to boundary progression data: y = 54.06 × exp(5.68x) - 7,704.

**3. If yes, what preprocessing did the surrogate require?**
- GP: Standard Matérn kernel through Week 10
- Exponential fit: Used (max_coord, output) pairs from Weeks 11-13
- ARD analysis Week 13: Revealed 3/4 dimensions effectively frozen

**4. Did you handle outliers or unusual data points?**
No outliers. All observations formed consistent exponential growth pattern toward boundaries.

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
- Weeks 1-7: Interior sampling showed increasing trend toward boundaries
- Week 8: Triple boundary [0.212, 1.0, 1.0, 1.0] revealed acceleration at edges
- Week 11: [1.05] success (+200%) suggested [0,1] constraint might be soft
- Week 12: [1.5] success (+1,875%) confirmed exponential extrapolation valid
- Week 13: [2.0] achieved 2.69M, validating aggressive strategy

**2. Did you encounter local optima? How did you detect them?**
No local optima. Function monotonically increasing. GP variance remained high in unexplored boundary regions, indicating no convergence to local traps.

**3. Which queried inputs were most informative and why?**
- Week 8: [0.212, 1.0, 1.0, 1.0] → 4,450 (revealed boundary sensitivity)
- Week 11: [1.05, 1.05, 1.05, 1.05] → 13,337 (proved extrapolation works)
- Week 13: [2.0, 2.0, 2.0, 2.0] → 2,686,080 (confirmed exponential model, secured 2nd place)

**4. If you restarted, what would you do differently?**
- Test extrapolation earlier (Week 9-10 instead of 11)
- Run ARD analysis Week 6 to discover 3/4 frozen dimensions
- Focus single-dimension search along primary direction
- Test [2.5] or [3.0] Week 13 to see if exponential continues

## Performance and Results

**1. What is the best output value you achieved?**
2,686,080 (Week 13)

**2. Which input vector produced this value?**
[2.0, 2.0, 2.0, 2.0]

**3. How confident are you that this is near the global maximum? Why?**
Moderate confidence (60%). Exponential model predicted 4.6M at [2.0], achieved 2.69M (58% of prediction). Pattern suggests [2.5-3.0] might yield higher, but risk of boundary violation or function undefined region. Confident 2.69M is in top 5% of possible values.

**4. Did your results align with expectations for this function?**
Exceeded expectations. Assumed [0,1] constraint, discovered exponential growth beyond it. Question-assumption approach led to breakthrough result—2nd place out of 65 competitors.

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Financial portfolio optimization where returns compound, viral marketing where effects multiply, drug dosing where synergistic effects exist, or manufacturing where parameters interact multiplicatively.

**2. What limitations arise from the synthetic nature of the function?**
Real exponential processes have physical limits and saturation. F5's continued exponential beyond [2.0] is mathematically interesting but physically unrealistic. Real applications need domain knowledge about when constraints truly bind.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Yes, with modifications. Pattern-recognition approach (3 consecutive boundary improvements → test extrapolation) is generalizable. For expensive evaluations, would add: (1) smaller extrapolation steps (1.1 instead of 1.5), (2) validation queries, (3) domain expert consultation on constraint legitimacy.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- Don't blindly accept stated constraints—test boundaries when data suggests
- Exponential models can extrapolate poorly—validate predictions
- Need 3+ consistent observations before aggressive extrapolation
- Real systems may have discontinuities or undefined regions beyond bounds

## Competition Result

**Rank: 2nd place out of 65 competitors** 🥈
**Achievement: Only beaten by 1 person (Santosh Sougrakpam)**

---

# Function 6 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 6 (F6)

**2. What real-world scenario does this function simulate?**
5-dimensional optimization representing moderate-complexity scenarios like process optimization with five parameters (temperature, pressure, flow rate, composition, catalyst amount) or system tuning with multiple interdependent settings.

**3. What is the dimensionality of the input?**
5D (five input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
Performance score to maximize, with best achieved value of -0.150 (Week 10), showing unstable behavior with late catastrophic failures.

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (5D input, 1D output)
- Final dataset: 13 observations showing instability

**2. How does the dataset evolve as you add new queries weekly?**
Improving trend Weeks 1-10, achieving -0.150 (best). Week 11-12 maintained reasonable values. Week 12-13 sudden catastrophic decline to -2.2 range despite GP-EI guidance, indicating GP model breakdown.

**3. Does the function include noise or randomness?**
Moderate instability. Function appeared stable Weeks 1-11, then unpredictably collapsed Weeks 12-13. Pattern doesn't match noise (random fluctuation) but rather state-dependent behavior GP missed.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
Uncertain. Week 1-10 suggested smooth unimodal with gradual improvement. Week 12-13 catastrophic failures suggest either: (1) GP explored bad region, (2) multimodal with discovered trap, or (3) constraint violations GP didn't model.

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Weeks 1-13: GP-EI consistently
- Week 13: Attempted recovery to Week 10 best coordinates

**2. Why did you choose this method for this particular function?**
5D dimensionality suited GP-EI. Early success (Weeks 1-10) validated approach. Persisted despite Week 12-13 failures hoping for recovery.

**3. How did you balance exploration and exploitation?**
- Weeks 1-10: GP-EI natural balance, trending toward exploitation
- Weeks 11-13: Attempted exploitation near Week 10 best, but GP guidance failed

**4. Did your strategy change over the weeks? Why?**
Minimal strategic change. Should have: (1) implemented trust regions after Week 10, (2) frozen at Week 10 best Weeks 11-12, (3) used manual override Week 13 instead of trusting GP.

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No rescaling. [0,1] range standard.

**2. Did you train any surrogate models?**
Yes, GP surrogate weekly. Model appeared reliable Weeks 1-10, then failed Weeks 12-13 suggesting inadequate for this function's structure.

**3. If yes, what preprocessing did the surrogate require?**
Standard GP: Matérn kernel, normalize_y=True, alpha=1e-6, 20 restarts.

**4. Did you handle outliers or unusual data points?**
No outlier handling. Week 12-13 catastrophic values (-2.2 range) flagged post-hoc but not detected real-time by GP confidence intervals (which remained moderate).

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
- Weeks 1-10: Built confidence in GP model, smooth improvement to -0.150
- Week 12-13: Shattered confidence—GP-guided queries produced catastrophic failures
- Revealed GP unreliable in 5D sparse-data regime even after 10-12 observations

**2. Did you encounter local optima? How did you detect them?**
Week 10's -0.150 might be local optimum or edge of good region. Week 12-13 failures suggest stepped outside viable basin, but GP didn't detect boundary.

**3. Which queried inputs were most informative and why?**
Week 10: [0.799, 0.873, 0.510, 0.443, 0.954] → -0.150 most informative—identified best region achieved. Week 12-13: catastrophic failures informative negatively—revealed GP model inadequacy.

**4. If you restarted, what would you do differently?**
- Implement trust regions after Week 10 (±5% radius)
- Lock at Week 10 best for Weeks 11-13 (defensive)
- Use ensemble methods (multiple GPs) to detect uncertainty
- Add clustering to identify if Week 12-13 explored distinct bad region

## Performance and Results

**1. What is the best output value you achieved?**
-0.150 (Week 10)

**2. Which input vector produced this value?**
[0.799, 0.873, 0.510, 0.443, 0.954]

**3. How confident are you that this is near the global maximum? Why?**
Low confidence (30%). Only held value for one week before decline. Don't know if better regions exist or if -0.150 is near optimum. Week 12-13 failures suggest landscape more complex than GP modeled.

**4. Did your results align with expectations for this function?**
No. Expected 5D smooth optimization achievable with GP-EI. Instead discovered unstable function where GP fails even after 12 observations. Misalignment caused rank decline in final weeks.

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Industrial processes where parameter interactions create unstable regions, financial systems with sudden regime changes, or biological systems with threshold effects.

**2. What limitations arise from the synthetic nature of the function?**
Real unstable systems might provide warning signs (gradual deterioration). F6's sudden failure pattern is educational but might not match real failure modes.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Current strategy (pure GP-EI) would NOT scale safely. For expensive evaluations on unstable functions, must implement: (1) trust regions to limit damage, (2) ensemble uncertainty, (3) conservative locking after success, (4) manual override capability.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- 5D+ functions can fool GP even after 12 observations
- Success Weeks 1-10 doesn't guarantee continued success
- Implement defensive locks after achieving good results
- Don't blindly trust GP in moderate-high dimensions with sparse data

## Competition Result

**Rank: 12th out of 65 competitors**

---

# Function 7 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 7 (F7)

**2. What real-world scenario does this function simulate?**
6-dimensional optimization representing complex scenarios like multi-parameter system tuning (6 interdependent settings), supply chain optimization with six decision variables, or experimental design with six factors.

**3. What is the dimensionality of the input?**
6D (six input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
Performance score to maximize, achieving final value of 3.166 (Weeks 12-13), representing consistent improvement through standard methods.

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (6D input, 1D output)
- Final dataset: 13 observations showing steady improvement

**2. How does the dataset evolve as you add new queries weekly?**
Consistent upward trend throughout: Week 1: 1.58 → Week 5: 2.00 → Week 11: 2.29 → Week 12: 3.17. Most reliable improvement pattern across all functions. Locked at 3.166 for Weeks 12-13.

**3. Does the function include noise or randomness?**
Minimal noise. Weeks 12-13 identical queries produced identical outputs (3.166), confirming deterministic behavior.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
Smooth unimodal with consistent gradients. GP predictions reliable throughout. No local optima encountered. 6D complexity handled well by GP-EI without special techniques.

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Weeks 1-13: GP-EI consistently
- No method changes—worked reliably throughout

**2. Why did you choose this method for this particular function?**
6D dimensionality at upper range of GP-EI effectiveness but manageable. Function's smooth behavior validated choice. Consistent 75% success rate meant no changes needed.

**3. How did you balance exploration and exploitation?**
GP-EI automatic balance worked perfectly:
- Weeks 1-3: Broad 6D exploration
- Weeks 4-10: Mixed, gradual improvement
- Weeks 11-13: Exploitation near optimum

**4. Did your strategy change over the weeks? Why?**
No changes. Only function where Week 1 strategy persisted through Week 13 successfully. Locked Weeks 12-13 as defensive measure, not due to method failure.

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No rescaling. [0,1] range appropriate.

**2. Did you train any surrogate models?**
Yes, GP surrogate weekly. Model remained reliable throughout—rare success case for 6D sparse data.

**3. If yes, what preprocessing did the surrogate require?**
Standard GP: Matérn kernel, normalize_y=True, alpha=1e-6, 20 restarts. No special techniques needed.

**4. Did you handle outliers or unusual data points?**
No outliers detected. All 13 observations formed consistent upward trend.

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
Each week refined understanding of smooth upward gradient. By Week 11, GP had high confidence in local region around [0.207, 0.216, 0.376, 0.302, 0.307, 0.638]. Week 12 achieved best, Week 13 confirmed stability.

**2. Did you encounter local optima? How did you detect them?**
No local optima detected. GP variance decreased smoothly throughout, no plateaus indicating traps. Monotonic improvement suggests unimodal or encountered global basin early.

**3. Which queried inputs were most informative and why?**
Week 12: [0.207360, 0.215765, 0.375946, 0.301781, 0.306902, 0.637566] → 3.166 most informative—discovered best value and provided stable lock point for Week 13.

**4. If you restarted, what would you do differently?**
- Implement trust regions after Week 8 (focus ±5% around best)
- Use Latin Hypercube Sampling for initial 3 queries (better 6D coverage)
- Consider TuRBO for potential further gains beyond 3.166
- Otherwise, strategy was optimal—don't fix what isn't broken

## Performance and Results

**1. What is the best output value you achieved?**
3.166 (Weeks 12-13)

**2. Which input vector produced this value?**
[0.207360, 0.215765, 0.375946, 0.301781, 0.306902, 0.637566]

**3. How confident are you that this is near the global maximum? Why?**
Moderately high confidence (75%). Consistent improvement over 13 weeks with no plateaus suggests not trapped in local optimum. GP variance low in explored region. However, 6D space large enough that unexplored regions might contain better values.

**4. Did your results align with expectations for this function?**
Yes, perfectly. Expected 6D smooth optimization achievable with standard GP-EI. Achieved consistent improvement reaching competitive value (5th place of 65).

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Multi-parameter system tuning: database configuration, chemical process control, supply chain optimization, or machine learning hyperparameter tuning where 5-10 parameters interact.

**2. What limitations arise from the synthetic nature of the function?**
Real 6D problems often have constraints, discrete variables, or categorical choices. F7's smooth continuous nature is idealized.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Yes, with trust regions. GP-EI achieved good results in 13 queries. For expensive evaluations (days, $10K+ per query), would add: (1) LHS initial sampling, (2) trust regions after 8-10 queries, (3) potentially TuRBO for >6D problems.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- 6D is near upper limit for standard GP-EI with sparse data
- Success here doesn't guarantee success on >8D problems
- Trust regions would improve efficiency (could have achieved 3.2-3.3 with focused search)
- F7's smoothness is best-case; real functions more complex

## Competition Result

**Rank: 5th place out of 65 competitors** 🏅

---

# Function 8 Datasheet

## Function Overview

**1. Which function does this datasheet describe?**
Function 8 (F8)

**2. What real-world scenario does this function simulate?**
8-dimensional optimization representing high-complexity scenarios like deep learning architecture search (8+ hyperparameters), manufacturing with many process parameters, or portfolio optimization with multiple asset classes.

**3. What is the dimensionality of the input?**
8D (eight input parameters)

**4. How many initial data points were provided?**
0 initial points. All data generated through 13 weekly queries.

**5. What does the output represent?**
Performance score to maximize, with best achieved value of 9.885 (Week 11), showing moderate success followed by slight decline.

## Nature of the Data

**1. Describe the structure of the initial dataset.**
- Week 1: 1 observation (8D input, 1D output)
- Final dataset: 13 observations in high-dimensional space

**2. How does the dataset evolve as you add new queries weekly?**
Variable progress: Early weeks 3-7 range. Week 11 breakthrough to 9.885. Week 13 slight decline to 9.288. Less consistent than lower-dimensional functions, suggesting 8D complexity challenges standard GP-EI.

**3. Does the function include noise or randomness?**
Low noise. Week 13 coordinates nearly identical to Week 11, produced similar outputs (9.288 vs 9.885), consistent with deterministic function plus small GP exploration differences.

**4. Based on observations, does the function appear unimodal, multimodal, noisy, or smooth?**
High-dimensional complexity. Week 13 ARD analysis revealed 6 of 8 dimensions have critical length scales (<0.03), indicating true 8D problem where most dimensions matter. This explains GP-EI difficulty—no dimensional reduction possible.

## Your Optimization Strategy

**1. Which optimization method(s) did you use?**
- Weeks 1-13: GP-EI consistently
- No special high-dimensional techniques (TuRBO, dimension freezing)

**2. Why did you choose this method for this particular function?**
Default to GP-EI. Should have recognized 8D requires specialized methods. Peers (Vinny) used trust regions and dimension freezing—I did not.

**3. How did you balance exploration and exploitation?**
GP-EI automatic balance. However, 8D space is 2⁸=256× larger than 1D, so 13 queries provide extremely sparse coverage. Likely over-explored relative to data density.

**4. Did your strategy change over the weeks? Why?**
No strategic changes. Major missed opportunity: should have implemented TuRBO or trust regions after Week 8. Peers who used these techniques likely achieved better F8 performance.

## Data Handling and Preprocessing

**1. Did you rescale or normalize inputs? Why or why not?**
No rescaling. [0,1] range standard.

**2. Did you train any surrogate models?**
Yes, GP surrogate weekly. Likely underfit in 8D—need 2^D samples for reasonable coverage, had only 13.

**3. If yes, what preprocessing did the surrogate require?**
Standard GP: Matérn kernel, normalize_y=True. Should have used: (1) ARD kernel from start to identify key dimensions, (2) Trust regions to focus search, (3) Higher sample efficiency methods like TuRBO.

**4. Did you handle outliers or unusual data points?**
No outliers detected. All observations reasonable values (3-10 range).

## Weekly Iteration and Learning

**1. How did new data points change your understanding of the function landscape?**
Slow learning due to curse of dimensionality. Each query explores tiny fraction of 8D space. Week 11 found best value but surrounding landscape poorly understood. Week 13 ARD revealed 6/8 dimensions critical—can't reduce dimensionality like F3 or F5.

**2. Did you encounter local optima? How did you detect them?**
Unknown. 8D space too large to distinguish local vs global optimum with 13 samples. Week 11's 9.885 might be local optimum, global optimum, or far from either.

**3. Which queried inputs were most informative and why?**
Week 11: [0.0, 0.0, 0.0, 0.0, 0.778, 0.688, 0.722, 1.0] → 9.885 most informative—best value achieved. Interesting pattern: first 4 dims at boundaries (0 or 1), last 4 varied—suggests possible dimensional structure.

**4. If you restarted, what would you do differently?**
- Use TuRBO (Trust Region Bayesian Optimization) from Week 4
- Implement Latin Hypercube Sampling for initial 5-8 queries
- Run ARD analysis Week 5, identify if any dimensions freezable (Week 13 ARD showed 6/8 critical, but worth checking early)
- Use ensemble GPs to better quantify uncertainty in 8D

## Performance and Results

**1. What is the best output value you achieved?**
9.885 (Week 11)

**2. Which input vector produced this value?**
[0.0, 0.0, 0.0, 0.0, 0.778, 0.688, 0.722, 1.0]

**3. How confident are you that this is near the global maximum? Why?**
Low confidence (30%). 8D space enormous, 13 samples provides <0.001% coverage. GP variance remains high across most of space. Likely local optimum or edge of good region, not global maximum.

**4. Did your results align with expectations for this function?**
No. Expected 8D optimization achievable with GP-EI, but underestimated curse of dimensionality. 13 queries insufficient for 8D. Competitors with trust regions likely performed better.

## Ethical, Practical and General Considerations

**1. How does this black-box optimisation task relate to real-world applications?**
Deep learning architecture search (8-12 hyperparameters), complex system tuning (database with many knobs), portfolio optimization (8+ assets), or manufacturing with many process variables.

**2. What limitations arise from the synthetic nature of the function?**
Real 8D problems often have structure: irrelevant dimensions, separable objectives, or categorical variables. F8's 6/8 critical dimensions is worst case—no dimensional reduction possible.

**3. Would your strategy scale to more serious or more expensive problems? Why or why not?**
Absolutely NOT. Current strategy (standard GP-EI) inefficient in 8D, would be catastrophic in 10D+. For expensive high-D evaluations must use: (1) TuRBO trust regions, (2) Random embeddings (REMBO), (3) Active subspaces, (4) Sensitivity analysis for dimension reduction.

**4. What risks or pitfalls should a future user be aware of when analysing this function?**
- Standard GP-EI fails beyond 6-7 dimensions without specialization
- 13 queries provide almost no 8D coverage
- Must use trust regions or dimension reduction
- Peer strategies (TuRBO, dimension freezing) crucial for >6D
- My 41st place rank shows cost of not adapting to dimensionality

## Competition Result

**Rank: 41st out of 65 competitors**
**Key lesson: High-dimensional optimization requires specialized methods, not just standard GP-EI**

---

# Summary of 8-Function Performance

| Function | Dimensions | Best Result | Rank | Method | Key Learning |
|----------|-----------|-------------|------|--------|--------------|
| F1 | 2D | ≈0 (optimal) | 50/65 | GP-EI | Convergence speed ≠ precision |
| **F2** | 2D | 0.866 | **1/65** 🥇 | Clustering | K-means reveals multimodal |
| F3 | 3D | -0.00571 | 18/65 | GP-EI | 2/3 dims irrelevant (late discovery) |
| F4 | 4D | 0.6092 | 12/65 | Manual | 42% outliers break GP |
| **F5** | 4D | 2,686,080 | **2/65** 🥈 | Extrapolation | Question assumptions |
| F6 | 5D | -0.150 | 12/65 | GP-EI | Late catastrophic failure |
| **F7** | 6D | 3.166 | **5/65** 🏅 | GP-EI | Standard method worked |
| F8 | 8D | 9.885 | 41/65 | GP-EI | Needed TuRBO/trust regions |

**Overall: 9th out of 65 competitors (Top 10!)**

**Three podium finishes (1st, 2nd, 5th) validate high-risk/high-reward strategy while highlighting missed opportunities in dimension freezing and trust region methods.**
