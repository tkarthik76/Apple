<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/f/fa/Apple_logo_black.svg" alt="Apple Logo" width="100"/>
</p>

<h1 align="center">📊 Apple Store & Product Analysis (2019 - Aug 2024)</h1>

<p align="center">
  <i>A business-driven analytical deep dive into Apple’s store performance, product sales trends, and warranty behavior.</i>
</p>

---

### 💼 Business Use Case

> **This project showcases how advanced data querying and analytics can be used to uncover critical business insights for Apple retail and product strategy — such as identifying underperforming stores, analyzing product and warranty trends, and optimizing sales forecasting — all through real-world, enterprise-scale questions.**

## Data Structure & Entity Relationship Diagram (ERD)
The Apple database comprises five tables: `dim_category`, `dim_product`, `fact_sales`, `fact_warranty`,`dim_store`.



![apple_erd](https://github.com/user-attachments/assets/66aff2bf-fda6-4fc9-b73a-b6ca72f34a65)

### Relationship between these tables are as follows:
- `dim_category[category_id]` to `dim_product[category_id]` **(one to one)**
 
- `dim_product[product_id]` to `fact_sales[product_id]` **(one to many)**
 
- `fact_sales[store_id]` to `dim_store[store_id]` **(many to one)**
  
- `fact_sales[sale_id]` to `fact_warranty[sale_id]` **(one to many)**


## 🚿 Data Cleaning 

### 🔧 Automated Error Handling in Power Query

**Objective:**  
Automatically replace errors across all columns (including future ones) with a specified value (e.g. `null` or `"ERROR"`), making my query resilient and scalable.

#### 🧹 M Code Steps

```m
// STEP 1: Create a dynamic list of {ColumnName, ReplacementValue} pairs
let
    ColumnReplacement = List.Transform(
        Table.ColumnNames(#"Changed Type"),
        each { _, null }  // Replace `null` with any custom error value like "ERROR"
    )
in
    ColumnReplacement

// STEP 2: Apply dynamic error replacement across all columns
= Table.ReplaceErrorValues(
    #"Changed Type",
    ColumnReplacement
)
```




### 1) What is the total number of units sold by each store?
-- Apple South Coast Plaza had sold the highest units of 47498.
-- Apple Sheffield had sold the lowest units of 647.



```
SELECT
    st.store_name AS store_name,
    SUM(sa.quantity) AS units_sold
FROM
    (SELECT store_id, store_name FROM dim_store ORDER BY store_id) st
JOIN
    (SELECT store_id, quantity FROM fact_sales ORDER BY store_id) sa
ON st.store_id = sa.store_id
GROUP BY 
    st.store_name
ORDER BY 
    units_sold DESC;
```


### 2)  Which stores did not make any sales in December 2023? 
--              Apple Leeds, Apple Bristol, Apple Cardiff

```
WITH store AS (
		SELECT
		store_id, store_name
		FROM 
		dim_store
		ORDER BY store_id
		),
store_with_sales AS  ( 
		SELECT
			store_id , SUM(quantity) AS units_sold
		FROM
			fact_sales
		WHERE
			fact_sales.sale_date >= '2023-12-01'
			AND fact_sales.sale_date < '2024-01-01'
		GROUP BY store_id
		ORDER BY store_id
		)
SELECT
    s.store_name, ss.units_sold
FROM
    store s 
LEFT JOIN 
    store_with_sales ss
    USING(store_id)
WHERE 
    units_sold IS NULL
GROUP BY 
    s.store_id;
```



### 3) How many stores sold a product but never had a warranty claim filed against any of their products?
-- 55

```
SELECT
    COUNT(DISTINCT(store_id)) AS total_stores_without_claim
FROM 
    fact_sales
WHERE 
    store_id NOT IN (
					SELECT
					    s.store_id
					FROM 
					    fact_sales s
					RIGHT JOIN 
                        fact_warranty w
					USING(sale_id)
					);
```

### 4) What percentage of warranty claims are marked as "Warranty Void"?
-- 23%


```
SELECT
    ROUND(((COUNT(CASE WHEN LOWER(repair_status) LIKE '%warranty void%' THEN 1 END) * 100) /
    COUNT(claim_id)),0) AS pct_of_warranty_void_claims
FROM fact_warranty;
```



### 5) Which store had the highest total units sold in the last year?

```
WITH last_year_stores_sales AS (
				SELECT
					ss.store_id, 
					s.store_name,
					SUM(ss.quantity) AS units_sold
				FROM 
				    fact_sales ss
				JOIN 
				    dim_store s 
				    ON ss.store_id=s.store_id
				    AND ss.sale_date>=(SELECT YEAR(MAX(sale_date))-1 FROM fact_sales) 
				GROUP BY 
                    ss.store_id,s.store_name
),
stores_rank AS (
		SELECT
			store_id,
			store_name,
			units_sold,
			DENSE_RANK() OVER(ORDER BY units_sold DESC) AS units_sold_rank
		FROM
		    last_year_stores_sales
)
SELECT
	store_name,
	units_sold
FROM
    stores_rank 
WHERE
    units_sold_rank=1;
```


### 6) Count the number of unique products sold in the last year.

```
SELECT
    COUNT(DISTINCT(product_id)) as num_of_unique_products
FROM 
    fact_sales 
WHERE
    sale_date>=(SELECT YEAR(MAX(sale_date))-1 FROM fact_sales) ;
```




### 7) What is the average price of products in each category?
--     among which, Laptop category had the highest avg price

```
SELECT
	p.category_id, 
	c.category_name,
	ROUND(AVG(p.price),0) AS avg_price
FROM
    dim_product p 
JOIN 
    dim_category c
    USING(category_id)
GROUP BY
	p.category_id,c.category_name
ORDER BY 
    avg_price DESC;
```



### 8)  Identify each store and best-selling day with day_type based on the highest quantity sold.


```
WITH  sales_day AS (
			SELECT
			store_id,
			DAYNAME(sale_date) AS day,
			SUM(quantity) AS units_sold,
			DENSE_RANK() OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS position
			FROM fact_sales
			GROUP BY store_id,day
) ,
best_selling_day AS (
			SELECT *
			FROM
			sales_day
			WHERE position=1
)

SELECT 
	s.store_name, 
	bs.day, 
	bs.units_sold, 
	s.city, 
	s.country,
	CASE WHEN day IN ('saturday','sunday') THEN 'weekend' ELSE 'weekday' END AS day_type
FROM
    dim_store s
INNER JOIN 
    best_selling_day bs
USING(store_id);
```


### 9) What percentage of stores has the best selling day recorded on a weekend?
--             33

```
WITH sales_day AS (
			SELECT
				store_id,
				DAYNAME(sale_date) AS day,
				SUM(quantity) AS units_sold,
				DENSE_RANK() OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS position
			FROM 
                fact_sales
			GROUP BY 
                store_id,day
),

best_selling_day AS (
			SELECT 
                store_id,
                day,
                units_sold
			FROM
			    sales_day
			WHERE 
                position=1
),


best_selling_day_weektype AS (
			SELECT 
				s.store_name, 
				bs.day, 
				bs.units_sold, 
				s.city, 
				s.country,
				CASE WHEN day IN ('saturday','sunday') THEN 'weekend' ELSE 'weekday' END AS week_type
			FROM
			    dim_store s
			INNER JOIN 
                best_selling_day bs
			USING(store_id)
)

SELECT 
	ROUND((COUNT(CASE WHEN week_type='weekend' THEN 1 END)*100
	/COUNT(store_name) ),0) AS pct_best_selling_day_on_weekend
FROM
    best_selling_day_weektype;
```


### 10)  Identify least-selling product of each country for each year based on the total units sold.

```
WITH paroduct_sales_per_country AS  (
		SELECT
			s.country,
			YEAR(ss.sale_date) AS year, 
			ss.product_id, 
			SUM(ss.quantity) AS total_quantity,
			DENSE_RANK() OVER(PARTITION BY s.country, YEAR(ss.sale_date) ORDER BY s.country,YEAR(ss.sale_date), SUM(ss.quantity)) AS rank_quantity
		FROM
			dim_store s
		JOIN
			fact_sales ss 
			ON s.store_id=ss.store_id
		GROUP BY 
			s.country,year,ss.product_id
) 
SELECT
	c.country,
	c.year,
	p.product_name,
	c.total_quantity
FROM
    paroduct_sales_per_country c
INNER JOIN 
    dim_product p 
    ON p.product_id=c.product_id
WHERE 
    rank_quantity=1;
```


### 11) How many warranty claims were filed within 180 days of a product sale?
-- 19907

-- et 3.91 sec
-- option a) Filtered at ON caluse itself, as this clause comes early in query execution plan
```
SELECT
    COUNT(w.claim_id) AS num_of_claims
FROM
    fact_warranty w
JOIN 
    fact_sales s 
    ON w.sale_id=s.sale_id AND DATEDIFF(w.claim_date,s.sale_date) <=180;
```



