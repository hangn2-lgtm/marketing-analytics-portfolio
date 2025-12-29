# IKEA — Digital Push: Optimizing Email Frequency + Personalization (Profit-LTV Memo)

**Author:** Claire Nguyen  
**Date:** 2025-10-23 
**Decision Type:** Prescriptive / decision analytics (cadence policy + content personalization)  
**Primary KPIs:**  
- **Cadence policy:** 6-month discounted **profit LTV**  
- **Personalization policy:** **Expected profit per customer / per send**  
**Artifacts:** Memo | Notebook | Dashboard  

---

## Executive summary (for someone seeing this case for the first time)
IKEA has been growing e-commerce, but many loyal customers still associate IKEA with the in-store “day-out” experience. Online sales are meaningful (about a quarter of revenue in the case) and skew toward newer, younger, or far-from-store customers—so management created a Digital Engagement Team to grow online without cannibalizing stores.

Email was the main lever, but the system had two problems:
1) Customers were being overwhelmed by too many promotional emails, which increased unsubscribes.
2) Even after IKEA capped email volume, the content was assigned randomly by department, which is fair internally but not necessarily profitable.

In this work, I built a simple, decision-ready approach:
- First, choose a **single weekly email frequency** that maximizes **6-month discounted profit LTV**.
- Second, once cadence is set, replace random rotation with **customer-level department selection** to maximize expected profit per send.

**Headline result (cadence):** The best policy is **2 emails/week**, producing **6-month discounted profit LTV = 27.819**, which is **+39.1% higher than 1 email/week**.  
**Next step (content):** Use the “random department assignment” history as built-in experiment data to predict each customer’s most profitable department theme.

---

## 1) Business decision (what was at stake)

### Context
IKEA’s email marketing had become cluttered because each of seven departments (Living Room, Bedroom, Kitchen Dining, Storage, Lighting, Home Decor, Outdoor) could email customers independently. Over time, many customers were receiving 10+ promotional emails per week, often with mixed themes. A new marketing manager, Sara, put a company-wide cap in place to protect the customer experience, because once a customer unsubscribes, IKEA loses the ability to influence them via email.

### Decision
This project answers two decisions IKEA needed to operationalize:
- **Cadence policy:** What weekly promotional email frequency should IKEA adopt as one simple, company-wide rule?
- **Content policy:** For each customer, which department theme should IKEA send to maximize expected profit (instead of rotating departments randomly)?

### Success metric
This project is successful if:
- The cadence rule maximizes **6-month discounted profit LTV** (balancing purchase stimulation and unsubscribe risk).
- The content rule improves **expected profit per customer / per send** relative to random rotation, without increasing fatigue signals.

---

## 2) Data (what I had)

### Sources
- The case provides the email-frequency experiment design and summary outcomes (unsubscribe rates and revenue for subscribers vs unsubscribers by frequency).
- I used the updated LTV spreadsheet (`IKEA-LTV.xlsx`) to compute discounted profit LTV.
- I used the customer-level modeling dataset described in the case for department personalization (training/validation split already provided).

### Unit of analysis
- **Cadence/LTV analysis:** Customer-month outcomes, because customers can transition from “subscribed” to “unsubscribed” over time.
- **Personalization analysis:** Individual customers, because the goal is to choose one best department for each person.

### Size + features (what’s in the case)
- **Part 1 experiment:** A randomized sample of **30,000 online customers** split into 4 groups (1/2/3/4 emails per week) and tracked over **6 months**, measuring monthly revenue and monthly unsubscription rate by group.
- **Part 2 modeling data:** Customer demographics (e.g., income), department contact history (most recent department emailed), purchase history by department (past 12 months), and email outcomes (clicked & purchased within 48 hours + order size). The modeling dataset is split into **200,000 training** and **50,000 validation** customers.

### Constraints / bias / imbalance
- The business is “noncontractual,” so unsubscribing does not mean customer churn; unsubscribers may still buy, but typically at a lower average level. That means an LTV model has to treat “unsubscribed” as a lower-value state, not a zero-revenue state.
- Purchases after email are rare in reality (the case states a true response rate of **0.84%**), so the modeling dataset was oversampled to roughly 50/50 buyers vs non-buyers to make modeling stable. Because oversampling inflates raw predicted probabilities, probabilities must be corrected back to real-world base rates when making decisions.

---

## 3) Method (what I did and why it fits)

### Baseline (what IKEA was doing)
- Before Sara’s fix, customers could receive very frequent emails across departments.
- Sara’s interim solution capped emails and randomly assigned each customer to one department theme each week, which reduced overlap and internal conflict but did not use customer preference signals.

