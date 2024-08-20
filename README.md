# SQL-2

More SQL tasks.

Here:

I use operators to work with strings;
I use conditional operators and tools to handle missing values.
!!! In this task, data from four tables is used:

facebook_ads_basic_daily
facebook_adset
facebook_campaign
google_ads_basic_daily

### Task 1

In a CTE query, join the data from the above tables to obtain:

ad_date - the date the ad was displayed on Google and Facebook;
url_parameters - the part of the campaign URL that contains UTM parameters;
spend, impressions, reach, clicks, leads, value - campaign and ad metrics for specific days;
!!! If any of the metrics in the table are missing (i.e., the value is NULL), set the value to zero (i.e., 0).

### Task 2

Based on the results obtained using the CTE, retrieve the following data:

ad_date - the ad date;
utm_campaign - the value of the utm_campaign parameter from the utm_parameters field, which meets the following conditions:

all letters are lowercase

if the utm_campaign value in utm_parameters is 'nan', then in the result table, this value must be empty (i.e., NULL)
Tip: Use the substring function with a regular expression.

The total amount of ad spend, as well as the number of impressions and clicks, and the total conversion value on the appropriate day for the respective campaign;
CTR, CPC, CPM, ROMI on the appropriate day for the respective campaign.
!!! Do not use WHERE, but you can prevent division by zero errors by using the CASE operator.

## Solution code

ITH facebook_cte1 AS (
    SELECT ad_date,
        url_parameters,
        campaign_name,
        spend,
        impressions,
        reach,
        clicks,
        leads,
        value
    FROM facebook_ads_basic_daily AS fabd
    INNER JOIN 
        facebook_adset AS fa ON fabd.adset_id = fa.adset_id
    INNER JOIN 
    	facebook_campaign AS fc ON fabd.campaign_id = fc.campaign_id
),
facebook_google_cte2 AS (
    SELECT cte1.ad_date,
    	 cte1.url_parameters,
        cte1.spend,
        cte1.impressions,
        cte1.reach,
        cte1.clicks,
        cte1.leads,
        cte1.value   
    FROM facebook_cte1 AS cte1
    INNER JOIN google_ads_basic_daily AS gabd ON cte1.campaign_name = gabd.campaign_name
)
SELECT ad_date,
	lower(CASE WHEN url_parameters = 'nan' THEN NULL ELSE url_parameters END) AS utm_campaign,
    SUM(spend) AS total_spend,
    SUM(impressions) AS total_impressions,
    SUM(clicks) AS total_clicks,
    SUM(value) AS total_value,
    CASE WHEN SUM(COALESCE(clicks, 0)) > 0 THEN SUM(COALESCE(spend, 0)) / SUM(COALESCE(clicks, 0)) ELSE 0 END AS "CPC",
    CASE WHEN SUM(spend) = 0 THEN NULL ELSE (SUM(COALESCE(impressions,0)) / SUM(spend)) * 1000 END AS "CPM",
    CASE WHEN SUM(impressions) = 0 THEN NULL ELSE round((SUM(1.0 * COALESCE(clicks, 0)) / SUM(impressions)) * 100, 1) END AS "CTR %",
    CASE WHEN SUM(spend) = 0 THEN NULL ELSE ((SUM(value) - SUM(spend)) / SUM(spend)) * 100 END AS "ROMI"
FROM facebook_google_cte2 AS cte2
GROUP BY ad_date,
	url_parameters
