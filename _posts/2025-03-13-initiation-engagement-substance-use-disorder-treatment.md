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

### Materials required
1. The code sets provided in HEDIS MY [YYYY] Volume 2. (Requires payment.)
2. HEDIS Medication List Directory for the relevant NDCs. (Free upon signup.)
3. "Quality Rating System Measure Technical Specifications" published by CMS.
4. Your MCO's medical (ICD-10, CPT, HCPCS codes), pharmacy claims (NDCs), and member enrollment info.

Below are shown snippets of working with these sets in Microsoft SQL Server.

The measure consists of two key components:

1. **Initiation of Treatment** – The percentage of members with a new SUD diagnosis who receive treatment within 14 days.
2. **Engagement in Treatment** – The percentage of members who engage in ongoing treatment by receiving at least two additional services within 34 days of initiation.

## Step 1: Prepare the insured population table

At a minimum you'll need a unique ID number, date of birth, date of death (for exclusion purposes), and an enrollment time frame. Age will be calculated later and will be as of the encounter date. 

Often, there will be a SQL "gaps and islands" problem with the dates, an extra step to accurately reflect time periods where the insured was continuously enrolled. That's only if the table splits these periods if the insured changes plans, or addresses, but is otherwise still with the same Managed Care Organization. For IET, an encounter only counts if the member was enrolled 194 days prior to 47 days after the substance abuse disorder episode date. 

So, you'll have something like this to start:

```sql
SELECT 
  a.insured_id
  , a.date_of_birth
  , a.date_of_death
  , b.enrolled_from
  , b.enrolled_thru
INTO 
  #INSURED
FROM
  insured_table AS a
-- ugly continuous enrollment part 
INNER JOIN 
(
  SELECT
    z.insured_id
    , z.enroll_session
    , MIN(date_from) AS enrolled_from
    , MAX(ISNULL(date_thru, '9999-12-31')) AS enrolled_thru
  FROM 
  (
    SELECT
      y.*
      , SUM(y.enroll_split) OVER (PARTITION BY y.ins_id ORDER BY y.date_from) AS enroll_session	
    FROM 
    (
      SELECT
        x.*
        , CASE 
          WHEN x.ins_row = 1 THEN 0
          WHEN x.ins_row != 1 AND x.lag_thru = x.date_from THEN 0
          ELSE 1
        END AS enroll_split
      FROM 
      (
        SELECT
          insured_id
          , date_from
          , date_thru
          , (LAG(date_thru, 1) OVER (PARTITION BY insured_id ORDER BY date_from, ISNULL(date_thru, '9999-12-31'))) + 1 AS lag_thru
          , ROW_NUMBER() OVER (PARTITION BY insured_id ORDER BY date_from) AS ins_row 
        FROM 
          insured_table
      ) AS x
    ) AS y
  ) AS z
  GROUP BY
    z.insured_id
    , z.enroll_session
) AS b
  ON a.insured_id = b.insured_id
```

## Step 2a: Prepare an SUD diagnosis set with cohorts

This will make things easier later. At each step of the specifications -- episodes, initiation, and engagement -- you'll notice the same diagnosis groups mentioned.

- Alcohol Abuse and Dependence Value Set
- Opioid Abuse and Dependence Value Set
- Other Drug Abuse and Dependence Value Set

Add in the two ICD-10 diagnosis codes for counseling and surveillance of alcohol and drug abuse, Z71.41 and Z71.51, and you've got all of your diagnosis codes covered.

Create a temporary table of these now. This assumes you already have a table arranged with one row per diagnosis per claim. Facility claims can have a lot of diagnoses!

```sql
SELECT
  claim_id
  , diagnosis_date
  , CASE
    WHEN diagnosis_code IN -- alcohol abuse and dependence set
      THEN 'alcohol'
    WHEN diagnosis_code IN -- opioid abuse and dependence set
      THEN 'opioids'
    WHEN diagnosis_code IN -- other drug abuse and dependence
      THEN 'other'
    ELSE 'surveillance'
  END AS cohort
INTO 
  #DIAGNOSES
FROM 
  diagnosis_table
WHERE
  diagnosis_code IN (
    -- bunch of ICD-10 F codes for alcohol abuse and dependence
    OR -- bunch of ICD-10 F codes for opioid abuse and dependence
    OR -- F codes for other drug abuse and dependence
    OR 'Z71.41','Z71.51'
  )
```

