# Assignment 2: Differential Privacy  
## Student Study Hours Dataset Analysis

This assignment demonstrates **differential privacy** in practice by applying Laplace noise to database queries over a synthetic student dataset. We explore the fundamental privacy-accuracy tradeoff by computing statistics under different privacy budgets (epsilon values).

---

## Dataset: Student Study Hours

### Overview
A synthetic dataset of **200 students** with their weekly study hours. The dataset was generated from a normal distribution and includes no personally identifying information (privacy by design).

### Bounds

| Parameter | Value | Meaning |
|-----------|-------|---------|
| **N_STUDENTS** | 200 | Total number of records in the dataset |
| **MIN_HOURS** | 0 | Minimum possible study hours (lower bound) |
| **MAX_HOURS** | 60 | Maximum possible study hours (upper bound) |
| **THRESHOLD** | 20 hours | Used to count students exceeding this threshold |

**Why bounds matter:** Differential privacy requires a bounded data domain. The Laplace noise injection is mathematically derived from the *sensitivity* of the query, which depends on these bounds:
- **Count sensitivity:** 1 (adding/removing one student changes count by at most 1)
- **Mean sensitivity:** $(MAX\_HOURS - MIN\_HOURS) / N = (60 - 0) / 200 = 0.30$ (one person's maximum effect on the mean)

---

## Epsilon (ε): The Privacy Budget

**Epsilon** is a single number that quantifies the privacy-accuracy tradeoff in differential privacy.

### Interpretation

| Epsilon | Privacy Strength | Accuracy | Noise Level | Use Case |
|---------|------------------|----------|-------------|----------|
| **0.1** | Very High | Low | Very Large | Highly sensitive data (medical records) |
| **0.5** | High | Moderate | Large | Sensitive surveys |
| **1.0** | Medium | Good | Medium | Balanced applications |
| **2.0** | Weak | Very Good | Small | Less sensitive aggregates |

### Chosen Epsilons: [0.1, 0.5, 1.0, 2.0]

We tested **4 epsilon values** to observe the privacy-accuracy spectrum:
- **ε = 0.1:** Maximum privacy protection—almost no information leakage about any individual student
- **ε = 0.5:** Strong privacy with acceptable accuracy loss
- **ε = 1.0:** Balanced tradeoff (common choice in practice)
- **ε = 2.0:** Weak privacy, high accuracy (minimal noise)

**Key insight:** As ε increases → noise decreases → accuracy improves → privacy weakens

---

## Results: Privacy vs. Accuracy Tradeoff

### Experimental Setup
- **50 independent runs** were conducted for each epsilon value
- For each run: Laplace noise was independently sampled and added to the COUNT and MEAN statistics
- **Both statistics were computed:** Count of students studying > 20 hours, and mean study hours

### Summary Statistics

| ε | Avg Error (Count) | Std Dev (Count) | Avg Error (Mean) | Std Dev (Mean) |
|---|---|---|---|---|
| **0.1** | 8.7522 | 8.7107 | 3.1569 | 2.8070 |
| **0.5** | 1.9355 | 1.8734 | 0.6841 | 0.8044 |
| **1.0** | 0.9462 | 0.9681 | 0.2820 | 0.2489 |
| **2.0** | 0.3959 | 0.3617 | 0.1751 | 0.1680 |

### Key Observations

1. **Error Decreases with Larger ε**
   - At ε=0.1: Average count error is **8.75** (relative error: ~6.5% of count)
   - At ε=2.0: Average count error drops to **0.40** (relative error: ~0.3% of count)
   - **The improvement is dramatic:** 22× reduction in error across just 2 orders of magnitude of ε

2. **Variable Noise (Non-Deterministic)**
   - Standard deviations show that noise varies substantially from run to run
   - This is **by design**—randomness is essential for privacy
   - An attacker cannot reliably infer which value the noise took

3. **Mean Errors Smaller Than Count Errors**
   - Mean has sensitivity 0.30 vs. count sensitivity 1.0
   - Therefore, for the same ε, the mean statistic is **less noisy** (3.3× smaller sensitivity)
   - This illustrates how sensitivity directly determines noise magnitude: `scale = sensitivity / ε`

4. **Cost of Maximum Privacy**
   - At ε=0.1, the average count error is 8.75 among ~130 true students
   - This level of noise makes the count statistic almost unusable
   - A practical deployment would likely choose ε ∈ [0.5, 2.0]

### What the Results Mean

This experiment demonstrates why **differential privacy is called a "privacy-accuracy tradeoff"**:

- **For researchers:** You must balance your statistical needs against privacy requirements. Smaller ε means stronger privacy guarantees but noisier results.
- **For policy makers:** Choosing ε is a deliberate decision. Example scenarios:
  - Medical data (ε=0.1): Maximum privacy, accept 6-8% count errors
  - Student survey (ε=1.0): Balanced, 1% count errors
  - Public aggregates (ε=2.0): Minimal privacy, near-exact counts
- **Practical insight:** There is no "perfect" epsilon, it depends on the application's sensitivity and utility requirements.

---

## Files in This Assignment

- **`differential_privacy.ipynb`**: Complete implementation with explanations
- **`student_study_hours.csv`**: Synthetic dataset (200 rows, 1 column)
- **`all_runs.csv`**: All 200 individual run results (4 epsilons × 50 runs)
- **`summary_table.csv`**: Aggregated statistics per epsilon (means and standard deviations)
- **`privacy_vs_accuracy.png`**: Visualization of the privacy-accuracy tradeoff
- **`error_distribution.png`**: Boxplots showing error variability for each epsilon

---

## Technical Notes

### Laplace Mechanism
The assignment uses the **Laplace mechanism** for adding noise:
```
noise ~ Laplace(0, sensitivity / ε)
private_statistic = true_statistic + noise
```

This guarantees (ε, δ)-differential privacy with δ=0.

### Why This Works
- Laplace distribution is symmetric and exponentially decaying
- The probability of observing a given noisy value depends on ε
- An attacker cannot reliably distinguish if a specific student was in the dataset
- Larger ε → narrower Laplace distribution → smaller noise → weaker privacy guarantee

---

## Conclusion

This assignment shows that **differential privacy is practical and quantifiable**. By testing multiple epsilon values, we see exactly how much accuracy we lose for a given level of privacy. The choice of ε should be driven by:
1. **Privacy policy requirements** (How much privacy is needed?)
2. **Downstream analysis needs** (How much error can be tolerated?)
3. **Regulatory/organizational constraints** (What does compliance require?)

A well-chosen epsilon enables organizations to share aggregate statistics safely while protecting individual privacy.
