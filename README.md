# NPS Performance Analysis

This project aims to analyze and monitor the Net Promoter Score (NPS) performance of healthcare companies using SQL queries and a Power BI dashboard. The project provides insights into customer satisfaction and loyalty, allowing stakeholders to make data-driven decisions to improve the customer experience.

The repository contains SQL queries to calculate NPS and a Power BI dashboard for visualizing and analyzing NPS metrics. By customizing the SQL queries and connecting to relevant data sources, this project can be applied to any healthcare company or organization that collects NPS data.

## Table of Contents
- [Project Overview](#project-overview)
- [Challenges](#challenges)
- [Dataset](#dataset)
- [SQL Query](#sql-query)
- [Power BI Dashboard](#power-bi-dashboard)
- [Usage](#usage)

## Project Overview

The project consists of two main components: a SQL query to calculate the NPS for each month based on patient satisfaction scores and a Power BI dashboard to visualize and monitor the NPS performance over time.

## Challenge

To understand the context and requirements of the project, please refer to the NPS document provided in the [NPS_Performance_Project.pdf](NPS_Performance_Project.pdf) file.

## Dataset

The dataset required for the second task is provided in the [NPS_data.csv](NPS_data.csv) file. This dataset contains information on NPS scores over time, including patient UUIDs, regions, companies, survey results, and session counts.

## SQL Query

To calculate the NPS for each month based on the satisfaction scores, use the following SQL query:

```sql
WITH latest_patient_scores AS (
  SELECT
    patient_uuid,
    CAST(survey_result AS INT) AS score,
    TO_DATE(year_month || '01', 'YYYYMMDD') AS month,
    ROW_NUMBER() OVER (
      PARTITION BY patient_uuid, year_month
      ORDER BY sessions DESC
    ) AS rank
  FROM
    nps_data
  WHERE
    survey_result IS NOT NULL
)
SELECT
  TO_CHAR(month, 'YYYY-MM') AS month,
  ROUND(
    (100.0 * (
      COUNT(CASE WHEN score > 8 THEN 1 END)
      - COUNT(CASE WHEN score < 7 THEN 1 END)
    ) / COUNT(DISTINCT patient_uuid)),
    0
  ) AS nps
FROM
  latest_patient_scores
WHERE
  rank = 1
GROUP BY
  month
ORDER BY
  month;
```

This query calculates the NPS by counting the number of promoters (scores > 8) and detractors (scores < 7) for each month. It rounds the NPS to the nearest whole number and returns the results grouped by month.

To run the query, replace the `nps_data` table with the appropriate table name in your database. Adjust the column names if necessary.

### Optimization Suggestions

The provided SQL code performs well on the given data. However, if the `scores` table grows significantly in size, it would be beneficial to optimize the table for efficient query execution. Consider the following suggestions:

**1. Indexing**: Ensure that the `scores` table is properly indexed to improve performance when grouping and filtering the data. Creating indexes on the `patient_id` and `date` columns can significantly enhance query execution speed. You can use the following SQL statements to create the necessary indexes:

```sql
CREATE INDEX ON scores (patient_id);
CREATE INDEX ON scores (date);
```
If you frequently filter scores for a specific patient within a date range, consider creating a multi-column index on `(patient_id, date)` to further improve performance when filtering data based on both columns. To create the multi-column index, use the following SQL statement:

```sql
CREATE INDEX ON scores (patient_id, date);
```

**2. Partitioning**: Partitioning the scores table can also enhance query performance, especially when dealing with large datasets. Partitioning the table by date or an appropriate column, such as `date`, can enable more efficient querying by scanning only relevant partitions. For example, you can partition the `scores` table by month using the `date` column as the partition key. Here's an example of creating partitioned tables for June and July:

```sql
CREATE TABLE scores_partitioned (
  patient_id INT NOT NULL,
  satisfaction INT NOT NULL,
  pain INT NOT NULL,
  fatigue INT NOT NULL,
  date DATE NOT NULL,
  PRIMARY KEY(patient_id, date)
) PARTITION BY RANGE (date);

CREATE TABLE scores_202006 PARTITION OF scores_partitioned
FOR VALUES FROM ('2020-06-01') TO ('2020-07-01');

CREATE TABLE scores_202007 PARTITION OF scores_partitioned
FOR VALUES FROM ('2020-07-01') TO ('2020-08-01');
```
Partitioning the table in this way allows queries that filter or group by date to scan only the relevant partitions, resulting in faster query times. However, keep in mind that the decision to partition a table should be carefully considered based on factors such as the size of the table, the frequency and complexity of queries, and the available resources.

These optimization suggestions can help ensure efficient query execution and improve performance as the `scores` table grows in size. Evaluate these recommendations based on your specific database system and workload to achieve optimal results.

## Power BI Dashboard

The Power BI dashboard provides visualizations and insights on the NPS performance. It follows the approach of "Mantra dos Dashboards," which focuses on providing an overview of the NPS performance, allowing for zooming and filtering to dive deeper, and providing detailed insights on demand. The dashboard aims to provide a holistic view of the NPS metrics for effective supervision and analysis of customer satisfaction and loyalty.

![Power BI Dashboard](nps_dashboard.gif)

The dashboard includes the following key visualizations and measures:

- Regional Analysis: Provides an overview of NPS scores by region, allowing comparisons and identification of potential regional differences.
- Company Analysis: Shows NPS scores by company, highlighting variations in customer satisfaction across different companies.
- Trend Analysis: Displays the trend of NPS scores over time, enabling identification of patterns, spikes, or drops.
- Active Patient Base: Analyzes the fluctuations in the active patient base, indicating changes in patient retention and acquisition.
- NPS by Session Count: Illustrates the correlation between NPS scores and the number of therapy sessions attended by patients.

## Usage

- **SQL Query:** Execute the SQL query against your database to obtain the NPS results. Customize the query as per your specific requirements or database system.
- **Power BI Dashboard:** Open the `NPS_Performance.pbix` file in Power BI Desktop. Interact with the visualizations, apply filters, and explore the NPS performance over time. Use the dashboard to gain insights into customer satisfaction and make informed decisions.

## Copyright
© 2023 Abel Tavares
