# SQL BigQuery E-commerce Website Exploring

## Table of Contents
1. [Introduction](#1-introduction)  
2. [Objectives](#2-objectives)  
3. [Dataset Overview](#3-dataset-overview)  
   - 3.1 [Importing the Dataset](#31-importing-the-dataset)  
   - 3.2 [Dataset Description](#32-dataset-description)  
4. [Data Processing and Exploratory Data Analysis (EDA)](#4-data-processing-and-exploratory-data-analysis-eda)  
5. [Analytical Questions and SQL Queries](#5-analytical-questions-and-sql-queries)  
   - 5.1 [Total Visits, Pageviews, Transactions, and Revenue (January–March 2017)](#51-total-visits-pageviews-transactions-and-revenue-januarymarch-2017)  
   - 5.2 [Bounce Rate by Traffic Source (July 2017)](#52-bounce-rate-by-traffic-source-july-2017)  
   - 5.3 [Revenue by Traffic Source (Weekly and Monthly) in June 2017](#53-revenue-by-traffic-source-weekly-and-monthly-in-june-2017)  
   - 5.4 [Average Number of Pageviews by Purchaser Type](#54-average-number-of-pageviews-by-purchaser-type)  
   - 5.5 [Average Number of Transactions per User (July 2017)](#55-average-number-of-transactions-per-user-july-2017)  
   - 5.6 [Average Money Spent Per Session (2017)](#56-average-money-spent-per-session-2017)  
   - 5.7 [Products Purchased Alongside “YouTube Men’s Vintage Henley” (July 2017)](#57-products-purchased-alongside-youtube-mens-vintage-henley-july-2017)  
   - 5.8 [Cohort Analysis: Product Views to Add-to-Cart Conversion](#58-cohort-analysis-product-views-to-add-to-cart-conversion)  
6. [Conclusion](#6-conclusion)  
## 1. Introduction:
The E-commerce Dataset in the public Google BigQuery dataset. The dataset is about the information of users sessions on their website collected in 2017.

Using this dataset, the author conducts various analyses to examine website activity during that year. These include calculating the bounce rate, identifying the days with the highest revenue, studying user behavior across pages, and performing other types of analysis. The objective of this project is to gain insights into the business performance, evaluate the efficiency of marketing activities, and analyze product-related trends.

To accomplish this, the author utilizes Google BigQuery to write and execute SQL queries, enabling effective data exploration and analysis.

## 2. Objectives:
The project aims to:
- Understand traffic sources and user behavior patterns.  
- Analyze e-commerce performance metrics such as revenue, transactions, bounce rate and product sales.  
- Provide data-driven recommendations for improving e-commerce business strategies.

## 3. Dataset Overview:
### 3.1 Importing the Dataset:
The dataset used in this project is the public Google Analytics sample dataset available on BigQuery:  
`bigquery-public-data.google_analytics_sample.ga_sessions_*`  

To use the dataset, follow these steps:
- Log in to your Google Cloud Platform account and create a new project if you don't have one.
- Navigate to the BigQuery console by selecting the BigQuery option in the Google Cloud Console.
- Select your project from the project selector dropdown.
- In the Explorer panel, click on the + ADD DATA button and choose "Pin a project".
- In the dialog that appears, enter the project ID: bigquery-public-data and click "Pin".
- Once the project is pinned, expand the bigquery-public-data project in the Explorer panel.
- Navigate to the google_analytics_sample dataset and expand it.
- You'll find multiple tables named ga_sessions_ and click on to open it.

### 3.2 Dataset Description:
The dataset contains only columns i used, structured as follows:
|Field Name|Data Type|Description|
|:----|:----|:----|
|fullVisitorId|STRING|The unique visitor ID.|
|date|STRING|The date of the session in YYYYMMDD format.|
|totals|RECORD|This section contains aggregate values across the session.|
|totals.bounces|INTEGER|Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.|
|totals.hits|INTEGER|Total number of hits within the session.|
|totals.pageviews|INTEGER|Total number of pageviews within the session.|
|totals.visits|INTEGER|The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session.|
|trafficSource.source|STRING|The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter.|
|hits|RECORD|This row and nested fields are populated for any and all types of hits.|
|hits.eCommerceAction|RECORD|This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected.|
|hits.eCommerceAction.action_type|STRING|The action type. Click through of product lists = 1, Product detail views = 2, Add product(s) to cart = 3, Remove product(s) from cart = 4, Check out = 5, Completed purchase = 6, Refund of purchase = 7, Checkout options = 8, Unknown = 0.Usually this action type applies to all the products in a hit, with the following exception: when hits.product.isImpression = TRUE, the corresponding product is a product impression that is seen while the product action is taking place (i.e., a product in list view).Example query to calculate number of products in list views:SELECTCOUNT(hits.product.v2ProductName)FROM [foo-160803:123456789.ga_sessions_20170101]WHERE hits.product.isImpression == TRUEExample query to calculate number of products in detailed view:SELECTCOUNT(hits.product.v2ProductName),FROM[foo-160803:123456789.ga_sessions_20170101]WHEREhits.ecommerceaction.action_type = 2AND ( BOOLEAN(hits.product.isImpression) IS NULL OR BOOLEAN(hits.product.isImpression) == FALSE )|
|hits.product|RECORD|This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data.|
|hits.product.productQuantity|INTEGER|The quantity of the product purchased.|
|hits.product.productRevenue|INTEGER|The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000).|
|hits.product.v2ProductName|STRING|Product Name.|


And you can see more at https://support.google.com/analytics/answer/3437719?hl=en




## 4. Data Processing and Exploratory Data Analysis (EDA):
- Preprocessed data to handle nested and repeated fields using `UNNEST` in SQL.
- Explored key metrics such as total sessions, revenue, and traffic sources to gain an initial understanding of the data.
- Performed data cleaning and filtering to focus on relevant e-commerce transactions.

**Total customers in 2017**
``` sql
SELECT
  COUNT(fullVisitorId) AS num_row
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`;
```
![image](https://github.com/user-attachments/assets/770e5c33-b84f-47b3-8693-8dcd139521b6)

**UNNEST hits and products**
```sql
SELECT 
  date, 
  fullVisitorId,
  eCommerceAction.action_type,
  product.v2ProductName,
  product.productRevenue,
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits) AS hits,
UNNEST(hits.product) as product
```
![image](https://github.com/user-attachments/assets/1f727ccd-56ab-424e-bcf2-d57e95d11058)

## 5. Analytical Questions and SQL Queries:

### 5.1 Total Visits, Pageviews, Transactions, and Revenue (January–March 2017)
Query: Calculate key metrics such as total visits, pageviews, transactions, and revenue for the first quarter of 2017.
``` sql 
SELECT
   EXTRACT(MONTH FROM PARSE_DATE("%Y%m%d",date)) month,
   SUM(totals.visits) visits,
   SUM(totals.pageviews) pagesviews,
   SUM(totals.transactions) transactions,
   ROUND(SUM(totals.totalTransactionRevenue)/POW(10,6),2) revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix between "0101" AND "0331"
GROUP BY month
```
![image](https://github.com/user-attachments/assets/a5019ce8-3e9c-483a-9029-1a0bb0d1fc1e)

The table shows the performance for the first three months of 2017, including total visits, pageviews, transactions, and revenue. All of these metrics increased over the three months, with steady growth in transactions and revenue. This is a good sign of improvement.
### 5.2 Bounce Rate by Traffic Source (July 2017)
Query: Calculate the bounce rate for each traffic source during July 2017.
``` sql 
SELECT
   trafficSource.source,
   SUM(totals.visits) visits,
   SUM(totals.bounces) total_bounces,
   ROUND(SUM(totals.bounces)/ SUM(totals.visits)*100, 2) bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY trafficSource.source
ORDER BY visits DESC
```
![image](https://github.com/user-attachments/assets/91179abc-8dc6-4f1d-8e79-b9324508b3ef)

The table provides an overview of website traffic from various sources and key metrics to evaluate user engagement and behavior. It highlights four elements: source, total visits, total bounces, and bounce rate.

The Google website received the highest traffic, with 38,400 visits in the first row. Out of these, 19,798 were single-page visits (bounces), resulting in a bounce rate of approximately 51.56%. Following Google, Direct traffic ranked second with 19,891 visits, 8,606 of which were bounces, leading to a bounce rate of about 43.27%.

In the third, YouTube.com generated 6,351 visits, with 4,238 bounces, resulting in a higher bounce rate of approximately 66.73%, compared to Google and Direct traffic. On the other hand, search.mysearch.com recorded the lowest total visits but the highest bounce rate. It had only 12 visits, with 11 being bounces, resulting in a bounce rate of approximately 91.67%. This indicates that most users from this source left without interacting with additional pages.

A high bounce rate from a source like search.mysearch.com suggests that visitors may not find the content relevant or that the landing page does not effectively engage them. Addressing high bounce rates is crucial, but it is equally important to analyze the context of each source and user behavior to develop effective improvement strategies. Generally, a lower bounce rate reflects better user engagement, while a higher bounce rate indicates users are leaving after viewing only one page.
### 5.3 Revenue by Traffic Source (Weekly and Monthly) in June 2017
Query: Analyze revenue generated by different traffic sources at weekly and monthly levels for June 2017.
``` sql 
WITH get_revenue_monthly AS 
(
   SELECT
      CASE WHEN 1=1 THEN "MONTH" END AS type, 
      trafficSource.source,
      CASE WHEN 1=1 THEN "2017M06" END AS time,
      SUM(totals.transactionRevenue)/POW(10,6) revenue
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
   GROUP BY trafficSource.source
),

get_revenue_weekly AS 
(
   SELECT
      CASE WHEN 1=1 THEN "WEEK" END AS type, 
      trafficSource.source,
      FORMAT_DATE("%YW%V", PARSE_DATE("%Y%m%d", date)) AS time,
   SUM(totals.transactionRevenue)/POW(10,6) revenue
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
   GROUP BY trafficSource.source, time
)

SELECT *
FROM get_revenue_monthly 
UNION ALL 
SELECT *
FROM get_revenue_weekly
ORDER BY revenue DESC
```
![image](https://github.com/user-attachments/assets/953c9083-dd57-4390-af56-5266309a7cf7)

This table shows the revenue in specific time periods like month or week in various sources.
In June 2017, direct traffic was the top revenue contributor with $97,231.62, followed by Google ($18,757.18) and dfa ($8,841.23). Weekly revenue trends showed a steady decline in direct traffic from $30,883.91 (Week 24) to $14,905.81 (Week 26). Low-performing sources like search.myway.com and chat.google.com generated minimal revenue, requiring further evaluation. Optimizing direct traffic, Google, and dfa while addressing low-performing sources can enhance overall performance.
### 5.4 Average Number of Pageviews by Purchaser Type
Query: Determine the average number of pageviews segmented by purchaser type.
``` sql 
WITH purchaser_data AS (
  SELECT
    DISTINCT fullVisitorId,
    totals.pageviews,
    CASE
      WHEN totals.transactions > 0 THEN 'purchaser'
      ELSE 'non_purchaser'
    END AS purchaser_type
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
  WHERE
    _TABLE_SUFFIX BETWEEN '0101' AND '1231'
)

SELECT
  purchaser_type,
  AVG(pageviews) AS avg_pageviews
FROM
  purchaser_data
GROUP BY
  purchaser_type
```
![image](https://github.com/user-attachments/assets/308d2220-d43a-4209-80a1-f7a4435c38ec)

The table illustrates the distinct behavior of customers who make purchases versus those who do not, focusing on the number of pageviews they generate.

**Pageviews for Purchasers vs. Non-Purchasers:** Users who make a purchase view significantly more pages on the website compared to non-purchasers. On average, non-purchasers generate 3.64 pageviews, while purchasers average 25.85 pageviews. This demonstrates that purchasers are 7.1 times more likely to browse more pages than non-purchasers.

**Engagement Patterns:** The higher average pageviews among purchasers suggest that users who are more engaged with the website's content have a greater likelihood of completing a purchase. This implies a positive correlation between user engagement and conversion.

**Behavioral Insights:** The data reflect varying user behaviors. Users who are interested in purchasing may spend more time exploring product details, reading reviews, and comparing options, leading to a higher number of pageviews.

**Conversion Optimization:** Leveraging this data, businesses can improve their website design and content to encourage greater engagement among potential purchasers. Enhancements could include better navigation, showcasing relevant products, and offering valuable content that guides users toward making a purchase.
### 5.5 Average Number of Transactions per User (July 2017)
Query: Calculate the average number of transactions per user who made a purchase in July 2017.
``` sql 
WITH get_info_july AS (
  SELECT
    CASE WHEN 1 = 1 THEN "201707" END AS time,
    COUNT(DISTINCT(CASE WHEN totals.transactions >=1 THEN fullVisitorId END )) AS Num_User,
    SUM(CASE WHEN totals.transactions > 0 THEN totals.transactions ELSE 0 END) AS transactions
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
)

SELECT 
  ROUND(transactions / Num_User, 2) AS avg_transactions_per_user
FROM get_info_july;
```
![image](https://github.com/user-attachments/assets/05c8b8c9-a3fa-43b1-972a-26af4eec6289)

The table calculates the average number of transactions per user who made a purchase in July 2017, resulting in 1.11. This indicates that, on average, a purchasing user made 1.11 transactions during the month. The data suggests that only a small percentage of users who made a purchase in July 2017 completed multiple transactions.
### 5.6 Average Money Spent Per Session (2017)
Query: Calculate the average revenue generated per session, including only sessions with purchases in 2017.
``` sql
SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE('%Y%m%d', date)) as Month,
    ROUND(SUM(totals.totalTransactionRevenue) / COUNT(*) / POW(10,6), 2) AS avg_revenue_per_session
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE totals.transactions > 0  -- Filter for sessions with transactions
GROUP BY Month
ORDER BY Month
```
![image](https://github.com/user-attachments/assets/1db1c270-f292-46bc-a76a-38f91cf41fb6)

The table illustrates the average revenue generated per session across different months.

**Overall Trend:**
The average revenue per session increased steadily from January to April, reaching its peak at $240.05 in April. After April, it experienced a decline in May and June, followed by a slight rebound in July and a substantial rise in August to $206.74.

**Month-to-Month Comparisons:**
- The most significant increase occurred between March and April, with a $70 jump.
- The steepest decline happened between April and May, with a drop of $118.
- Additional notable increases include $6.13 between February and March and $50.83 between July and August.
### 5.7 Products Purchased Alongside “YouTube Men’s Vintage Henley” (July 2017)
Query: Identify other products purchased by customers who bought the product “YouTube Men’s Vintage Henley” during July 2017.
``` sql 
WITH customer_id_purchased_VH AS
(
   SELECT 
     DISTINCT fullVisitorId as customer_purchased_VH
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
   UNNEST(hits) AS hits,
   UNNEST(hits.product) AS product
   WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
     AND product.productRevenue IS NOT NULL
)

SELECT 
  product.v2ProductName as Name, 
  SUM(product.productQuantity) as Amount
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` A 
  RIGHT JOIN customer_id_purchased_VH B 
    ON A.fullVisitorId=B.customer_purchased_VH,
UNNEST(hits) AS hits,
UNNEST(hits.product) AS product
WHERE A.fullVisitorId = B.customer_purchased_VH 
  AND product.v2ProductName <> "YouTube Men's Vintage Henley"
  AND product.productRevenue IS NOT NULL
GROUP BY product.v2ProductName
ORDER BY Amount DESC
```
![image](https://github.com/user-attachments/assets/f4e2d2dc-e7ca-44ba-a870-177f2b8508a8)

The data includes information about products purchased alongside the "YouTube Men's Vintage Henley" or other specific product combinations, allowing us to identify potential items that may be bought together with the "YouTube Men's Vintage Henley" in July 2017. This analysis offers valuable insights into customer preferences, product popularity, and potential opportunities for marketing and merchandising strategies. Further exploration of historical data, along with integration of customer demographics, could provide a deeper understanding of these trends.

### 5.8 Cohort Analysis: Product Views to Add-to-Cart Conversion
Query: Create a cohort map to calculate the conversion rate from product views to add-to-cart actions.
``` sql 
WITH productview AS (
  SELECT
      FORMAT_DATE("%Y%m", PARSE_DATE('%Y%m%d', date)) AS month,
      COUNT(eCommerceAction.action_type) AS num_product_view
  FROM
      `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
      UNNEST (hits) AS hits
  WHERE
      _table_suffix BETWEEN '0101' AND '0331'
      AND eCommerceAction.action_type = '1' 
  GROUP BY
      month
),
addtocart AS (
  SELECT
      FORMAT_DATE("%Y%m", PARSE_DATE('%Y%m%d', date)) AS month,
      COUNT(eCommerceAction.action_type) AS num_addtocart
  FROM
      `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
      UNNEST (hits) AS hits
  WHERE
      _table_suffix BETWEEN '0101' AND '0331'
      AND eCommerceAction.action_type = '2' 
  GROUP BY
      month
),
purchase AS (
  SELECT
      FORMAT_DATE("%Y%m", PARSE_DATE('%Y%m%d', date)) AS month,
      COUNT(eCommerceAction.action_type) AS num_purchase
  FROM
      `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
      UNNEST (hits) AS hits
  WHERE
      _table_suffix BETWEEN '0101' AND '0331'
      AND eCommerceAction.action_type = '6' 
  GROUP BY
      month
)
SELECT
  month,
  num_product_view,
  num_addtocart,
  num_purchase,
  ROUND(SAFE_DIVIDE(num_addtocart, num_product_view) * 100, 2) AS add_to_cart_rate,
  ROUND(SAFE_DIVIDE(num_purchase, num_product_view) * 100, 2) AS purchase_rate
FROM productview
JOIN addtocart USING (month)
JOIN purchase USING (month)
ORDER BY month
```
![image](https://github.com/user-attachments/assets/735f53b5-e4a7-4674-8095-3a7179505b94)

The table presents five different metrics and rates associated with user behavior from January to March 2017.

**Increasing Engagement:** From January to March, we see an increase in product views, adds to cart, and purchases. This suggests growing interest and engagement with the products.

**Improved Conversion Rates:** Both the add-to-cart rate and purchase rate increased over the three months. This indicates that a higher percentage of users who viewed products are adding them to their carts, and a higher percentage of users who added products to their carts are proceeding to purchase them.
## 6. Conclusion:
- This analysis provided the author with a valuable opportunity to learn about the marketing industry and the customer journey through an e-commerce dataset.

- By examining the dataset in BigQuery, the author gained insights into customer behavior through metrics such as bounce rate, transactions, revenue, visits, and purchases.

- The author also identified which marketing channels drive traffic and sales by analyzing referral sources. By investing in high-performing channels and optimizing underperforming ones, the business can improve marketing results.

- In conclusion, the exploration of the e-commerce dataset in BigQuery revealed critical insights that support strategic decision-making. These findings can help the business optimize operations, enhance customer experiences, and drive revenue growth.
