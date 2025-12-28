# Nuova Simonelli — Round 2 Upgrade Targeting Memo

**Author:** Claire Nguyen  
**Date:** 2025-10-15  
**Decision Type:** Predictive + Economic (profit-based targeting)  
**Primary KPI:** Expected Round-2 profit (evaluated on validation sample)  
**Artifacts:** Memo | Notebook | Dashboard

---

## Executive Summary
Round 1 of the upgrade campaign was profitable, but most customers said “no.” In Round 2, response is expected to drop significantly, so calling everyone again risks losing money.

I used Round-1 data to build a **profit-based targeting policy**: predict each customer’s Round-2 upgrade probability, apply a breakeven cutoff, and contact only the customers with positive expected value.

**Result:** A **Random Forest** targeting strategy maximized expected Round-2 profit (**~$271K**) by targeting **3,675** customers (validation-based estimate), instead of blasting the full list.

---

## Decision statement
**Which customers should we contact in Round 2 to maximize expected profit, instead of calling everyone again?**

---

## 1) Business decision

### Context
Round 1 was profitable, but the majority declined the upgrade. In Round 2, expected response is lower, so a blanket re-contact strategy can easily fall below breakeven.

### Decision
Use a model trained on Round-1 outcomes to **score customers** and **only contact** those whose expected return clears breakeven.

### Success metric
Round-2 response and profit **above breakeven**, where breakeven response rate is:
\[
p^* = 64 / 2500 \approx 2.56\%
\]

---

## 2) Data

### Sources
Case dataset: `Nuova Simonelli.csv` (Round-1 outcomes + customer features).

### Unit of analysis
Customer account / café establishment.

### Size + features
- **45,000 customers**
- Outcome: `buy1` (accepted Round-1 upgrade)
- Predictors include: behavioral recency/frequency/spend, product/supply purchase history, geographic proxies, and café type (categorical).

### Constraints / bias / imbalance
- Round-2 acceptance is not observed; the case assumes Round-2 probability is **half of Round 1**, so predictions must be adjusted:
  \[
  p_2 = p_1 / 2
  \]
- Must train on the training split and evaluate economics on the validation split.

---

## 3) Method

### Baseline
Contact everyone again. Since Round-1 response was **4.14%**, Round-2 is assumed ~**2.07%**, which is **below** breakeven (**2.56%**)—so “call all” is expected to lose money.

### Final model / workflow
1) Train multiple classifiers on the training set to predict `buy1`:
   - Logistic regression (baseline interpretability)
   - Tree-based models (Decision Tree / Random Forest)
   - Boosting
   - Neural net (MLP)
2) Convert predicted Round-1 probabilities to Round-2 estimates using:
   \[
   p_2 = p_1 / 2
   \]
3) Apply a profit-based targeting rule:
   - Target if \( p_2 \ge 2.56\% \)
   - Compute expected profit on validation:
     \[
     \text{Profit} = (\#\text{upgrades} \times 2500) - (\#\text{contacted} \times 64)
     \]
4) Select the model that maximizes **out-of-sample profit**.

**Chosen model:** Random Forest (best expected Round-2 profit on validation).

### Why this method
This is a “model-to-decision” problem: the best model isn’t the one with the fanciest accuracy—it’s the one that drives the best **profit** after applying real campaign unit economics.

---

## 4) Evaluation

### Offline metrics
- **AUC** to check ranking quality
- **Expected profit** on the validation sample to choose the decision policy

### Profit logic
- Contact cost = **$64**
- Profit per upgrade = **$2,500**
- Breakeven response = **2.56%** (= 64/2500)
- Adjust Round-2 probability: \( p_2 = p_1 / 2 \)

### Validation approach
Compare models using expected profit on the **13,500-customer validation set** (unseen during training) to reduce overfitting risk.

---

## 5) Results

### Hero number #1 (Round 2 decision result)
**Random Forest** achieved the highest expected Round-2 profit: **~$271K**, while targeting **3,675** customers.

### Hero number #2 (Round 1 context)
Round 1 economics showed why the upgrade matters:
- 45,000 contacted → 1,862 upgrades (**4.14%**) → estimated profit **~$1.775M**
But Round 2 needs targeting to stay above breakeven.

### Practical implication
Run Round 2 as a **targeted campaign** using Random Forest scores for “who to call,” and use logistic regression as a companion model to explain key drivers to stakeholders when needed.

---

## 6) Risks & pitfalls (what could go wrong)

### Bias / ethics / customer experience
Re-contacting customers who already said “no” may create customer annoyance or brand damage that isn’t captured in short-term profit math.

### Failure modes
- **Probability inflation** if you forget the Round-2 adjustment (\(p_2 = p_1 / 2\))
- **Overfitting** if decisions are based on training performance rather than validation economics

### Monitoring / retraining plan
Track realized response and profit by score bands each campaign, recalibrate the “÷2” assumption if actual Round-2 response differs, and refresh the model as behavior and product adoption shift.

---

## 7) Recommendation (next Monday plan)

### Rollout plan
Score eligible accounts with Random Forest, apply the Round-2 adjustment, and contact only the segment above breakeven (starting with the highest scores).

### Experiment plan
Run a controlled test:
- Treatment: targeted list (model-based)
- Control: small random holdout (or “no contact” holdout)
Measure incremental upgrades, profit, and negative signals (opt-outs/complaints).

### Next improvements
Replace the single “50% drop” assumption with a **segment-specific** adjustment and expand optimization beyond short-term profit to include longer-term value and customer relationship costs.

