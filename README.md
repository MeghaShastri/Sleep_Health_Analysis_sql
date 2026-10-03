# 📊 Sleep Health & Workplace Stress Data Analysis

An end-to-end SQL analysis exploring how occupational demands, daily activity levels, and sleep patterns impact cardiovascular health risks across 374 workers.

---

## 🎯 Project Overview
This project analyzes the Kaggle **Sleep Health and Lifestyle Dataset** (374 records) using SQL to uncover actionable wellness insights across different career roles. 

The goal is to demonstrate how data-driven segmentation can highlight occupational burnout, sleep inefficiencies, and cardiovascular risk factors that standard corporate wellness programs miss.

---

## 🛠️ Tech Stack & Tools
* **Database:** SQLite / DB Browser for SQLite
* **Query Language:** SQL (`CASE WHEN` conditional flags, `GROUP BY`, `HAVING`, aggregation functions)
* **Dataset:** 374 worker records including Sleep Duration, Quality of Sleep, Daily Steps, Physical Activity Level, Stress Level, BMI Category, and Blood Pressure.

---

## 🔍 Key Findings & SQL Queries

### 1. Occupational Stress Tiers
**Business Question:** Which job roles suffer from severe perceived workplace stress?
* **Key Finding:** Sales Representatives, Scientists, and Salespersons report severe stress scores (≥ 7.0/10). Teachers, Accountants, and Software Engineers sit in the lowest stress tier.

```sql
SELECT 
    Occupation,
    COUNT(*) AS total_workers,
    ROUND(AVG("Stress_Lvl"), 2) AS avg_stress_score,
    
    -- Visual Flag for Stress
    CASE 
        WHEN AVG("Stress_Lvl") >= 7.0 THEN 'SEVERE STRESS'
        WHEN AVG("Stress_Lvl") >= 5.0 THEN 'MODERATE STRESS'
        ELSE 'LOW STRESS'
    END AS stress_flag

FROM Sleep_health_and_lifestyle_dataset
GROUP BY Occupation
ORDER BY avg_stress_score DESC;

```

---

### 2. Daily Physical Activity vs. Baseline Resting Heart Rate
**Business Question:** How do step tiers impact cardiovascular strain?
* **Key Finding:** Sedentary workers (< 5,000 steps daily) exhibit elevated resting heart rates (~74 bpm), whereas active roles (8,000+ steps) display a significantly lower baseline (~67 bpm).

```sql
SELECT 
	Occupation,
    COUNT(*) AS total_workers,
    ROUND(AVG("Daily_Steps"), 0) AS avg_daily_steps,
    ROUND(AVG("Heart_Rate"), 1) AS avg_heart_rate,
	
    CASE 
        WHEN "Daily_Steps" < 5000 THEN '1. Low Activity (<5k steps)'
        WHEN "Daily_Steps" BETWEEN 5000 AND 8000 THEN '2. Moderate Activity (5k-8k steps)'
        ELSE '3. High Activity (>8k steps)'
    END AS step_tier,
    
    
    -- Visual Flag for Activity & Cardiovascular Strain
    CASE 
        WHEN "Daily_Steps" < 5000 AND AVG("Heart Rate") >= 72 THEN 'SEDENTARY & ELEVATED HR'
        WHEN "Daily_Steps" >= 8000 THEN 'HIGH'
        ELSE 'BALANCED'
    END AS activity_cardio_flag

FROM Sleep_health_and_lifestyle_dataset
GROUP BY Occupation
ORDER BY avg_daily_steps ASC;

```

---

### 3. Sleep Duration vs. Sleep Efficiency Disparity
**Business Question:** Does total sleep duration guarantee high sleep quality?
* **Key Finding:** Sales Representatives are the only population experiencing severe sleep deprivation (< 6 hours). Shift workers like Nurses log normal sleep hours (~7.0 hours) but suffer from lower sleep quality scores (6.4/10) compared to corporate desk workers.

```sql
SELECT 
    Occupation,
    ROUND(AVG("Sleep_Duration"), 1) AS avg_sleep_hours,
    ROUND(AVG("Quality_of_Sleep"), 2) AS avg_sleep_quality,
    
    -- Visual Flag for Unrestful Sleep (Long sleep hours but low quality score)
    CASE 
        WHEN AVG("Sleep_Duration") >= 7.0 AND AVG("Quality_of_Sleep") < 6.0 
            THEN 'POOR SLEEP EFFICIENCY (Long sleep, low quality)'
        WHEN AVG("Sleep_Duration") < 6.0 
            THEN 'SLEEP DEPRIVED (<6 hrs)'
        ELSE 'RESTFUL SLEEP'
    END AS sleep_efficiency_flag

FROM Sleep_health_and_lifestyle_dataset
GROUP BY Occupation
ORDER BY avg_sleep_hours DESC;

```

