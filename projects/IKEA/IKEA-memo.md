# IKEA — Email Cadence + Department Personalization (Profit-LTV Memo)

**Author:** Claire Nguyen  
**Date:** 2025-10-25  
**Decision Type:** Prescriptive / decision analytics (policy + personalization)  
**Artifacts:** Memo | Notebook | Dashboard  

---

## Executive summary
IKEA’s promo emails have a real tradeoff: more sends can drive short-term sales, but they also increase fatigue and unsubscribes—which hurts long-term value.  
In this project, I answered two practical questions: **(1) what send frequency maximizes profit over the next 6 months**, and **(2) what department theme should each customer get instead of rotating content at random**.

**Bottom line:** A simple default of **2 emails/week** produces the highest **6-month discounted profit LTV (27.819)**—a **+39.1% uplift vs 1 email/week**. Personalization is the next layer: once cadence is set, we can improve efficiency by sending the most relevant department theme per customer rather than “fair but random” rotation.

---

## 1) Business decision (what was at stake)

### Context
IKEA’s email program needed to be scalable and profitable. The team faced two tensions:
- Sending too frequently risks **fatigue → unsubscribes → lower long-term value**
- Random department rotation is “fair” internally, but it can leave **profit on the table** if customers clearly prefer certain categories.

### Decision
1) Pick one weekly email cadence that maximizes **long-run profit**.  
2) Replace random department rotation with **customer-level department selection** to maximize expected profit.

### Success metric
Maximize **6-month discounted profit LTV**, and beat **random outreach** on expected profit per customer / per send.

---

## 2) Data (what I had)

### Sources
- Updated LTV spreadsheet: `IKEA-LTV.xlsx`
- Customer-level modeling dataset from the case / notebook

### Unit of analysis
- **Cadence/LTV:** customer-month transitions (subscribed vs unsubscribed) by frequency policy  
- **Personalization:** individual customers with features used to predict purchase by department

### Size + features
- **LTV inputs:** unsubscribe rate + revenue for subscribers vs unsubscribers under each frequency (1–4 emails/week)  
- **Personalization features:** demographics, purchase history, department indicators, order size

### Constraints / bias / imbalance
Purchases are rare events in the raw population, so the modeling dataset uses oversampling; probability estimates need correction to stay calibrated.

---

## 3) Method (what I did and why it fits)

### Baseline
- **Cadence:** compare the tested policies (1–4 emails/week)  
- **Content:** random department rotation (each theme equally likely)

### Final workflow
**A) Profit LTV policy**
- Converted revenue into profit using **COGS = 65% (margin = 35%)**
- Discounted future profit at **1% per month**, summed over **6 months**
- Compared LTV across frequency options to pick the best policy

**B) Department personalization**
- Trained department purchase models (one per department)
- Corrected for oversampling to avoid inflated probabilities
- Ranked customers by expected profit and selected the best department theme per customer

### Why this approach
It keeps the analysis tied to what leadership actually cares about: **profit and long-run value**, not vanity engagement metrics. It also separates the problem cleanly:
- First set a **simple cadence policy**
- Then optimize content **within that policy**

---

## 4) Evaluation (how I proved it works)

### Offline metrics
- **Cadence:** 6-month discounted profit LTV for each frequency option  
- **Personalization:** holdout validation performance + calibrated probabilities

### Profit logic
- Compare expected profit under three approaches:
  1) Personalized department theme (best-by-profit per customer)
  2) Best single department for everyone
  3) Random rotation

### Validation approach
Used the provided training/validation split to prevent leakage and monitored probability calibration due to oversampling.

---

## 5) Results (what changed)

### Hero result #1 — Best cadence
✅ **2 emails/week** produces the highest **6-month discounted profit LTV = 27.819**  
(1/week = 19.996 | 3/week = 26.431 | 4/week = 25.115)

### Hero result #2 — Lift vs alternatives
✅ **+39.1% LTV uplift** for **2/week vs 1/week**  
(+5.25% vs 3/week; +10.76% vs 4/week)

### Practical implication
Adopt **2 emails/week** as the default policy (high value, low complexity). Then use personalization to make each send count more by matching customers to the department theme most likely to drive profitable purchase.

---

## 6) Risks & pitfalls (what could go wrong)

### Bias / ethics / privacy
Over-personalization can feel creepy. Keep features and messaging focused on being helpful (avoid sensitive signals and “you were looking at…” vibes).

### Failure modes
- Probability miscalibration if oversampling correction is wrong  
- Over-optimizing for short-term conversion could increase unsubscribes and reduce long-term value

### Monitoring / retraining plan
Monthly monitoring: unsubscribe rate, profit/customer, conversion by department and segment. Refresh models quarterly (or when merchandising/seasonality shifts materially).

---

## 7) Recommendation (next Monday plan)

### Rollout plan
Switch default cadence to **2 emails/week** immediately.

### Experiment plan
A/B test personalization vs random rotation with guardrails:
- Primary: incremental profit and long-run value
- Guardrails: unsubscribe rate, spam complaints, fatigue signals

### Next improvements
Add co
