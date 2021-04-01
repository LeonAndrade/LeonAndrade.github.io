# Analytics Test - Online Retail

## Overview

This document describes my process for the Exploratory Data Analysis (EDA) of the `online_retail_II` dataset.

`TLDR`: The present document contains a more throughout description of the process, queries and reasoning i use as starting point for most analysis.

For **business users**, there is a **Dashboard** with filters that allow for more intuitive acess to the data.
it can be found here: [Online Retail - Dashboard](https://dash-analytics-test.herokuapp.com/dashboard/1?quarter_and_year=Q1-2010)

#### Technologies used to deploy and share the analysis

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

    ) as columns
```

table        |rows     |columns
:-----------:|:-------:|:-----:
online_retail|1,067,371|8

<br/><br/>
The dataset has **8 columns** and around **1 million rows**. Let's see what else we can find...
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


Although that may be very relevant because it represents a large fraction of the data and it looks like the main way to identify an unique customer, we still have plenty to work with and to maybe make some assumptions about why is this data missing or what other ways we can identify a customer.
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
    - `stockcode`: Categorical / Serial. It seems to be the unique ID of a product.
    - `price`: Quantitative. Assuming its the price for that row's `quantity` value.
    - `description`: Categorical. Description of the product.

  - Customers:
    - `customer_id`: Serial. Unique ID of a customer.
    - `country`: Categorical. Assuming it's the country of origin for an invoice/customer.

  - Orders:
    - `invoice`: Serial. Unique ID of each order. A single order can have multiple products.
    - `invoicedate`: Timestamp. Assuming it's in UTC-0 because most of the customers are from the UK.
    - `quantity`: Quantitative. Number of items ordered for any given product on an invoice.

<br/><br/>

## Understanding the Context

Now that we know more about what data and how much of it we have, it's time to ask some more questions to get a sense of context.
We want to use our data to answer some basic **_who_**, **_when_**, **_where_**, **_what_** and **_how many/much_** questions.
<br/><br/>

#### How many unique values?
```sql
SELECT 'invoice'     as column_name, count(distinct invoice)     as n_unique FROM online_retail
UNION
SELECT 'stockcode'   as column_name, count(distinct stockcode)   as n_unique FROM online_retail
UNION
SELECT 'description' as column_name, count(distinct description) as n_unique FROM online_retail
UNION
SELECT 'quantity'    as column_name, count(distinct quantity)    as n_unique FROM online_retail
UNION
SELECT 'invoicedate' as column_name, count(distinct invoicedate) as n_unique FROM online_retail
UNION
SELECT 'price'       as column_name, count(distinct price)       as n_unique FROM online_retail
UNION
SELECT 'customer_id' as column_name, count(distinct customer_id) as n_unique FROM online_retail
UNION
SELECT 'country'     as column_name, count(distinct country)     as n_unique FROM online_retail
ORDER BY n_unique DESC

```
column_name|unique_values
:---------:|:-----------:
invoice    |53628
invoicedate|47635
customer_id|5942
description|5698
stockcode  |5305
price      |2807
quantity   |1057
country    |43

<br/><br/>
This dataset contains data from **53,628 invoices**, made by approximately **5942 different customers** from *43 countries* who bought something between **5698** (description) and **5305** (stockcode) **unique products**
<br/><br/>

#### How many different customers have bought a product from the company?
```sql
SELECT 'customer_id' as column_name, count(distinct customer_id) as n_unique FROM online_retail
```
`5942`. _(*note that maybe 1/4 of the total is not being considered because of the null values*)_
<br/><br/>

### 2. Which day had the most transactions happening?

```sql
select
    cast(invoicedate as date) as day,
    count(invoice) as transaction_count
from online_retail
group by invoicedate
order by transaction_count desc
limit 1

```
day        |transaction_count
:---------:|:---------------:
2010-12-06|1350
<br/><br/>

#### 3. What is the average value of a transaction?

<br/><br/>

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

<br/><br/>

#### How many distinct products are being bought?
```sql
SELECT 'stockcode'   as column_name, count(distinct stockcode)   as n_unique FROM online_retail
UNION
SELECT 'description' as column_name, count(distinct description) as n_unique FROM online_retail
```
There are `5698` distinct `description` but only `5305` distinct `stockcode`.
This can mean that we have products with same code but variations, or that some descriptions or stockcodes were misplaced or missing.
<br/><br/>

#### Are the orders coming from how many countries?

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

## Dashboard
Develop a dashboard for a business user to have access to all data you find important for a daily monitoring of the company performance.