### 12. What product registered the highest warranty claims whithin 180 days of it's sale?
--   AirTag with 2764 claims

```
WITH claims_product_id AS (
				SELECT
					COUNT(w.claim_id) AS num_of_claims,
					s.product_id
				FROM
				    fact_warranty w
				INNER JOIN 
				    fact_sales s 
                    ON w.sale_id=s.sale_id AND DATEDIFF(w.claim_date,s.sale_date) <=180
				GROUP BY 
                    s.product_id
),
product AS (
	SELECT
		product_id,
		product_name
	FROM
	    dim_product
)
SELECT
	p.product_name,
	cp.num_of_claims
FROM 
    product p
JOIN
    claims_product_id cp 
    ON p.product_id=cp.product_id
ORDER BY 
    cp.num_of_claims DESC;
```


### 13) How many warranty claims have been filed for products launched in the last two years?
--     iphone 15 Pro Max 320
--     iphone 15         321
--     iphone 15Pro      290

```
WITH claim_product_id AS (
				SELECT
				s.product_id,
				w.claim_id
				FROM
				fact_warranty w
				LEFT JOIN 
				fact_sales s ON
				w.sale_id=s.sale_id 
)

SELECT
	p.product_name,
	COUNT(cp.claim_id) AS num_of_claims
FROM
    dim_product p
INNER JOIN
	claim_product_id cp 
    ON p.product_id=cp.product_id AND p.launch_date>=(SELECT YEAR(MAX(sale_date))-2 FROM fact_sales) 
GROUP BY 
    p.product_name;
```

