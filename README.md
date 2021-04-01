# Analytics Test - Online Retail

## Overview

This document describes the exploratory data analysis (EDA) for the online_reatail_II dataset.

**Database**: AWS RDS Postgres instance
> (for more information: https://aws.amazon.com/rds/postgresql/)

**Dashboard**: Metabase on Heroku
>(for more information: https://www.metabase.com/docs/latest/operations-guide/running-metabase-on-heroku.html)


## Understanding the data:


- What are the dimensions of the dataset?
```sql
SELECT
    'online_retail' AS table,
    (
        SELECT COUNT(*)
        FROM online_retail

    ) as rows,
    (
        SELECT COUNT(*)
        FROM information_schema.columns
        WHERE table_name = 'online_retail'

    ) as Columns
```

table|rows|columns
:---:|:--:|:-----:
online_retail|1,067,371|8



We now know that our data contains 8 columns and around 1 million rows.<br>
But that information alone is not enough, let's see what else can help us understand
what our data contains.


- List table columns and data types:
```sql
SELECT
    column_name,
    data_type
FROM information_schema.columns
WHERE table_name = 'online_retail'
```

column_name|data_type
:----:|:------------------------------:
invoice|Character varying
stockcode|character varying
description|character varying
quantity|bigint
invoicedate|timestamp without time zone
price|double precision
customer_id|double precision
country	character|varying


- Are there any null values?
```sql
SELECT
    sum(case when invoice     is null then 1 else 0 end) as invoice_null,
    sum(case when stockcode   is null then 1 else 0 end) as stockcode_null,
    sum(case when description is null then 1 else 0 end) as description_null,
    sum(case when quantity    is null then 1 else 0 end) as quantity_null,
    sum(case when invoicedate is null then 1 else 0 end) as invoicedate_null,
    sum(case when price       is null then 1 else 0 end) as price_null,
    sum(case when customer_id is null then 1 else 0 end) as customer_id_null,
    sum(case when country     is null then 1 else 0 end) as country_null
FROM online_retail
```







## Now that we know what we are looking at, we can start to ask a few questions:

**1. How many different customers have bought a product from the company?**



**2. Which day had the most transactions happening?**



**3. What is the average value of a transaction?**



**4. Which products are most popular?**










```sql
select count(distinct customer_id) from online_retail
```
```
5942
```

#### 2. Which day had the most transactions happening?

```sql
select
    date_trunc('d', cast(invoicedate as timestamp)) as day,
    count(invoice) as transaction_count
from online_retail
group by invoicedate
order by transaction_count desc
limit 10

<center>
```
day|transaction_count
:---:|      :---:
2010-12-06 00:00:00|1350
2010-12-09 00:00:00|1304
2010-12-07 00:00:00|1202
2010-12-06 00:00:00|1194
2010-12-03 00:00:00|1186
2010-12-01 00:00:00|1184
2010-12-08 00:00:00|1182
2010-12-06 00:00:00|1136
2011-10-31 00:00:00|1114
2010-12-07 00:00:00|1072
</center>

#### 3. What is the average value of a transaction?


#### 4. Which products are most popular?

```sql
select
	stockcode,
	description,
	count(invoice) as invoice_count

from online_retail

group by 1, 2
order by 3 desc
limit 10
```
stockcode|description|invoice_count
  :---:  |   :---:   |    :---:
85123A|WHITE HANGING HEART T-LIGHT HOLDER|5817
22423|REGENCY CAKESTAND 3 TIER|4412
85099B|JUMBO BAG RED RETROSPOT|3444
84879|ASSORTED COLOUR BIRD ORNAMENT|2958
47566|PARTY BUNTING|2765
21232|STRAWBERRY CERAMIC TRINKET BOX|2613
20727|LUNCH BAG  BLACK SKULL.|2529
21931|JUMBO STORAGE BAG SUKI|2434
22469|HEART OF WICKER SMALL|2319
22411|JUMBO SHOPPER VINTAGE RED PAISLEY|2297

## Dashboard
Develop a dashboard for a business user to have access to all data you find important for a daily monitoring of the company performance.