### Final workflow (what I built)
**A) Cadence policy via discounted profit LTV**
- I converted revenue into profit using **COGS = 65%** (so margin = 35%), because the decision should be based on profit—not revenue.
- I used a **1% monthly discount rate** and summed expected profit over **6 months**.
- I computed expected monthly revenue as a weighted average of subscriber and unsubscribed revenue using the monthly unsubscription rate, because unsubscribing reduces value but does not eliminate it.
- I compared LTV across the four tested cadence policies (1–4 emails/week) and selected the highest.

**B) Content personalization via “next best department”**
- I treated IKEA’s random department assignments as built-in experimental variation that reveals which customers respond to which themes.
- I trained **one logistic regression model per department** to estimate purchase probability if that department emails the customer.
- I corrected predicted probabilities to reflect the true response rate (0.84%) rather than the oversampled 50/50 modeling rate.
- I translated purchase likelihood into expected profit by combining predicted probability with expected margin, using **order size** as the value component and applying IKEA’s COGS assumption.
- For each customer, I selected the department that maximizes **expected profit**, and compared that to two baselines: “send one department to everyone” and “random rotation.”

### Why this method
This approach is intentionally decision-first:
- The cadence question is not “what increases clicks,” but “what maximizes long-run profit when unsubscribes are real.”
- The content question is not “what is fair across departments,” but “what message earns the highest expected return for each customer.”

---

## 4) Evaluation (how I proved it works)

### Offline metrics
- **Cadence:** 6-month discounted profit LTV for each frequency policy.
- **Personalization:** holdout validation scoring (with careful probability calibration due to oversampling).

### Profit logic
- For cadence, the “winner” is the frequency with the highest 6-month discounted profit LTV.
- For personalization, I compared:
  1) **Personalized best department per customer**
  2) **Best single department for everyone**
  3) **Random department assignment** (equal probability across seven messages)

### Validation approach
- I used the provided training/validation split to avoid leakage.
- I treated calibration as a first-class requirement because oversampling can make a model look “strong” while still producing unrealistic probabilities in production.

---

## 5) Results (what changed)

### Hero result #1 (cadence)
✅ **2 emails/week** delivers the highest 6-month discounted **profit LTV = 27.819**  
For comparison:  
- 1/week = 19.996  
- 3/week = 26.431  
- 4/week = 25.115  

### Hero result #2 (lift vs alternatives)
✅ **2/week vs 1/week:** **+39.1%** profit LTV uplift  
✅ **2/week vs 3/week:** **+5.25%** profit LTV uplift  
✅ **2/week vs 4/week:** **+10.76%** profit LTV uplift  

### Practical implication
If IKEA wants one clean, scalable rule, “**2 emails/week**” is the best default because it maximizes profit over six months while keeping fatigue risk under control. Once cadence is fixed, personalization becomes the “second gear” that improves profit per send and reduces wasted outreach.

---

## 6) Risks & pitfalls (what could go wrong)

### Bias / ethics / privacy
Personalized marketing can feel invasive if it uses sensitive traits or implies too much knowledge. A safer approach is to anchor personalization primarily on purchase behavior and practical needs (e.g., storage vs lighting), and keep messaging framed as helpful rather than hyper-targeted.

### Failure modes
- If oversampling correction is wrong, probabilities will be inflated and the personalization policy will over-predict purchase.
- If the team optimizes for short-term conversion without guardrails, they can unintentionally increase unsubscribes and harm long-run value.

### Monitoring / retraining plan
I would monitor monthly:
- Unsubscribe rate by frequency and by department theme
- Profit/customer and profit/send by segment
- Conversion by recommended department vs actual department sent  
I would refresh models quarterly or when seasonality/assortment changes materially.

---

## 7) Recommendation (next Monday plan)

### Rollout plan
I would implement the simplest high-impact change immediately: switch the company-wide cadence to **2 emails/week**.

### Experiment plan
I would run an A/B test on content:
- **Treatment:** personalized best department per customer (profit-based)
- **Control:** random rotation (current baseline)  
I would set guardrails up front (unsubscribe rate and spam complaints) so we do not “win” short-term profit by damaging the channel.

### Next improvements
I would add controlled exploration so customers are not repeatedly shown the same department, and I would optimize toward long-run value (repeat purchases and retention) rather than only 48-hour conversion.

---

*Case context source:* IKEA case write-up (“IKEA’s Digital Push: Optimizing Email Frequency and Personalization”).  