## Step 2b: Find hospice patients for exclusion

Their encounters will be excluded in the next step. 

## Step 3: Identify qualifying episodes

Oh boy, here in the specifications you'll see a wall of text. New "episodes" will be from November 15 of the previous year to November 14 of the target year.

First, find SUD episodes. Flag inpatient discharges and monthly opioid treatment services because these are treated differently later.

```sql
SELECT
  c.*
  , DATEDIFF(YEAR, i.date_of_birth, c.date_of_service) -
		CASE
			WHEN DATEADD(YEAR, DATEDIFF(YEAR, i.date_of_birth, c.date_of_service), i.date_of_birth) > c.date_of_service THEN 1
			ELSE 0
	END AS episode_age  -- Calculate age
  , d1.cohort
  , CASE
    WHEN c.revenue_code IN -- ... Inpatient Stay value set
      THEN 1
    ELSE 0
  END AS ep_inpat
  , CASE
    WHEN c.procedure_code IN -- ... OUD Monthly Office Based Treatment value set
      THEN 1
    ELSE 0
  END AS ep_opioid
INTO 
  #EPISODES
FROM 
  claims_table AS c
-- Here is where you exclude not-continuously enrolled patients. And those that died.
INNER JOIN 
  #INSURED AS i
  ON i.insured_id = c.insured_id
  AND i.enrolled_from <= DATEADD(DAY, -194, c.date_of_service)
  AND i.enrolled_thru >= DATEADD(DAY, 47, c.date_of_service)
  AND YEAR(ISNULL(i.date_of_death, '9999-12-31')) != 2025 -- Change to whatever your measure year is.
INNER JOIN
  #DIAGNOSES AS d1
  ON c.claim_id = d1.claim_id
  AND d1.cohort IN ('alcohol','opioids','other')
LEFT JOIN  -- Need it again for the counseling and surveillance value set
  #DIAGNOSES AS d2
  ON c.claim_id = d1.claim_id
  AND d1.cohort IN ('surveillance')
WHERE 
-- WHERE... one of those many value sets is true. Lots to put here.
-- Also exclude hospice patients!
  AND c.insured_id NOT IN (SELECT insured_id FROM #HOSPICE)
```

Next, remove episodes found above where the insured already has a history of SUD. You'll have to do something like this.

```sql
#EPISODES AS e
...
WHERE NOT EXISTS (
	SELECT 
    c.insured_id
    , c.date_of_service
	FROM 
    claims_table AS c
	WHERE  
	-- ... procedure and revenue codes for SUD history 
	  AND DATEDIFF(DAY, c.date_of_service, e.date_of_service) BETWEEN 1 AND 194
	  AND e.insured_id = c.insured_id)
```

Lastly, remove SUD medication history. It's a similar process to the above except it includes pharmacy claims.

## Step 4: Find instance of initiation of treatment

This is best done in four steps, combined later.

1. Count episodes flagged as inpatient discharges or as monthly opioid treatment as initiation compliant. The episode date doubles as the initiation date.
2. Find medication dispensing and administration events up to 14 days after the episode date.
3. Find qualifying initiation treatment by a different provider on the same date of service as the episode.
4. Find qualifying initiation treatment, by any provider, up to 14 days after the episode.

## Step 5: Find treatment that qualifies as engagement

Here you are looking for three types of services that occur up to 34 days after the initiation of treatment.
1. Long-acting medication administration events. Includes the injection of naltrexone or buprenorphine, or a buprenorphine implant.
2. Engagement visits.
3. Engagement medication treatment events.

## Step 6: Compute the Final Rates

Remember that the denominator for this measure is based on episodes, not on the number of insured with an episode. One member can have multiple episodes, each requiring its own initiation and engagement treatment to be numerator compliant. 

Also, there can be only one episode per day.

According to the MAC Scorecard, the median rates in 2023 for initiation and engagement were 44.5 percent and 15.5 percent, respectively. [Go there](https://www.medicaid.gov/state-overviews/scorecard/main) to see how your MCO compares.

![Medicaid Initiation and Engagement of Substance Use Disorder Treatment map](/assets/img/MACScorecard-Initiation-and-Engagement-of-Substance-Use-Disorder-Treatment-map.jpeg "Medicaid Initiation and Engagement of Substance Use Disorder Treatment United States map")

---

[Contact Charadata](https://charadata.github.io/#contact) for more help with this metric.