---

### 4. Clinical Blood Pressure Risk Prevalence
**Business Question:** Which occupations exceed standard clinical hypertension cutoffs (≥ 130/80 mmHg)?
* **Key Finding:** Using strict Stage 1 Hypertension thresholds, almost all occupational groups display elevated risk due to overall sample characteristics. Accountants serve as the dataset's sole normal baseline exception due to regular hours and low stress.

```sql
SELECT 
    Occupation,
    COUNT(*) AS total_workers,
    SUM(CASE WHEN systolic_bp >= 130 OR diastolic_bp >= 80 THEN 1 ELSE 0 END) AS hypertensive_count,
    ROUND(
        100.0 * SUM(CASE WHEN systolic_bp >= 130 OR diastolic_bp >= 80 THEN 1 ELSE 0 END) / COUNT(*), 
        1
    ) AS hypertensive_pct,
    
    -- Visual Flag for Clinical Risk Prevalence
    CASE 
        WHEN (100.0 * SUM(CASE WHEN systolic_bp >= 130 OR diastolic_bp >= 80 THEN 1 ELSE 0 END) / COUNT(*)) >= 50.0 
            THEN 'CRITICAL: MAJORITY HYPERTENSIVE'
        WHEN (100.0 * SUM(CASE WHEN systolic_bp >= 130 OR diastolic_bp >= 80 THEN 1 ELSE 0 END) / COUNT(*)) >= 25.0 
            THEN 'ELEVATED RISK (>25%)'
        ELSE 'LOW RISK'
    END AS bp_risk_flag

FROM Sleep_health_and_lifestyle_dataset
GROUP BY Occupation
HAVING COUNT(*) >= 10
ORDER BY hypertensive_pct DESC;

```

---

### 5. Demographics Deep-Dive: Occupation & Gender Matrix
**Business Question:** Does demographic segmentation reveal specific risk patterns?
* **Key Finding:** High physical activity (8,000+ steps) alone does not protect shift workers from elevated blood pressure or high BMI if sleep quality and occupational stress remain unmanaged.

```sql
SELECT 
    Occupation,
    Gender,
    COUNT(*) AS sample_size,
    
    -- Average Metrics
    ROUND(AVG("Quality_of_Sleep"), 2) AS avg_sleep_quality,
    ROUND(AVG("Daily_Steps"), 0) AS avg_daily_steps,
    ROUND(AVG(systolic_bp), 1) || '/' || ROUND(AVG(diastolic_bp), 1) AS avg_blood_pressure,
    
    -- Visual Risk Indicator 1: BMI Status
    CASE 
        WHEN ROUND(100.0 * SUM(CASE WHEN "BMI_Category" IN ('Overweight', 'Obese') THEN 1 ELSE 0 END) / COUNT(*), 1) >= 50.0 
            THEN 'HIGH BMI RISK'
        ELSE 'Normal Range'
    END AS bmi_flag,
    
    -- Visual Risk Indicator 2: Blood Pressure Status
    CASE 
        WHEN ROUND(100.0 * SUM(CASE WHEN systolic_bp >= 130 OR diastolic_bp >= 80 THEN 1 ELSE 0 END) / COUNT(*), 1) >= 50.0 
            THEN 'HIGH BP RISK'
        ELSE 'Normal Range'
    END AS bp_flag

FROM Sleep_health_and_lifestyle_dataset
GROUP BY Occupation, Gender
HAVING COUNT(*) >= 3
ORDER BY Occupation, Gender;

```

---

## 📈 Key Takeaways for Corporate Wellness Strategy
  **1. Focus Beyond Movement:** Physical activity (steps) isn't a complete fix for poor health outcomes when shift disruption and high stress are present.
  **2. Targeted Interventions:** High-stress commercial roles (Sales Reps) require workload/recovery management, while healthcare roles require shift recovery support.
  **3. Data Literacy Value:** Demonstrates how multi-variable SQL aggregation reveals insights that surface-level averages miss.

---