### 14. How many warranty claims have been filed for products launched in the last two years? Also, state the sales as well.

```
WITH claim_product_id AS (
				SELECT
				s.product_id,S.sale_id,
				w.claim_id
				FROM
				fact_warranty w
				RIGHT JOIN 
				fact_sales s ON
				w.sale_id=s.sale_id 
)

SELECT
	p.product_name,
	COUNT(cp.claim_id) AS num_of_claims,
	COUNT(cp.sale_id) AS num_of_sales
FROM
    dim_product p
JOIN
    claim_product_id cp 
    ON p.product_id=cp.product_id AND
       p.launch_date>=(SELECT YEAR(MAX(sale_date))-2 FROM fact_sales) 
GROUP BY 
    p.product_name;
```



### 15)  List the months in the last 3 years where sales exceeded 5000 units from USA.

```
SELECT 
	ss.month_year, 
	SUM(ss.quantity) AS total_quantity
FROM
    (SELECT store_id,country FROM dim_store WHERE country='USA') s
LEFT JOIN 
	(SELECT
	DATE_FORMAT(sale_date, '%m-%Y') AS month_year,store_id,quantity
	FROM
	fact_sales
	WHERE 
    sale_date>=(SELECT YEAR(MAX(sale_date))-3 FROM fact_sales) 
    ) ss 
ON
    s.store_id=ss.store_id
GROUP BY 
    ss.month_year
HAVING 
    total_quantity>5000;
```





### 16) Which product category had the most warranty claims filed in the last 2 years?

```
WITH product_category_warranty_claims AS (
					SELECT 
						c.category_name,
						(COUNT(w.claim_id)) AS total_claims
					FROM
						(SELECT sale_id,claim_id FROM fact_warranty) w
					LEFT JOIN 
						(SELECT
						sale_id,product_id
						FROM
						fact_sales
						WHERE sale_date>=(SELECT YEAR(MAX(sale_date))-2 FROM fact_sales) 
						) ss 
						  ON w.sale_id=ss.sale_id
					INNER JOIN 
						(
						SELECT
						product_id,category_id FROM dim_product
						) p
						ON ss.product_id=p.product_id
					INNER JOIN
						(
						SELECT
						category_id,category_name FROM dim_category
						) c 
						ON p.category_id=c.category_id
					GROUP BY 
						c.category_id
),
rank_of_product_category AS (
					SELECT
					    category_name,
					    DENSE_RANK() OVER(ORDER BY total_claims DESC) AS claims_rank
					FROM
					    product_category_warranty_claims
)

SELECT
    category_name
FROM
    rank_of_product_category
WHERE
    claims_rank=1 ;

```



