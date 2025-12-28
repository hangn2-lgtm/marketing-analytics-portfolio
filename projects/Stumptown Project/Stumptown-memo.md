# Stumptown — Targeted Wholesale Sampling Memo (Honduras El Puente)

**Author:** Claire Nguyen  
**Date:** 2025-Nov-12  
**Decision Type:** Predictive + Profit-based targeting (B2B)  
**Artifacts:** Memo | Notebook | Dashboard/Gains Chart

---

## Executive Summary
Stumptown’s wholesale growth plateaued, and sending a $20 free 1-lb coffee sample to every account is expensive and inefficient.  
I built a **logistic regression targeting model** to estimate each account’s probability of placing a **bulk reorder (≥20 lbs)** and translated that score into a **profit threshold decision**.  
Result: **Targeting concentrates buyers in the highest-ranked accounts and improves expected profit at rollout scale (45,000 accounts) vs. blanket sampling.**

---

## 1) Business decision

### Context
Stumptown, a third-wave coffee pioneer, faced plateauing growth in its wholesale channel and wanted to demonstrate how marketing analytics can improve B2B efficiency.

### Decision
Move from **blanket sampling** (send free coffee to everyone) to a **targeted sampling strategy** for the seasonal single-origin **Honduras El Puente**.

### Success metric
Maximize **expected profit** for the remaining **45,000 accounts** in the database.

---

## 2) Data

### Sources
Internal wholesale customer database of ~60,000 active and lapsed accounts.

### Unit of analysis
Individual wholesale establishments (e.g., cafés, bakeries).

### Size + features
A randomized test set of **15,000 accounts**, including:
- Establishment type (e.g., `is_cafe`)
- Facility size (`facility_sqft`)
- Tenure
- Past case counts of blends (e.g., `hair_bender`, `cold_brew`)
- Other account / purchase history signals

### Constraints / bias / imbalance
Samples are expensive (**~$20 per unit total cost**). Targeting the wrong accounts creates immediate marketing losses.

---

## 3) Method

### Baseline
- Blanket sampling (all accounts), or
- Simple heuristic targeting (e.g., RFM)

### Final model / workflow
- Train a **logistic regression (logit)** model to estimate **P(reorder)** for a bulk reorder (≥20 lbs).
- Score each account with a predicted probability.
- Convert the probability into a **profit-based cutoff** and target only accounts above breakeven.

### Why this method
Compared with simple heuristics, logit allows richer predictors (facility size, establishment type, blend-specific history) and learns the weighting from data—producing a scalable, repeatable targeting rule.

---

## 4) Evaluation

### Offline metrics
- **Decile response rates** (response by rank bucket)
- **Gains chart** (share of buyers captured by top X% of accounts)

### Profit logic (turning predictions into decisions)
- Sample cost: **$20**
- Profit if reorder occurs: **$270** (after shipping + COGS)
- Breakeven response probability:
  \[
  p^* = 20 / 270 \approx 7.4\%
  \]
Target accounts with **P(reorder) ≥ 7.4%** to ensure positive expected profit.

### Validation approach
Fit the logit on the randomized test sample to validate separation (deciles/gains), then project expected outcomes for the remaining **45,000 accounts**.

---

## 5) Results

### Hero number #1 (profit at scale)
Rolling out to 45,000 accounts:
- **Targeted strategy:** **~$463K expected profit**
- **Send to all:** **~$159K expected profit**
➡️ **+$303.7K incremental expected profit** from targeting.

### Hero number #2 (gains / efficiency)
Targeting the **top 10%** of accounts captures **35% of buyers** — **3.5× better than random**.

### Practical implication
Stumptown can scale sampling with financial discipline: send expensive samples to the highest-potential accounts first, and expand only when expected profit stays positive.

---

## 6) Risks & pitfalls

### Bias / ethics / privacy
Over-reliance on features like `is_cafe` or `facility_sqft` could under-target non-traditional but high-growth partners (e.g., tech offices, hotels).

### Failure modes
Preference varies by origin (e.g., Ethiopia vs. Honduras). A model trained on one single-origin may not generalize perfectly to another.

### Monitoring / retraining plan
Run new test campaigns for different origins (e.g., Spring Ethiopia release) and refresh the model to reflect origin-specific demand patterns.

---

## 7) Recommendation (next Monday plan)

### Rollout plan
Score the remaining **45,000 accounts** and send samples only to accounts **above the 7.4% breakeven threshold**.

### Experiment plan
For the upcoming Ethiopia origin launch, run a small randomized A/B test:
- A: Use Honduras model targeting
- B: Re-estimate origin-specific model
Compare conversion and profit to validate generalization.

### Next improvements
Incorporate unstructured signals (reviews, barista feedback, notes) to understand *why* accounts reorder and improve targeting beyond purchase history.
