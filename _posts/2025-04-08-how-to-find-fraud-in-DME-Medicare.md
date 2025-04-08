---
layout: post
title: "How to find fraud in DME claims in one step"
date: 2025-04-08
author: "Charadata"
categories: [Medicare, Fraud Waste and Abuse]
tags: [Fraud Waste and Abuse, Durable Medical Equipment]
---

At [Data.CMS.gov](https://data.cms.gov/) there is a publicly available data set called Medicare Durable Medical Equipment, Devices and Supplies by Referring Provider and Service. For purposes of finding fraud, waste, and abuse, the year 2019 is a good one to look at, as it is prior to covid and with enough time for investigations to complete, and for disciplinary actions, if any, to become publicly available.

The arrangement of the data doesn't allow the same kind of analysis that might be done with claims data. There is one row per HCPCS per referring provider with summary data. And it looks like the set is limited to only referring provider-HCPCS combinations with more than ten claims.

Skimming the data, one thing that stands out is the number of patients seen, or at least billed, by some of the referring providers. Here the maximum number of beneficiaries among the HCPCS listed would actually be the <em>minimum</em> possible number of patients. Here is a simple list sorted by most to least number of beneficiares for each referring provider.  
<br>

```sql
-- Query in PostgreSQL for the top 25.
-- It was a mistake to keep the column names with capital letters! So tedious to type out the quotation marks each time.
SELECT 
	"Rfrg_NPI"
	, "Rfrg_Prvdr_Last_Name_Org" || ', ' || "Rfrg_Prvdr_First_Name" || ' ' || "Rfrg_Prvdr_Crdntls" AS Rfrg_Prvdr
	, "Rfrg_Prvdr_City"
	, "Rfrg_Prvdr_State_Abrvtn"
	, "Rfrg_Prvdr_Spclty_Desc"
	, MAX("Tot_Suplr_Benes") AS Min_Possible_Benes
	, MAX("Tot_Suplrs") AS Min_Possible_Suplrs
	, SUM("Tot_Suplr_Srvcs") AS Suplr_Srvcs
	, CAST(SUM("Tot_Suplr_Srvcs" * "Avg_Suplr_Mdcr_Pymt_Amt") AS DECIMAL(14,2)) AS Mdcr_Pymt_Amt
FROM postgres.public.mc_dme_rfrg_2019
WHERE 
	"Tot_Suplr_Benes" IS NOT NULL
GROUP BY 
	"Rfrg_NPI"
	, Rfrg_Prvdr
	, "Rfrg_Prvdr_City"
	, "Rfrg_Prvdr_State_Abrvtn"
	, "Rfrg_Prvdr_Spclty_Desc"
ORDER BY 
	Min_Possible_Benes DESC
LIMIT 25
```
<br>  
The results: Wow! That is a lot of fraud, especially from a single metric.  
<br>

<iframe title="Medicare 2019 top DME referrers by no. beneficiaries" aria-label="Table" id="datawrapper-chart-jZPyO" src="https://datawrapper.dwcdn.net/jZPyO/1/" scrolling="no" frameborder="0" style="width: 0; min-width: 100% !important; border: none;" height="1444" data-external="1"></iframe><script type="text/javascript">!function(){"use strict";window.addEventListener("message",(function(a){if(void 0!==a.data["datawrapper-height"]){var e=document.querySelectorAll("iframe");for(var t in a.data["datawrapper-height"])for(var r,i=0;r=e[i];i++)if(r.contentWindow===a.source){var d=a.data["datawrapper-height"][t]+"px";r.style.height=d}}}))}();
</script>

<br>  
Dr. Pirani, ranked third, was disciplined by Medical Board of California in 2023.

Number five, Dr. Canchola, was sentenced to ten years in prison for fraud.

Ranked seventh in terms of number of beneficiaries, Daphne Jenkins, was sentenced to 18 months in prison for fraud involving telemedicine.

In all, 13 of the top 25 received disciplinary action.

And this with only summary data instead of complete claims.

---

[Contact Charadata](https://charadata.github.io/#contact) for help finding fraud, waste, and abuse.