### 17) Determine the percentage chance of receiving claims after each purchase for each country.


```
SELECT
	s.country,
	CASE 
	   WHEN COUNT(w.claim_id)<>0 THEN COUNT(w.claim_id) * 100 /SUM(ss.quantity) ELSE NULL 
	END AS percentage_chance_receiving_claims
FROM
	( SELECT store_id,country FROM dim_store ) s
INNER JOIN 
    ( SELECT sale_id,store_id,quantity FROM fact_sales ) ss
    ON s.store_id=ss.store_id
LEFT JOIN 
    ( SELECT claim_id,sale_id FROM fact_warranty ) w
    ON ss.sale_id=w.sale_id
GROUP BY 
	s.country
ORDER BY 
    percentage_chance_receiving_claims DESC;

```




### 18) Analyze each store's yearly growth ratio.


```
WITH current_year_sales AS (
    SELECT
        s.store_name,
        YEAR(ss.sale_date) AS year,
        SUM(ss.quantity * p.price) AS current_year_sales
    FROM 
        (SELECT price, product_id FROM dim_product) p
    JOIN 
        (SELECT sale_date, store_id, product_id, quantity FROM fact_sales) ss 
    ON 
        p.product_id = ss.product_id
    JOIN 
        (SELECT store_id, store_name FROM dim_store) s 
    ON 
        s.store_id = ss.store_id
    GROUP BY 
        s.store_name, YEAR(ss.sale_date)
),
current_previous_year_sales AS (
    SELECT
        *,
        LAG(current_year_sales) OVER (PARTITION BY store_name ORDER BY year) AS previous_year_sales
    FROM
        current_year_sales
),
YOY_growth_pct AS (
    SELECT
        store_name,
        year,
        ROUND((current_year_sales - previous_year_sales) * 100 / previous_year_sales, 0) AS YOY_growth_pct
    FROM
        current_previous_year_sales
    WHERE 
        previous_year_sales IS NOT NULL 
        AND year <> YEAR(CURDATE())
)
SELECT
    store_name,
    SUM(CASE WHEN year = 2020 THEN YOY_growth_pct ELSE NULL END) AS YOY_growth_pct_2020,
    SUM(CASE WHEN year = 2021 THEN YOY_growth_pct ELSE NULL END) AS YOY_growth_pct_2021,
    SUM(CASE WHEN year = 2022 THEN YOY_growth_pct ELSE NULL END) AS YOY_growth_pct_2022,
    SUM(CASE WHEN year = 2023 THEN YOY_growth_pct ELSE NULL END) AS YOY_growth_pct_2023,
    SUM(CASE WHEN year = 2024 THEN YOY_growth_pct ELSE NULL END) AS YOY_growth_pct_2024
FROM
    YOY_growth_pct
    GROUP BY store_name;
```


### 19)  What is the correlation between product price and warranty claims for products sold in the

--      Expensive is having a negative co-relation, Less Expensive has positive co-relation, Moderately Expensive has zero co-relation


```
SELECT
	CASE 
		WHEN p.price<500 THEN 'Less Expensive'
		WHEN p.price BETWEEN 500 AND 1000 THEN 'Moderately Expensive'
		ELSE 'Expensive'
	END AS price_segments,
	COUNT(w.claim_id) AS toal_claims    
FROM
    (SELECT claim_id,claim_date,sale_id FROM fact_warranty) w
LEFT JOIN
    (SELECT sale_id,sale_date,product_id FROM fact_sales WHERE sale_date>(SELECT YEAR(MAX(sale_date))-5 FROM fact_sales) ) ss 
    ON w.sale_id=ss.sale_id AND
       w.claim_date>ss.sale_date
INNER JOIN 
    (SELECT product_id, price FROM dim_product) p 
    ON ss.product_id=p.product_id
GROUP BY 
    price_segments;
```




### 20) Identify the store with the highest percentage of "Paid Repaired" claims in relation to total
--     claims filed.

