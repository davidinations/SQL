Order (Case Study Questions):

The following case study questions require some data cleaning steps
before we start to unpack key business questions in more depth. In a
single query, perform the following operations and generate a new table
in the data_mart schema named clean_weekly_sales:

1.  Convert the week date to a DATE format.

UPDATE test

SET date = STR_TO_DATE(date, '%d/%m/%y');

2.  Add a week_number as the second column for each week_date value, for
    example any value from the 1st of January to 7th of January will be
    1, 8th to 14th will be 2 etc.

> -- Add the week_number column
>
> ALTER TABLE weekly_sales
>
> ADD week_number INT;
>
> -- Update the week_number column
>
> UPDATE weekly_sales
>
> SET week_number = ceiling((DAY(week_date)) / 7);

-- Update column position into 2<sup>nd</sup> column position

ALTER TABLE weekly_sales

MODIFY COLUMN week_number INT AFTER week_date;

3.  Add a month_number with the calendar month for each week_date value
    as the 3rd column.

> -- Add the month_number column
>
> ALTER TABLE weekly_sales
>
> ADD month_number INT;
>
> -- Update column position into 3rd column position
>
> ALTER TABLE weekly_sales
>
> MODIFY COLUMN month_number INT AFTER week_number;
>
> -- Update the month_number column
>
> UPDATE weekly_sales
>
> SET month_number = MONTH(week_date);

4.  Add a calendar_year column as the 4th column containing either 2018,
    2019 or 2020 values.

> -- Add the calendar_year column
>
> ALTER TABLE weekly_sales
>
> ADD calendar_year INT;
>
> -- Update the calendar_year column
>
> UPDATE weekly_sales
>
> SET calendar_year = YEAR(week_date);
>
> -- Update column position into 4th column position
>
> ALTER TABLE weekly_sales
>
> MODIFY COLUMN calendar_year INT AFTER month_number;

5.  Add a new column called age_band after the original segment column
    using the following mapping on the number inside the segment value:

<img src="media/image1.png" style="width:2.58356in;height:1.60847in"
alt="A black background with white text Description automatically generated" />

ALTER TABLE weekly_sales

ADD COLUMN age_band VARCHAR(255) AFTER segment;

UPDATE weekly_sales

SET age_band =

CASE

WHEN segment LIKE '%1%' THEN 'Young Adults'

WHEN segment LIKE '%2%' THEN 'Middle Aged'

WHEN segment LIKE '%3%' OR segment LIKE '%4%' THEN 'Retirees'

END

WHERE segment IS NOT NULL;

6.  Add a new demographic column using the following mapping for the
    first letter in the segment values:

<img src="media/image2.png" style="width:2.55022in;height:1.23344in"
alt="A black screen with white text Description automatically generated" />

ALTER TABLE weekly_sales

ADD COLUMN demographoc VARCHAR(255) AFTER segment;

update weekly_sales

SET demographic =

CASE

WHEN segment LIKE '%C%' THEN 'Couples'

WHEN segment LIKE '%F%' THEN 'Families'

END

WHERE segment IS NOT NULL;

7.  Ensure all null string values with an "unknown" string value in the
    original segment column as well as the
    new age_band and demographic columns

UPDATE weekly_sales

SET

segment = COALESCE(segment, 'unknown'),

age_band = COALESCE(age_band, 'unknown'),

demographic = COALESCE(demographic, 'unknown');

8.  Generate a new avg_transaction column as the sales value divided
    by transactions rounded to 2 decimal places for each record

> ALTER TABLE weekly_sales
>
> ADD avg_transaction DECIMAL(10, 2) after sales;
>
> UPDATE weekly_sales
>
> SET avg_transaction = ROUND(sales / transactions, 2);

Segment 1 Data Exploration:

1.  What day of the week is used for each week_date value?

> select count(week_date) as jumlah, dayname(week_date) as hari from
> weekly_sales group by hari;
>
> result : only monday

2.  What range of week numbers are missing from the dataset?

> select count(week_number) as jumlah, week_number from weekly_sales
> group by week_number;
>
> None

3.  How many total transactions were there for each year in the dataset?

> select sum(transactions) as total_transaksi , year(week_date) as tahun
> from weekly_sales group by tahun;
>
> '375813651', '2020'
>
> '365639285', '2019'
>
> '346406460', '2018'

4.  What is the total sales for each region for each month?

> select distinct region, monthname(week_date) as nama_bulan,
> count(sales) as jumlah from weekly_sales group by nama_bulan,region;
>
> region month total_sales

| ASIA          | August    | 442 |
|---------------|-----------|-----|
| USA           | August    | 442 |
| EUROPE        | August    | 439 |
| AFRICA        | August    | 442 |
| CANADA        | August    | 442 |
| OCEANIA       | August    | 442 |
| SOUTH AMERICA | August    | 440 |
| AFRICA        | July      | 476 |
| CANADA        | July      | 476 |
| USA           | July      | 476 |
| EUROPE        | July      | 475 |
| OCEANIA       | July      | 476 |
| SOUTH AMERICA | July      | 475 |
| ASIA          | July      | 476 |
| OCEANIA       | June      | 442 |
| USA           | June      | 442 |
| SOUTH AMERICA | June      | 442 |
| EUROPE        | June      | 441 |
| ASIA          | June      | 442 |
| CANADA        | June      | 442 |
| AFRICA        | June      | 442 |
| EUROPE        | May       | 406 |
| USA           | May       | 408 |
| SOUTH AMERICA | May       | 407 |
| OCEANIA       | May       | 408 |
| ASIA          | May       | 408 |
| AFRICA        | May       | 408 |
| CANADA        | May       | 408 |
| CANADA        | April     | 476 |
| SOUTH AMERICA | April     | 473 |
| EUROPE        | April     | 474 |
| ASIA          | April     | 476 |
| OCEANIA       | April     | 476 |
| AFRICA        | April     | 476 |
| USA           | April     | 476 |
| AFRICA        | March     | 136 |
| OCEANIA       | March     | 136 |
| EUROPE        | March     | 135 |
| SOUTH AMERICA | March     | 136 |
| USA           | March     | 136 |
| CANADA        | March     | 136 |
| ASIA          | March     | 136 |
| OCEANIA       | September | 68  |
| EUROPE        | September | 66  |
| ASIA          | September | 68  |
| SOUTH AMERICA | September | 68  |
| USA           | September | 68  |
| AFRICA        | September | 68  |
| CANADA        | September | 68  |

