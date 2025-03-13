---
layout: post
title: "How to compute CMS' Initiation and Engagement of Substance Use Disorder Treatment"
date: 2025-03-13
author: "Charadata"
categories: [CMS, Medicaid]
tags: [Quality Rating System Measures, Initiation and Engagement of Substance Use Disorder Treatment]
---

# Guide to calculating the Initiation and Engagement of Substance Use Disorder Treatment (IET) Metric

## Overview
The **Initiation and Engagement of Substance Use Disorder Treatment (IET)** measure is used to evaluate the effectiveness of health plans and providers in ensuring timely access to treatment for individuals diagnosed with substance use disorder (SUD). It is commonly used by Medicaid Managed Care Organizations (MCOs) and follows **NCQA HEDIS** specifications.

The measure consists of two key components:

1. **Initiation of Treatment** – The percentage of members with a new SUD diagnosis who receive treatment within 14 days.
2. **Engagement in Treatment** – The percentage of members who engage in ongoing treatment by receiving at least two additional services within 34 days of initiation.

---

## Data Requirements

Before calculation, ensure you have access to the following data sources:

- **Medical claims** (ICD-10, CPT, HCPCS codes)
- **Pharmacy claims** (NDC drug codes)
- **Enrollment and eligibility data** (to confirm continuous enrollment)
- **Provider taxonomy data** (for provider specialty)
- **Diagnosis and procedure codes** (to identify SUD-related encounters)

---

## Step 1: Define the Eligible Population

### 1.1 Continuous Enrollment Criteria
- Members must be **continuously enrolled** for **five months**:  
  - The measurement month and the **four months before it**.
  - A **one-day gap** in enrollment is allowed.

### 1.2 Index Episode Identification
- Identify **new SUD diagnoses** from medical claims using valid **ICD-10** diagnosis codes.
- The **Index Episode Start Date (IESD)** is the first claim with an SUD diagnosis within the intake period.
- Exclude members with an SUD claim in the **60 days prior to the IESD** to ensure a **new episode**.

---

## Step 2: Calculate the Initiation of Treatment Rate

- Look for an **SUD treatment encounter** occurring within **14 days** of the IESD.
- Valid treatment encounters include:
  - **Outpatient, intensive outpatient, or partial hospitalization visits** (CPT, HCPCS codes)
  - **Inpatient admission**
  - **Medication treatment for SUD** (e.g., buprenorphine, naltrexone, methadone)

### Exclusions:
- Members in hospice
- Members with an SUD diagnosis due to tobacco use disorder only (i.e., no other SUD diagnosis)

---

## Step 3: Calculate the Engagement in Treatment Rate

- Identify members who **initiated treatment**.
- Check if they received **at least two additional SUD treatment services** within **34 days** after initiation.
- The additional treatment can include:
  - **Behavioral health therapy** (individual or group)
  - **Medication-assisted treatment (MAT)**
  - **SUD-related inpatient or outpatient visits**

---

## Step 4: Compute the Final Rates

1. **Initiation Rate** =  
   \[
   \frac{\text{Members who initiated SUD treatment within 14 days}}{\text{Eligible population with a new SUD diagnosis}} \times 100
   \]

2. **Engagement Rate** =  
   \[
   \frac{\text{Members who received 2 additional treatments within 34 days}}{\text{Members who initiated treatment}} \times 100
   \]

---

## Step 5: Reporting and Benchmarking

- Compare results to **NCQA HEDIS benchmarks**.
- Identify disparities by **age, gender, race/ethnicity, and provider type**.
- Develop strategies to **improve engagement**, such as:
  - Expanding **telehealth options**
  - Increasing **case management outreach**
  - Enhancing **provider education on MAT prescribing**

---

## Summary

| Step | Description |
|------|-------------|
| 1 | Identify the eligible population |
| 2 | Find SUD treatment within 14 days (Initiation) |
| 3 | Check for 2 additional services within 34 days (Engagement) |
| 4 | Calculate Initiation & Engagement rates |
| 5 | Report and analyze results |

By following these steps, you can accurately calculate and improve your **IET quality metric** to support individuals in substance use disorder treatment.