```
WITH paid_repaired_claims AS (
				SELECT
					s.store_name,
					COUNT(w.claim_id) AS paid_repaired_claims
				FROM (SELECT claim_id,sale_id FROM fact_warranty WHERE repair_status LIKE '%Paid Repaired%') w
				LEFT JOIN 
				     (SELECT sale_id,store_id FROM fact_sales ) ss 
                     ON w.sale_id = ss.sale_id
				INNER JOIN 
				     (SELECT store_id,store_name FROM dim_store) s ON ss.store_id = s.store_id
				GROUP BY 
                     s.store_name
) ,
total_claims AS (
				SELECT
					s.store_name,
					COUNT(w.claim_id) AS total_claims
				FROM fact_warranty w
				LEFT JOIN fact_sales ss 
                     ON w.sale_id = ss.sale_id
				INNER JOIN dim_store s 
                     ON ss.store_id = s.store_id
				GROUP BY 
                     s.store_name
), 
pct_paid_repaired_claims_per_store AS (
				SELECT
					prc.store_name,
					ROUND(prc.paid_repaired_claims*100/tc.total_claims,0) AS pct_paid_repaired_claims
				FROM
					paid_repaired_claims prc
				INNER JOIN 
					total_claims tc  
					ON prc.store_name=tc.store_name
				GROUP BY 
					prc.store_name
),
store_ranking AS (
				SELECT
					store_name,
					DENSE_RANK() OVER(ORDER BY pct_paid_repaired_claims DESC) AS store_rank
				FROM
					pct_paid_repaired_claims_per_store
				)

SELECT
   store_name
FROM
   store_ranking
WHERE
   store_rank=1

;
```



### 21) Calculate the monthly running total sales for each store over the past
--     four years and compare the trends across this period?


```
WITH monthly_sales_each_store AS (
				SELECT
					s.store_name,
					s.store_id,
					YEAR(ss.sale_date) AS year,
					MONTH(ss.sale_date) AS month,
					SUM(ss.quantity*p.price) AS sales
				FROM
				    (SELECT product_id,price FROM dim_product)p
				RIGHT JOIN
				    (SELECT store_id,product_id,quantity,sale_date FROM fact_sales)ss 
                    ON p.product_id=ss.product_id
				INNER JOIN
				    (SELECT store_id,store_name FROM dim_store)s 
                    ON ss.store_id=s.store_id
				GROUP BY 
                    s.store_id,year,month
)

SELECT
	store_name,
	year,
	month,
	sales,
	SUM(sales) OVER(PARTITION BY store_id ORDER BY store_id, year,month ASC) AS aggregated_monthly_sales
FROM 
    monthly_sales_each_store
;

```

    
### 22) Analyze sales trends of product over time, segmented into key time periods: from launch to 6
--     months, 6-12 months, 12-18 months, and beyond 18 months


```
WITH sales_trend AS (
			SELECT
				p.product_name,
				CASE
					WHEN s.sale_date > p.launch_date AND 
                         s.sale_date <= DATE_ADD(p.launch_date, INTERVAL 6 MONTH) 
					THEN '0-6 Months'
					WHEN s.sale_date > DATE_ADD(p.launch_date, INTERVAL 6 MONTH) AND 
                         s.sale_date <= DATE_ADD(p.launch_date, INTERVAL 12 MONTH) 
					THEN '6-12 Months'
					WHEN s.sale_date > DATE_ADD(p.launch_date, INTERVAL 12 MONTH) AND 
                         s.sale_date <= DATE_ADD(p.launch_date, INTERVAL 18 MONTH) 
					THEN '12-18 Months'
					WHEN s.sale_date > DATE_ADD(p.launch_date, INTERVAL 18 MONTH) 
                    THEN '18+ Months'
				END AS sales_period,
				SUM(p.price * s.quantity) AS sales
			FROM
				(SELECT product_id, product_name, launch_date, price FROM dim_product) p
			JOIN
				(SELECT sale_date, product_id, quantity FROM fact_sales) s
			ON
				p.product_id = s.product_id
			WHERE 
				s.sale_date>p.launch_date
			GROUP BY
				p.product_name,
				sales_period
)

SELECT
    product_name,
    sales_period,
    sales
FROM
    sales_trend
ORDER BY
    product_name,
    FIELD(sales_period, '0-6 Months', '6-12 Months', '12-18 Months', '18+ Months');
```





