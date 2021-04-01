# Analytics Test - Online Retail

## Overview

This document describes the exploratory data analysis (EDA) for the online_reatail_II dataset.

**Database**: AWS RDS Postgres instance
> (for more information: https://aws.amazon.com/rds/postgresql/)

**Dashboard**: Metabase on Heroku
>(for more information: https://www.metabase.com/docs/latest/operations-guide/running-metabase-on-heroku.html)

<br/>

## Understanding the Data
#### What are the dimensions of the dataset?
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

table        |rows     |columns
:-----------:|:-------:|:-----:
online_retail|1,067,371|8


<br/>
This dataset has **8 columns** and around **1 million rows**. Let's see what else we can find about this dataset...
<br/><br/>

#### Are there any null/missing values?
```sql
SELECT
SELECT 'invoice'     as column_name, sum(case when invoice     is null then 1 else 0 end) as null_values FROM online_retail
UNION
SELECT 'stockcode'   as column_name, sum(case when stockcode   is null then 1 else 0 end) as null_values FROM online_retail
UNION
select 'description' as column_name, sum(case when description is null then 1 else 0 end) as null_values FROM online_retail
UNION
SELECT 'quantity'    as column_name, sum(case when quantity    is null then 1 else 0 end) as null_values FROM online_retail
UNION
SELECT 'invoicedate' as column_name, sum(case when invoicedate is null then 1 else 0 end) as null_values FROM online_retail
UNION
SELECT 'price'       as column_name, sum(case when price       is null then 1 else 0 end) as null_values FROM online_retail
UNION
SELECT 'customer_id' as column_name, sum(case when customer_id is null then 1 else 0 end) as null_values FROM online_retail
UNION
SELECT 'country'     as column_name, sum(case when country     is null then 1 else 0 end) as null_values FROM online_retail
order by null_values desc
FROM online_retail
```
column_name|null_values
:---------:|:---------:
customer_id|243007
description|4382
invoice    |0
stockcode  |0
invoicedate|0
price      |0
quantity   |0
country    |0


<br/><br/>
It seems that about a 1/4 of the values are missing from our `customer_id` column.


Although that may be very relevant because it represents a large fraction of the data and it looks like the main way to identify a unique customer, we still have plenty to work with and maybe even ask some questions about why this data is missing or what other ways we can identify an unique customer.
<br/><br/>



#### What are the attributes (columns) and which data types they hold?
```sql
SELECT
    column_name,
    data_type
FROM information_schema.columns
WHERE table_name = 'online_retail'
```

column_name|data_type
:---------:|:--------------------------:
invoice    |character varying
stockcode  |character varying
description|character varying
quantity   |bigint
invoicedate|timestamp without time zone
price      |double precision
customer_id|double precision
country	   |character varying

</br></br>
From a quick look at this we can say that this data is about orders made by customers.

  - Products:
    - `stockcode`: Categorical / Serial. It seems to be the unique id of a product.
    - `price`: Quantitative. Assuming its the price for that row's `quantity` value.
    - `description`: Categorical. Description of the product.

  - Customers:
    - `customer_id`: Serial. Unique id of a customer.
    - `country`: Categorical. Assuming it's the country of origin for an invoice/customer.

  - Orders:
    - `invoice`: Serial. Unique id of each order. same order can have multiple products.
    - `invoicedate`: Timestamp. Assuming it's in UTC-0 because most of de customer are from the UK.
    - `quantity`: Quantitative. Number of items ordered for any given product of an invoice.


With this new information we can now start asking some questions to get some context from all this data.

  - How many unique values?
    - How many unique customers? (*note that maybe 1/4 of the total is not being considered because of the null values*)
    - How many distinct products are being bought?
    - Are the orders coming from how many countries?

  - What time window are we looking at?
    - Are we looking for short-term daily, weekly basis?
    - Or we want to see monthly, quarterly reports?

  - Are there any relevant outliers?
    - What is the range of prices?
    - How many products per invoice?
    - How many customers per country?

  - How many different customers have bought a product from the company?
  - Which day had the most transactions happening?
  - What is the average value of a transaction?
  - Which products are most popular?

</br></br>













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