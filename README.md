# Analytics Test - Online Retail

This document describes the exploratory data analysis (EDA) for the online_reatail_II dataset.

## Database

For it's simplicty i decided to uploade the data to an AWS RDS PostgreSQL instance, AWS offers a free tier that is suitable for simple projetcs and will be enough for the our needs.

More information on how to setup the database can be found at AWS: https://aws.amazon.com/rds/postgresql/

**Set up**





### Purpose:
On a daily basis we have multiple different challenges and most of them includes understanding and bringing insights on how to improve and monitor a process with multiple backgrounds, for example, transactions, onboarding of clients, controls, etc....

So the main purpose of this test is:

  1. Understand how you approach a brand-new database with their particularities, and your line of thinking during the whole process.

  2. Identify how you present your work for both technical and business teams.

## Questions:

#### 1. How many different customers have bought a product from the company?

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
Develop a dashboard for a business user to have access to all data you find important for a daily monitoring of the company performance, feel free to go beyond the previous questions and bring more insights. You can use the BI tool of your preference to create the dashboard.

## Observation
We recommend that you upload the file into a database and use it to extract the information compiled when answering the questions and creating the dashboard, in the document include the queries you used to get to the answers.

## Conclusion
You will not have a chance to present this work personally so the document you send must be clear visually and demonstrate well how you developed your solution.