### 23) Advanced Warranty Claim Trends: Analyze trends in warranty claims compared to product sales over the past two years from the last sales date available.

-- MAX aggregate function cannot be used in where clause and case function without using group by
-- product_sales increased and pct_warranty_claims halved indicating product improvement 

```
WITH date_ranges AS (
	  SELECT 
		MAX(s.sale_date) AS latest_sale_date,
		DATE_ADD(MAX(s.sale_date), INTERVAL -1 YEAR) AS last_year_start_date,
		DATE_ADD(MAX(s.sale_date), INTERVAL -2 YEAR) AS second_last_year_start_date
	  FROM 
		fact_sales s
),
last_two_years_sales_and_warranty_trend AS (
	SELECT
	  CASE 
		WHEN s.sale_date > d.last_year_start_date THEN 'last_year'
		WHEN s.sale_date > d.second_last_year_start_date AND s.sale_date <= d.last_year_start_date THEN 'second_last_year'
	  END AS TREND,
	  SUM(s.quantity) AS product_sales,
	  COUNT(w.claim_id) AS warranty_claims
	FROM 
	  fact_sales s
	LEFT JOIN 
	  fact_warranty w ON s.sale_id = w.sale_id
	CROSS JOIN 
	  date_ranges d
	WHERE 
	  s.sale_date > d.second_last_year_start_date AND 
      s.sale_date <= d.latest_sale_date
	GROUP BY 
	  TREND
)
SELECT
	 TREND,
     product_id,
	 product_sales,
	 warranty_claims,
	 ROUND(warranty_claims*100.0/COALESCE(NULLIF(product_sales,0),1),2) AS pct_warranty_claims
FROM
     last_two_years_sales_and_warranty_trend;
```     
     

### 24) product performance by average_pct_growth 

```
WITH product_units_sold AS (
				SELECT
					p.product_id,
					YEAR(sale_date) AS year,
					SUM(quantity) AS units_sold
				FROM
				    dim_product p
				LEFT JOIN
				    fact_sales s
				    USING(product_id)
				WHERE YEAR(sale_date)<>(SELECT YEAR(MAX(sale_date)) FROM fact_sales )
				GROUP BY 
				    p.product_id,YEAR(sale_date)
),
units_sold_comparison AS (
				SELECT
				product_id,
				year,
				units_sold,
				LAG(units_sold) OVER (PARTITION BY product_id) AS last_year_units_sold
				FROM
				product_units_sold
),
growth_units_sold AS  (
				SELECT 
				product_id,
				year,
				(units_sold-last_year_units_sold)*100.0/last_year_units_sold AS pct_growth_units_sold
				FROM
				units_sold_comparison
)

SELECT
	product_id,
	ROUND(AVG(pct_growth_units_sold),2)  AS avg_growth_units_sold
FROM
    growth_units_sold
GROUP BY
    product_id
ORDER BY 
    avg_growth_units_sold DESC;
```


### 25) Product Performance Analysis: Analyze sales performance over five years and identify the top 10 products with the highest percentage growth in units sold.


```
WITH sales_by_year AS (
		  SELECT
			product_id,
			YEAR(sale_date) AS sale_year,
			SUM(quantity) AS total_units
		  FROM
			fact_sales
		  WHERE
			YEAR(sale_date) <> (SELECT YEAR(MAX(sale_date)) FROM fact_sales)
		  GROUP BY
			product_id, YEAR(sale_date)
),
min_max_years AS (
		  SELECT
			product_id,
			MIN(sale_year) AS start_year,
			MAX(sale_year) AS end_year
		  FROM
			sales_by_year
		  GROUP BY
			product_id
),
start_end_units AS (
		  SELECT
			s.product_id,
			m.start_year,
			m.end_year,
			MAX(CASE WHEN s.sale_year = m.start_year THEN s.total_units END) AS start_units,
			MAX(CASE WHEN s.sale_year = m.end_year THEN s.total_units END) AS end_units
		  FROM
			sales_by_year s
		  JOIN
			min_max_years m ON s.product_id = m.product_id
		  GROUP BY
			s.product_id, m.start_year, m.end_year
),
growth_calc AS (
		  SELECT
			product_id,
			start_units,
			end_units,
			ROUND((end_units - start_units) * 100.0 /
            COALESCE(NULLIF(start_units, 0),1), 2) AS pct_growth
		  FROM
			start_end_units
		  WHERE
			start_units IS NOT NULL AND end_units IS NOT NULL
)

SELECT 
		product_id,
	    start_units,
		end_units,
        pct_growth
FROM 
        growth_calc
ORDER BY 
        pct_growth DESC
;

```

