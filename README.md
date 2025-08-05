Project Overview
This SQL project analyzes retail marketing performance metrics to evaluate campaign effectiveness, return on marketing investment (ROMI), and conversion rates across different dimensions.

Database Structure
The analysis is performed on a Marketing_kpi table which appears to contain the following fields:

campaign_name: Name of the marketing campaign

c_date: Date of campaign activity

revenue: Generated revenue

mark_spent: Marketing spend

clicks: Number of clicks

leads: Number of leads generated

orders: Number of orders placed

category: Type of campaign (social, banner, influencer, search)


Key Business Questions Answered

1. Overall ROMI (Return on Marketing Investment)

   SELECT 
    SUM(revenue) AS TotalRevenue,
    SUM(mark_spent) AS TotalSpend,
    ((SUM(revenue) - SUM(mark_spent)) / NULLIF(SUM(mark_spent), 0)) * 100 AS OverallROMI
FROM Marketing_kpi;

2. ROMI by Campaign

   SELECT 
    campaign_name,
    SUM(revenue) AS TotalRevenue,
    SUM(mark_spent) AS TotalSpend,
    ((SUM(revenue) - SUM(mark_spent)) / NULLIF(SUM(mark_spent), 0)) * 100 AS ROMI
FROM Marketing_kpi
GROUP BY campaign_name
ORDER BY ROMI DESC;

3. Campaign Performance by Date

   SELECT 
    c_date,
    campaign_name,
    SUM(mark_spent) AS TotalSpend
FROM Marketing_kpi
GROUP BY c_date,campaign_name
ORDER BY TotalSpend DESC;

4.Highest Spending Date

 SELECT TOP 1
    c_date,
    SUM(mark_spent) AS TotalSpend
FROM Marketing_kpi
GROUP BY c_date
ORDER BY TotalSpend DESC;

5. Revenue and Conversion Rates Analysis

   SELECT 
    c_date,
    SUM(revenue) AS TotalRevenue,
    SUM(leads) / SUM(clicks) * 100 AS ClickToLeadConversion,
    SUM(orders) / SUM(leads) * 100 AS LeadToOrderConversion
FROM Marketing_kpi
WHERE clicks > 0 AND leads > 0
GROUP BY c_date,revenue
ORDER BY ClickToLeadConversion DESC, LeadToOrderConversion DESC,revenue desc;

6. Average Order Value

   SELECT 
    c_date,
    SUM(revenue) / SUM(orders) AS AvgOrderValue
FROM Marketing_kpi
WHERE orders > 0
GROUP BY c_date
ORDER BY AvgOrderValue DESC;

7. Campaign Type Effectiveness

   SELECT 
    category,
    SUM(revenue) AS TotalRevenue,
    SUM(mark_spent) AS TotalSpend,
    ((SUM(revenue) - SUM(mark_spent)) / NULLIF(SUM(mark_spent), 0)) * 100 AS ROMI,
    (SUM(leads) / NULLIF(SUM(clicks), 0)) * 100 AS ClickToLeadConversion,
    (SUM(orders) / NULLIF(SUM(leads), 0)) * 100 AS LeadToOrderConversion
FROM Marketing_kpi
GROUP BY category
ORDER BY ROMI DESC;

8. Geographic Targeting Effectiveness

   WITH GeoTierData AS (
    SELECT 
        CASE 
            WHEN campaign_name LIKE '%tier1' THEN 'Tier 1'
            WHEN campaign_name LIKE '%tier2' THEN 'Tier 2'
            ELSE 'Other'
        END AS GeoTier,
        revenue,
        mark_spent,
        clicks,
        leads,
        orders
    FROM Marketing_kpi
    WHERE campaign_name LIKE '%tier%'
)
SELECT 
    GeoTier,
    SUM(revenue) AS TotalRevenue,
    SUM(mark_spent) AS TotalSpend,
    ((SUM(revenue) - SUM(mark_spent)) / NULLIF(SUM(mark_spent), 0)) * 100 AS ROMI,
    (SUM(CAST(leads AS FLOAT)) / NULLIF(SUM(CAST(clicks AS FLOAT)), 0)) * 100 AS ClickToLeadConversion,
    (SUM(CAST(orders AS FLOAT)) / NULLIF(SUM(CAST(leads AS FLOAT)), 0)) * 100 AS LeadToOrderConversion
FROM GeoTierData
GROUP BY GeoTier
ORDER BY ROMI DESC;