5.  What is the total count of transactions for each platform

> select sum(transactions) as jumlah_transaksi, platform from
> weekly_sales group by platform;

Transactions Platform

| 1081934227 | Retail  |
|------------|---------|
| 5925169    | Shopify |

6.  Which age_band and demographic values contribute the most to Retail
    sales?

> select demographic, age_band, sum(sales) from weekly_sales where
> platform = 'Retail' group by demographic,age_band order by sum(sales)
> asc limit 1;
>
> demographic age_band sales

| Families | Young Adults | 1770889293 |
|----------|--------------|------------|

7.  Can we use the avg_transaction column to find the average
    transaction size for each year for Retail vs Shopify? If not - how
    would you calculate it instead?

> select avg(avg_transaction) as rata_rata_transaksi, platform from
> weekly_sales group by platform;
>
> avg transaction platfom

<table>
<colgroup>
<col style="width: 52%" />
<col style="width: 47%" />
</colgroup>
<thead>
<tr class="header">
<th><blockquote>
<p>41.838224</p>
</blockquote></th>
<th><blockquote>
<p>Retail</p>
</blockquote></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><blockquote>
<p>180.226179</p>
</blockquote></td>
<td><blockquote>
<p>Shopify</p>
</blockquote></td>
</tr>
</tbody>
</table>

Segment 2 Before & After Analysis:

This technique is usually used when we inspect an important event and
want to inspect the impact before and after a certain point in time.

Taking the week_date value of 2020-06-15 as the baseline week where the
Data Mart sustainable packaging changes came into effect. We would
include all week_date values for 2020-06-15 as the start of the
period **after** the change and the previous week_date values would
be **before.**

Using this analysis approach - answer the following questions:

1.  What is the total sales for the 4 weeks before and after 2020-06-15?
    What is the growth or reduction rate in actual values and percentage
    of sales?

> SELECT DISTINCT week_number
>
> FROM clean_weekly_sales
>
> WHERE week_date = '2020-06-15'
>
> AND calendar_year = '2020';
>
> Result:
>
> WITH packaging_sales AS (
>
> SELECT
>
> week_date,
>
> week_number,
>
> SUM(sales) AS total_sales
>
> FROM clean_weekly_sales
>
> WHERE (week_number BETWEEN 21 AND 28)
>
> AND (calendar_year = 2020)
>
> GROUP BY week_date, week_number
>
> )
>
> , before_after_changes AS (
>
> SELECT
>
> SUM(CASE
>
> WHEN week_number BETWEEN 21 AND 24 THEN total_sales END) AS
> before_packaging_sales,
>
> SUM(CASE
>
> WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS
> after_packaging_sales
>
> FROM packaging_sales
>
> )
>
> SELECT
>
> after_packaging_sales - before_packaging_sales AS sales_variance,
>
> ROUND(100 \*
>
> (after_packaging_sales - before_packaging_sales)
>
> / before_packaging_sales,2) AS variance_percentage
>
> FROM before_after_changes;

2.  What about the entire 12 weeks before and after?

> WITH packaging_sales AS (
>
> SELECT
>
> week_date,
>
> week_number,
>
> SUM(sales) AS total_sales
>
> FROM clean_weekly_sales
>
> WHERE (week_number BETWEEN 13 AND 37)
>
> AND (calendar_year = 2020)
>
> GROUP BY week_date, week_number
>
> )
>
> , before_after_changes AS (
>
> SELECT
>
> SUM(CASE
>
> WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS
> before_packaging_sales,
>
> SUM(CASE
>
> WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) AS
> after_packaging_sales
>
> FROM packaging_sales
>
> )
>
> SELECT
>
> after_packaging_sales - before_packaging_sales AS sales_variance,
>
> ROUND(100 \*
>
> (after_packaging_sales - before_packaging_sales) /
> before_packaging_sales,2) AS variance_percentage
>
> FROM before_after_changes;

3.  How do the sale metrics for these 2 periods before and after compare
    with the previous years in 2018 and 2019?

> WITH changes AS (
>
> SELECT
>
> calendar_year,
>
> week_number,
>
> SUM(sales) AS total_sales
>
> FROM clean_weekly_sales
>
> WHERE week_number BETWEEN 21 AND 28
>
> GROUP BY calendar_year, week_number
>
> )
>
> , before_after_changes AS (
>
> SELECT
>
> calendar_year,
>
> SUM(CASE
>
> WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS
> before_packaging_sales,
>
> SUM(CASE
>
> WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS
> after_packaging_sales
>
> FROM changes
>
> GROUP BY calendar_year
>
> )
>
> SELECT
>
> calendar_year,
>
> after_packaging_sales - before_packaging_sales AS sales_variance,
>
> ROUND(100 \*
>
> (after_packaging_sales - before_packaging_sales)
>
> / before_packaging_sales,2) AS variance_percentage
>
> FROM before_after_changes;