### 26) Product Lifetime Value per year

```
WITH product_sales_summary AS (
			  SELECT
				product_id,
				MIN(sale_date) AS first_sale_date,
				MAX(sale_date) AS last_sale_date,
				COUNT(DISTINCT sale_id) AS total_purchases,
				SUM(quantity * price) AS total_revenue,
				COUNT(DISTINCT YEAR(sale_date)) AS active_years
			  FROM
				fact_sales
			  LEFT JOIN 
				dim_product
				USING(product_id)
			  GROUP BY
				product_id
),
product_ltv_calc AS (
			  SELECT
				product_id,
				total_purchases,
				total_revenue,
				DATEDIFF(last_sale_date, first_sale_date) / 365.0 AS lifespan_years,
				ROUND(total_revenue / COALESCE(NULLIF(total_purchases, 0),1), 2) AS avg_order_value,
				ROUND(total_purchases / 
                      COALESCE(NULLIF(DATEDIFF(last_sale_date, first_sale_date) / 
                               365.0, 0),1), 
					  2) AS purchase_freq_per_year
			  FROM
				product_sales_summary
),
final_calc AS (
			  SELECT
				product_id,
				total_revenue,
				lifespan_years,
				avg_order_value,
				purchase_freq_per_year,
				ROUND(purchase_freq_per_year * avg_order_value * lifespan_years, 2) AS product_lifetime_value
			  FROM
				product_ltv_calc
)

SELECT 
   product_id,
   total_revenue,
   lifespan_years,
   avg_order_value,
   purchase_freq_per_year,
   product_lifetime_value
FROM 
   final_calc
ORDER BY 
   product_lifetime_value DESC
LIMIT 10;
```




### 27) Product Lifetime Value per MONTH


```
WITH product_sales_summary AS (
				  SELECT
					product_id,
					MIN(sale_date) AS first_sale_date,
					MAX(sale_date) AS last_sale_date,
					COUNT(DISTINCT sale_id) AS total_purchases,
					SUM(quantity * price) AS total_revenue
				  FROM
					fact_sales
				  LEFT JOIN
					dim_product
					USING(product_id)
				  GROUP BY
					product_id
),
product_ltv_calc AS (
				  SELECT
					product_id,
					total_purchases,
					total_revenue,
					TIMESTAMPDIFF(MONTH, first_sale_date, last_sale_date) AS lifespan_months,
					ROUND(total_revenue / COALESCE(NULLIF(total_purchases, 0),1), 2) AS avg_order_value,
					ROUND(total_purchases /
                          COALESCE(NULLIF(TIMESTAMPDIFF(MONTH, first_sale_date, last_sale_date), 0),1), 
                          2
                          ) AS purchase_freq_per_month
				  FROM
					product_sales_summary
),
final_calc AS (
	  SELECT
		product_id,
		total_revenue,
		lifespan_months,
		avg_order_value,
		purchase_freq_per_month,
		ROUND(purchase_freq_per_month * avg_order_value * lifespan_months, 2) AS product_lifetime_value
	  FROM
		product_ltv_calc
)

SELECT 
   product_id,
   total_revenue,
   lifespan_months,
   avg_order_value,
   purchase_freq_per_month,
   product_lifetime_value
FROM 
   final_calc
ORDER BY 
   product_lifetime_value DESC
LIMIT 10;
```




  
### 28) Market Basket Analysis: Perform a market basket analysis to find products frequently purchased together.


-- P-40 & P-49 were bought together more frequently and P-23 & P-9 less frequently


CREATE INDEX idx_sale_date_product_id ON fact_sales(sale_date, product_id);


```
WITH limited_sales AS (
		  SELECT sale_date,product_id
		  FROM fact_sales
		  ORDER BY sale_date
  
),
product_pairs AS (
		  SELECT
			a.sale_date,
			a.product_id AS product_a,
			b.product_id AS product_b
		  FROM
			limited_sales a
		  JOIN
			limited_sales b
			ON a.sale_date = b.sale_date AND a.product_id < b.product_id
)

SELECT
  product_a,
  product_b,
  COUNT(*) AS times_bought_together
FROM
  product_pairs
GROUP BY
  product_a, product_b
ORDER BY
  times_bought_together DESC;
 ``` 
  
  
### 29) Warranty Claim Rate per product

-- iPad Air (4th Gen) has the highest warranty claim rate

```
SELECT
	product_name,
	COUNT(claim_id)*100.0/COUNT(fact_sales.sale_id) AS warranty_claim_rate
FROM
   dim_product
INNER JOIN 
   fact_sales 
   USING(product_id)
LEFT JOIN
   fact_warranty
   USING(sale_id)
GROUP BY 
   product_name
ORDER BY 
   warranty_claim_rate DESC;
```  
   

### 30) Show store-product pairs with consistent sales every month in 2023.

*/

-- no store, product pairs have any consistent sales every month in 2023

```
WITH store_sales AS (
SELECT 
	Store_id,
	Product_id,
    sale_date,
    YEAR(Sale_date) AS year	
FROM 
	fact_sales
),
store_sales_2023 AS (
SELECT 
	Store_id,
	Product_id,
    sale_date
FROM 
	store_sales
WHERE
    year=2023
)

SELECT
	store_id,
	product_id,
    COUNT(DISTINCT MONTH(Sale_date)) AS num_monthly_sales
FROM
    store_sales_2023 
GROUP BY 
    Store_id, Product_id
HAVING 
     num_monthly_sales = 12
;
```


### 31) Compute the lag in days between each sale and its corresponding warranty claim (if any), and rank the top 5 fastest claims. 

```
WITH warranty_claim_sales AS ( 
			SELECT
			fact_sales.sale_id,
			DATEDIFF(claim_date,sale_date) AS lag_in_days_for_warranty_claim
			FROM
			fact_sales
			INNER JOIN
			fact_warranty
			USING(sale_id)
),
ranking AS (
			SELECT
			sale_id,
			lag_in_days_for_warranty_claim,
			DENSE_RANK() OVER(ORDER BY lag_in_days_for_warranty_claim ASC) AS ranking
			FROM
			 warranty_claim_sales 
)

SELECT
	sale_id,
	lag_in_days_for_warranty_claim
FROM
    ranking
WHERE
    ranking<=5;
``` 



### 32) Identify products that haven’t been sold in the past year but had warranty claims. 
-- iphone X, ipad (6th Gen), Apple Watch Series 4, iphone XS

```
WITH ProductsWithSalesLastYear AS (
			SELECT DISTINCT product_id
			FROM fact_sales
			WHERE YEAR(sale_date) = (SELECT YEAR(MAX(sale_date))-1 FROM fact_sales) 
),
ProductsWithWarrantyClaims AS (
			SELECT DISTINCT product_id
			FROM fact_warranty
			INNER JOIN fact_sales USING(sale_id) -- Link warranty claim to sale
)

SELECT 
   DISTINCT p.product_name
FROM 
   dim_product p
INNER JOIN 
   ProductsWithWarrantyClaims w ON p.product_id = w.product_id
LEFT JOIN 
   ProductsWithSalesLastYear s ON p.product_id = s.product_id
WHERE 
   s.product_id IS NULL;
```



### 33) Calculate the average time to warranty claim per category and rank them */

```
WITH category_analysis AS (
			SELECT
				category_name,
				AVG(DATEDIFF(claim_date,sale_date)) AS avg_no_days_to_warranty_claim
			FROM
			    fact_sales
			LEFT JOIN
			    fact_warranty
			    USING(sale_id)
			INNER JOIN
			    dim_product
			    ON fact_sales.product_id=dim_product.product_id
			INNER JOIN
			    dim_category
			    ON dim_product.category_id=dim_category.category_id
			GROUP BY
			    category_name
			ORDER BY
			    avg_no_days_to_warranty_claim DESC
)

SELECT
	category_name,
	avg_no_days_to_warranty_claim,
	DENSE_RANK() OVER(ORDER BY avg_no_days_to_warranty_claim ASC) AS ranking
FROM
    category_analysis ;
```





