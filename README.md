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


### 📊 Use Case Summary Table

| Use Case # | Focus Area                        | 💼 Business Problem Solved                        | 🌟 Strategic Benefit                                | Related Queries |
|------------|-----------------------------------|--------------------------------------------------|-----------------------------------------------------|----------------|
| 1          | Sales & Inventory Optimization    | Inaccurate demand planning                       | Improved stock planning and marketing               | [Q22](#22-analyze-sales-trends-of-product-over-time-segmented-into-key-time-periods-from-launch-to-6-months-6-12-months-12-18-months-and-beyond-18-months), [Q23](#23-advanced-warranty-claim-trends-analyze-trends-in-warranty-claims-compared-to-product-sales-over-the-past-two-years-from-the-last-sales-date-available)       |
| 2          | Product Lifecycle Strategy        | Suboptimal timing of product launches/EOL        | Smarter launches and end-of-life decisions          | [Q22](#22-analyze-sales-trends-of-product-over-time-segmented-into-key-time-periods-from-launch-to-6-months-6-12-months-12-18-months-and-beyond-18-months), [Q24](#24-product-performance-by-average_pct_growth),[Q25](#25-product-performance-analysis-analyze-sales-performance-over-five-years-and-identify-the-top-10-products-with-the-highest-percentage-growth-in-units-sold),[Q26](#26-product-lifetime-value-per-year),[Q27](#27-product-lifetime-value-per-month)   |
| 3          | Warranty & Customer Support       | High service costs and poor claim insights       | Reduced warranty costs and improved satisfaction    | [Q4](#4-what-percentage-of-warranty-claims-are-marked-as-warranty-void), [Q11](#11-how-many-warranty-claims-were-filed-within-180-days-of-a-product-sale),[Q12](#12-what-product-registered-the-highest-warranty-claims-whithin-180-days-of-its-sale),[Q13](#13-how-many-warranty-claims-have-been-filed-for-products-launched-in-the-last-two-years),[Q14](#14-how-many-warranty-claims-have-been-filed-for-products-launched-in-the-last-two-years-also-state-the-sales-as-well), [Q16](#16-which-product-category-had-the-most-warranty-claims-filed-in-the-last-2-years), [Q29](#29-warranty-claim-rate-per-product), [Q31](#31-compute-the-lag-in-days-between-each-sale-and-its-corresponding-warranty-claim-if-any-and-rank-the-top-5-fastest-claims), [Q33](#33-calculate-the-average-time-to-warranty-claim-per-category-and-rank-them) |
| 4          | Market Segmentation               | One-size-fits-all promotions                     | More targeted regional marketing                    | [Q10](#10--identify-the-least-selling-product-of-each-country-for-each-year-based-on-the-total-units-sold), [Q15](#15--list-the-months-in-the-last-3-years-where-sales-exceeded-5000-units-from-usa), [Q17](#17-determine-the-percentage-chance-of-receiving-claims-after-each-purchase-for-each-country), [Q30](#30-show-store-product-pairs-with-consistent-sales-every-month-in-2023) |
| 5          | Store Operational Efficiency      | Inconsistent performance across stores           | Operational benchmarking and resource optimization  | [Q8](#8--identify-each-store-and-best-selling-day-with-day_type-based-on-the-highest-quantity-sold),[Q9](#9-what-percentage-of-stores-has-the-best-selling-day-recorded-on-a-weekend), [Q18](#18-analyze-each-stores-yearly-growth-ratio), [Q21](#21-calculate-the-monthly-running-total-sales-for-each-store-over-the-past-four-years-and-compare-the-trends-across-this-period) |
| 6          | Pricing & Profitability           | Pricing misalignment with customer value         | Better margin control and competitive pricing       | [Q7](#7-what-is-the-average-price-of-products-in-each-category), [Q19](#19--what-is-the-correlation-between-product-price-and-warranty-claims-for-products-sold-in-the-last-five-years)        |
| 7          | Warranty Risk Prediction          | Reactive service model                           | Brand trust and proactive service design            | [Q12](#12-what-product-registered-the-highest-warranty-claims-whithin-180-days-of-its-sale), [Q19](#19--what-is-the-correlation-between-product-price-and-warranty-claims-for-products-sold-in-the-last-five-years), [Q29](#29-warranty-claim-rate-per-product), [Q31](#31-compute-the-lag-in-days-between-each-sale-and-its-corresponding-warranty-claim-if-any-and-rank-the-top-5-fastest-claims), [Q33](#33-calculate-the-average-time-to-warranty-claim-per-category-and-rank-them)|
| 8          | Product Bundling/Upselling        | Missed cross-sell opportunities                  | Boost in AOV and customer retention                 | [Q28](#28-market-basket-analysis-perform-a-market-basket-analysis-to-find-products-frequently-purchased-together)           |
| 9          | Sales Trend Tracking              | Inability to react to sales shifts quickly       | Quicker marketing or stocking adjustments           | [Q21](#21-calculate-the-monthly-running-total-sales-for-each-store-over-the-past-four-years-and-compare-the-trends-across-this-period),[Q22](#22-analyze-sales-trends-of-product-over-time-segmented-into-key-time-periods-from-launch-to-6-months-6-12-months-12-18-months-and-beyond-18-months)   |
| 10         | Product Gap/Opportunity Detection | Ignoring unloved but service-heavy products      | Data-driven relaunch or discontinuation decisions   | [Q32](#32-identify-products-that-havent-been-sold-in-the-past-year-but-had-warranty-claims)            |




### 1) What is the total number of units sold by each store?
-- Apple South Coast Plaza had sold the highest units of 47498.
-- Apple Sheffield had sold the lowest units of 647.

![1](https://github.com/user-attachments/assets/7ceb9edc-e238-40a9-a9ed-5a55fc224fb5)

#### 💼 Business Use Case:
Measure store-wise product movement to optimize resource allocation and sales performance tracking.

#### 🧩 Business Problem Solved:
Reveals which stores contribute the most or least to overall sales volume.

Helps identify underperforming or overperforming locations for operational benchmarking.

Informs staffing, marketing, and stock replenishment decisions at the store level.

#### 🌟 Strategic Benefit:
Enables data-driven store performance management and investment prioritization across regions.


### 2)  Which stores did not make any sales in December 2023? 
--              Apple Leeds, Apple Bristol, Apple Cardiff

![2](https://github.com/user-attachments/assets/97b55bed-627f-4d85-9841-21f74d72b7ac)

#### 🏪 Store Operational Efficiency
#### Business Use Case:
Identify underperforming or inactive stores during a specific period (e.g., December 2023).

#### 💼 Business Problem Solved:

Surfaces stores with zero sales activity, signaling potential operational or logistical issues.

Enables targeted follow-up: staffing issues, supply chain gaps, or store closures.

Supports decisions on resource reallocation, store audits, or promotional support.

#### 🌟 Strategic Benefit:
Improves store network productivity by detecting inefficiencies early and enabling prompt corrective action.


### 3) How many stores sold a product but never had a warranty claim filed against any of their products?
-- 55
![3](https://github.com/user-attachments/assets/e4ca6085-0860-4a56-a95b-183d8d55c667)


#### 🏪 Store Sales Quality
#### 💼 Business Problem Solved:

Highlights stores with consistently reliable customer experiences.

Identifies regions with potentially better handling, customer education, or sales practices.

Flags discrepancies where poor post-sales tracking or underreporting may exist.

#### 🌟 Strategic Benefit:

Supports targeted knowledge sharing or training from high-performing stores.

Enhances post-sales service monitoring and warranty cost forecasting.

Improves brand trust by identifying and replicating high-quality retail practices.



### 4) What percentage of warranty claims are marked as "Warranty Void"?
-- 23%

![4](https://github.com/user-attachments/assets/f190bb3e-2f76-4121-bf99-fa6a186b5d9c)


#### 🛡️ Warranty Analytics & Policy Insights
Query: What percentage of warranty claims are marked as "Warranty Void"?

#### 💼 Business Use Case:
Assess Warranty Claim Denial Trends to Enhance Transparency and Trust

#### 🔍 Business Problem Solved:

Identifies how often customer claims are denied due to policy limitations.

Highlights potential mismatch between customer expectations and warranty coverage.

Flags stores, regions, or products with unusually high void rates.

#### 🌟 Strategic Benefit:

Supports clearer warranty policy communication.

Helps refine training, product quality control, or eligibility criteria.

Strengthens customer experience and post-sales credibility.


### 5) Which store had the highest total units sold in the last year?

Apple South Coast Plaza	with 47498 units sold.

![5](https://github.com/user-attachments/assets/aec9274d-22db-4118-9346-71ae8c80ca7f)

#### 🏆 Use Case: Top Performing Stores
#### 💼 Business Problem Solved:

Identifies which store drives the highest volume, informing best practices and potential expansion strategies.

Helps benchmark store performance and identify outliers (top and underperformers).

Supports resource reallocation—staffing, marketing, inventory—to capitalize on high-performing stores.

#### 🌟 Strategic Benefit:

Enables data-driven decision making for location-based investments.

Promotes internal knowledge sharing from top performers.

Aligns with corporate growth and operational efficiency strategies.

### 6) Count the number of unique products sold in the last year.

64
![Slide6](https://github.com/user-attachments/assets/e0bdedb9-88fe-4f17-931e-329da6ea74d6)

#### 🏬 Store Performance Benchmarking
Use Case: Count the number of unique products sold in the last year.

#### 💼 Business Problem Solved:

Identifies product diversity sold at retail.

Highlights underperforming stores or regions with low product variety.

Assists in planning for restocking or new product introductions.

#### 🌟 Strategic Benefit:

Enables SKU rationalization and smarter assortment planning.

Drives decisions on localized merchandising and promotions.

Supports performance-linked compensation and store-level goals.



### 7) What is the average price of products in each category?
--     among which, Laptop category had the highest avg price

![Slide7](https://github.com/user-attachments/assets/ad446f99-f8b7-4a61-add1-9216bdb55ea4)

#### 💰 Product Pricing Strategy
Use Case: Calculate the average price of products in each category.

#### 💼 Business Problem Solved:

Enables understanding of price distribution across categories.

Highlights pricing gaps or inconsistencies within product lines.

Supports competitive analysis and price positioning.

#### 🌟 Strategic Benefit:

Helps in pricing strategy optimization, ensuring competitive but profitable pricing.

Assists in category management, identifying areas for price adjustment.

Guides promotional discounting decisions and margin protection.


### 8)  Identify each store and best-selling day with day_type based on the highest quantity sold.

![Slide8](https://github.com/user-attachments/assets/d1b26a5c-9242-47b5-a9fa-d25643bf94fa)

#### 🏬 Store Sales Optimization
Use Case: Identify the best-selling day for each store and the corresponding day type based on the highest quantity sold.

#### 💼 Business Problem Solved:

Helps pinpoint optimal sales days for each store.

Enables targeted promotional efforts and scheduling.

Optimizes staffing and resource allocation on peak sales days.

#### 🌟 Strategic Benefit:

Drives improved store operations and performance.

Provides actionable insights for marketing campaigns and inventory management.



### 9) What percentage of stores has the best selling day recorded on a weekend?
--             33
![Slide10](https://github.com/user-attachments/assets/da864f8b-db8f-4c39-bee7-0e0d99394ea4)

#### 🏬 Store Sales Optimization
#### 💼 Business Problem Solved:

📊 Identifies stores with the highest sales on weekends.

📈 Helps allocate resources like staff and inventory efficiently.

🛒 Ensures products are stocked and staff is ready for high-demand days.

#### 🌟 Strategic Benefit:

🏆 Boosts sales potential by focusing resources where needed most.

⏱️ Improves operational efficiency and enhances customer experience on peak days.



### 10)  Identify the least-selling product of each country for each year based on the total units sold.

![Slide10](https://github.com/user-attachments/assets/ef1ef2ab-28fd-4114-a667-823ea7c07215)

#### Business Use Case: 🔍 Analyze least-selling products per country per year to optimize product offerings.

#### 💼 Business Problem Solved:

📉 Identifies underperforming products in specific regions.

🛑 Helps phase out low-demand products.

📊 Enables better inventory allocation by focusing on higher-demand items.

#### 🌟 Strategic Benefit:

🏆 Improves sales by removing non-performing products.

📦 Optimizes inventory and resource management.

🌍 Tailors product offerings to regional preferences.


### 11) How many warranty claims were filed within 180 days of a product sale?
-- 19907

![Slide11](https://github.com/user-attachments/assets/fd75b687-d596-417a-877b-9b8dadc1b91e)

#### Business Use Case: 🔍 Monitor post-sale warranty claims within the first 180 days to track early defects.

#### 💼 Business Problem Solved:

🛠️ Identifies products with early reliability issues.

📅 Helps pinpoint manufacturing or design flaws.

🚚 Assists in improving product quality and post-sale service.

#### 🌟 Strategic Benefit:

📈 Enhances product quality and brand trust.

🔧 Minimizes warranty costs by addressing root causes early.

🛒 Strengthens customer retention and satisfaction.




### 12. What product registered the highest warranty claims whithin 180 days of it's sale?
--   AirTag with 2764 claims

![Slide12](https://github.com/user-attachments/assets/330eca6c-20dd-431f-9c5d-732292a10092)

#### Business Use Case: 🚨 Product Quality Monitoring through Warranty Claims
#### 💼 Business Problem Solved:

🛠️ Highlights products with potential manufacturing or quality issues early on.

📉 Helps in identifying patterns of defects that may lead to recalls or improvements.

🧰 Guides post-sale support and customer satisfaction strategies.

#### 🌟 Strategic Benefit:

📊 Improves product development and quality control.

🏷️ Reduces warranty-related costs.

📈 Enhances customer trust and brand loyalty by addressing issues promptly.

### 13) How many warranty claims have been filed for products launched in the last two years?
-- AirTag with the highest claims of 5097 
-- AirPods Max with the lowest claims of 32
![Slide13](https://github.com/user-attachments/assets/3263b889-4b24-43ec-944a-1bbadf835eb8)

#### Business Use Case: 📉 Monitoring Warranty Claims for New Products

#### 💼 Business Problem Solved:

Tracks warranty claims for products launched within the last two years.

Identifies potential product quality issues early in the product lifecycle.

Helps prioritize improvement actions for products with high claim frequencies.

#### 🌟 Strategic Benefit:

Enables proactive quality control and risk mitigation strategies.

Protects brand reputation by addressing issues promptly.

Supports data-driven decisions for future product development.

### 14. How many warranty claims have been filed for products launched in the last two years? Also, state the sales as well.

![Slide14](https://github.com/user-attachments/assets/5e11edf3-ccc6-4c04-a824-66944feb894f)

Business Use Case same as 13



### 15)  List the months in the last 3 years where sales exceeded 5000 units from USA.
-- All months of 2019, 20220, 
-- oct, nov, and dec of 2021, and 
-- mar,apr, oct to dec of 2022 and 
no months of 2023 and 2024 

![Slide15](https://github.com/user-attachments/assets/3839e8c6-1131-4bf4-9c24-acdc7da1642e)

#### 🔍 Business Use Case:
Seasonality & Market Readiness Analysis
“Identify peak demand periods in specific regions to inform promotional campaigns, staffing, and inventory planning.”

#### 💼 Business Problem Solved:

Pinpoints months of peak sales activity in the U.S. market.

Helps determine when demand surges, enabling better stock and supply chain planning.

Informs marketing timing (e.g., product pushes or holiday promotions) with factual sales data.

#### 🌟 Strategic Benefit:

Enables seasonal sales forecasting and campaign scheduling based on historical trends.

Enhances country-specific market agility and reduces lost sales due to stockouts.

Supports cross-departmental planning—from procurement to marketing.


### 16) Which product category had the most warranty claims filed in the last 2 years?
SmartPhone.
![Slide16](https://github.com/user-attachments/assets/46b1fc1b-5f71-4b8d-9df2-ae2736be655c)

#### 📦 Warranty Performance by Category
#### 💼 Business Use Case:
Evaluate warranty claim trends across product categories to pinpoint areas of concern.

#### 🔍 Business Problem Solved:

Identifies categories with potential quality or durability issues.

Highlights need for category-specific quality control or vendor performance review.

Reduces after-sales service burden by targeting frequent-claim products.

#### 🌟 Strategic Benefit:

Improves brand trust and reduces long-term service costs.

Enables category-level warranty forecasting and contract renegotiation.

Supports targeted product improvement and customer satisfaction.

### 17) Determine the percentage chance of receiving claims after each purchase for each country.
UAE with the highest of 66 %.

![Slide17](https://github.com/user-attachments/assets/07615b14-c2b1-4e51-9717-339b166484a7)

#### 🌍 Use Case: Country-Level Claim Rate Analysis
#### 💼 Business Problem Solved:

Highlights regional differences in product performance and customer satisfaction.

Identifies high-risk markets with elevated claim frequencies.

Surfaces potential quality control or environmental issues impacting product longevity.

#### 📊 How It Helps:

Enables prioritization of after-sales support and warranty servicing by geography.

Informs localized product improvements or training for retailers.

##### 🌟 Strategic Benefit:

Enhances brand reputation by reducing post-sale friction in key markets.

Supports risk-based warranty cost forecasting and quality assurance scaling globally.



### 18) Analyze each store's yearly growth ratio.
Most of the stores had a negative yearly growth rate in 2023.

![Slide18](https://github.com/user-attachments/assets/2de38cfd-2898-4ba3-b552-6e773c3bf926)


#### 📈 Business Use Case: Store-Level Growth Analytics
#### 🔧 Business Problem Solved:

📊 Identifies underperforming vs. high-growth stores year over year.

🧩 Helps uncover regional trends, seasonality, and expansion opportunities.

⚙️ Enables data-driven decisions on resource allocation and marketing efforts.

#### 🎯 Strategic Benefit:

💰 Drives efficient investment and staffing based on growth trends.

📍 Supports territory planning and localized promotional strategies.

📉 Helps mitigate decline through early detection and intervention.

### 19)  What is the correlation between product price and warranty claims for products sold in the last five years?

--      Expensive is having a negative co-relation, Less Expensive has positive co-relation, Moderately Expensive has zero co-relation

![Slide19](https://github.com/user-attachments/assets/d2c0b6ce-d3de-4e20-8fe9-2c13f61a55cc)

#### 🛡️ Product Warranty Risk Evaluation Use Case:
Analyze warranty claim behavior for products sold in the last 2 years to identify high-risk product categories and patterns in post-sale issues.

#### Business Problem Solved:

Identifies which product categories or SKUs lead to more warranty claims, allowing quality control teams to investigate root causes.

Supports product redesign or service process improvements.

#### Strategic Benefit:

Helps reduce warranty-related costs.

Enhances brand trust and customer satisfaction.

Informs better manufacturing and vendor decisions.






### 20) Identify the store with the highest percentage of "Paid Repaired" claims in relation to total claims filed.
Apple Kurfürstendamm
![Slide20](https://github.com/user-attachments/assets/22f7fd37-794d-44fd-9fee-a8d1bdfa3e14)

#### 🛠️ Warranty Claims Analysis Based on Launch Timeline
Use Case: Warranty Monitoring for Recently Launched Products

#### Business Problem Solved: Identifies reliability concerns in newly launched products by tracking the number of warranty claims submitted within the last two years.

#### Strategic Benefit:

✅ Enables early detection of product quality issues

✅ Supports improved vendor or manufacturer accountability

✅ Reduces risk of brand damage due to faulty early-life products

✅ Helps forecast warranty reserve budgets more accurately


### 21) Calculate the monthly running total sales for each store over the past four years and compare the trends across this period?

![Slide21](https://github.com/user-attachments/assets/0cf586ee-bdff-445d-93cf-dfa8d69319e1)

#### 📈 Use Case: Store-Level Sales Performance Analysis
#### 💼 Business Problem Solved
Identifies long-term sales growth patterns across all retail locations.

Highlights top-performing and underperforming stores for performance reviews.

Enables forecasting future demand and budgeting store-level resources.

#### 🎯 Strategic Benefits
📊 Performance Benchmarking: Track growth momentum for strategic planning and bonuses.

📦 Inventory Management: Adjust restocking based on sales trajectories.

📍 Location Strategy: Refine decisions on expansion or consolidation.

💬 Personalized Coaching: Use growth trends to inform staff development at specific locations.
    
### 22) Analyze sales trends of product over time, segmented into key time periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.

![Slide22](https://github.com/user-attachments/assets/73f5e72b-462d-44c1-99e8-35ea7fcc8f15)

#### 🧮 Warranty Claims Intelligence by Launch Year
Use Case: Track warranty claims for newly launched products to assess post-launch quality and readiness.

### 📌 Business Solved:

Detect early reliability issues with new models.

Validate product readiness at launch.

Allocate support and warranty budget more accurately.

### 🎯 Strategic Benefit:

Supports launch quality assurance.

Enhances brand trust by ensuring post-launch reliability.

Drives predictive warranty modeling for future rollouts.



### 23) Advanced Warranty Claim Trends: Analyze trends in warranty claims compared to product sales over the past two years from the last sales date available.


![23](https://github.com/user-attachments/assets/d160e969-7f8d-4b29-8f0c-cfe86e97ab64)

### Business Use Case: Compare warranty claims to sales volume for each product, helping identify reliability issues over time.

#### 💼 Business Problem Solved:

Surfaces products with disproportionately high warranty claims.

Enables proactive action: improved product design, enhanced customer support, or recall decisions.

Helps correlate claim spikes with specific product batches or timeframes.

#### 🌟 Strategic Benefit:
Supports both warranty cost management and customer satisfaction improvement.    

### 24) product performance by average_pct_growth 
Apple Watch Series 4 registered growth of 614.57 %
![Slide24](https://github.com/user-attachments/assets/0906ae87-04f1-4b4a-a830-d70acb7a52c9)

#### Business Use Case: Product Performance by Average Percentage Growth

#### 💼 Business Problem Solved:

Identifies products with the highest and lowest growth rates.

Helps prioritize marketing efforts and stock allocation.

Assesses the effectiveness of promotional campaigns and product strategies.

#### 🌟 Strategic Benefit:

Provides insights for demand forecasting and inventory optimization.

Supports product development decisions and pricing adjustments based on growth trends.

### 25) Product Performance Analysis: Analyze sales performance over five years and identify the top 10 products with the highest percentage growth in units sold.

Apple Watch Series 4		614.57
iPad (6th Gen)			549.90
iPhone XS			543.79
MacBook Pro (M1, 13-inch)	15.31
HomePod mini			14.40
iPhone 12 Mini			12.47
iPad Pro (M2, 12.9-inch)	11.49
Mac mini (M1)			7.21
iPad (10th Gen)			4.33
MacBook Air (M1)		4.04

![Slide25](https://github.com/user-attachments/assets/07ae97ac-51d4-48e5-913b-7134ef983834)

#### Business Use Case: Product Performance Analysis

#### 💼 Business Problem Solved:

Identifies the top-performing products over the last five years.

Unveils growth patterns and shifts in product demand.

Guides stock replenishment and marketing focus based on high-growth products.

#### 🌟 Strategic Benefit:

Helps in optimizing product portfolios by focusing on high-growth items.

Drives marketing campaigns to support trending products.

Aids in inventory management by identifying products that need scaling based on performance.

### 26) Product Lifetime Value per year
MacBook Pro (M1 Max, 16-inch)	with the highest 145481803.36
HomePod mini	with the lowest	 530816.82
![Slide26](https://github.com/user-attachments/assets/547dbe35-8199-426e-8137-1624a865842c)

#### Business Use Case: Product Lifetime Value Analysis

#### 💼 Business Problem Solved:

Calculates the projected value of a product over its lifespan.

Helps in assessing the long-term profitability of products.

Guides decision-making on pricing, promotions, and product development.

#### 🌟 Strategic Benefit:

Optimizes product development strategies by focusing on high-value products.

Enhances customer retention strategies based on product value.

Supports financial forecasting by understanding future product revenue streams.


### 27) Product Lifetime Value per MONTH

same as 26.

![Slide27](https://github.com/user-attachments/assets/849b231c-e9af-4837-b8f2-70e5d7f8cc95)



  
### 28) Market Basket Analysis: Perform a market basket analysis to find products frequently purchased together.

MacBook Pro (M1 Max, 16-inch) & AirTag are frequently purchased together and iPhone 11 Pro Max & Apple Watch Series 3 are the least bought bundle.


![Slide28](https://github.com/user-attachments/assets/d8be2b5a-762a-4a4b-981c-59a32faba459)

#### 🧺 Product Bundling & Upselling 
#### Business Use Case: Market Basket Analysis — Identify products frequently bought together to inform bundling, promotions, and layout optimization.

#### Business Problem Solved:

Difficult to manually uncover frequent purchase combinations.

Missed cross-sell opportunities across product lines.

#### Strategic Benefits:

🎯 Targeted Promotions: Enable product bundling and combo deals to drive sales.

🛒 Store & App Layout Optimization: Place complementary products near each other.

📈 Boost Average Order Value (AOV): Encourage customers to purchase more in a single transaction.
  
### 29) Warranty Claim Rate per product

-- iPad Air (4th Gen) has the highest warranty claim rate

![Slide29](https://github.com/user-attachments/assets/1414ec84-5807-47c3-8007-9a4125bdb7a9)

#### 🛠️ Warranty Claim Rate per Product
#### 🔍 Business Use Case
Analyze the reliability and quality of individual products by measuring how often customers file warranty claims.

#### ✅ Business Problem Solved

Identifies low-performing or defect-prone products.

Enables manufacturers and retailers to proactively improve product quality or provide additional support.

Supports product recalls or redesign decisions with data.

##### 🎯 Strategic Benefit

Improves customer satisfaction and brand trust.

Reduces long-term support costs.

Enhances forecasting for warranty reserve funds.  

### 30) Show store-product pairs with consistent sales every month in 2023.


-- no store, product pairs have any consistent sales every month in 2023
![Slide30](https://github.com/user-attachments/assets/927436d1-ca93-4e4a-80f0-9ede70a58b19)


#### 🔍 Business Use Case:
Identify store-product pairs that demonstrated consistent monthly sales in 2023, indicating stable demand and customer loyalty.

#### 🧩 Business Problem Solved:
Detects products with reliable, predictable performance across stores, helping reduce inventory volatility and streamline replenishment.

#### 🚀 Strategic Benefit:
Enables more accurate demand forecasting, strengthens store-level assortment planning, and helps prioritize supply chain focus for steady performers.



### 31) Compute the lag in days between each sale and its corresponding warranty claim (if any), and rank the top 5 fastest claims. 
15 to 19 days for the top 5 fastest claims.
![Slide31](https://github.com/user-attachments/assets/4bfa7404-7e82-47e0-8be5-0cc9be40454b)

#### Business Use Case: Fastest Warranty Claim Resolution 
### Business Problem Solved: Uncovers product or process failures causing immediate issues post-sale, enabling quality and operational improvements.

### Strategic Benefit: Accelerates root-cause identification for rapid product feedback loops and proactive quality assurance.

### 32) Identify products that haven’t been sold in the past year but had warranty claims. 
-- iphone X, ipad (6th Gen), Apple Watch Series 4, iphone XS
![Slide32](https://github.com/user-attachments/assets/450b80a4-058b-427b-a6cc-96fcf549f8ef)

#### 💼 Business Use Case:
Flag products that haven’t been sold recently but continue to generate warranty claims.

#### 🧩 Business Problem Solved:
Detects lingering product quality issues long after the sales cycle.

Highlights the need for continued post-sale support even when a product is out of rotation.

Prevents blind spots in support planning and spare parts availability.

#### 🌟 Strategic Benefit:
Ensures proactive warranty service continuity.

Improves customer experience and trust, especially for legacy products.

Informs better retirement strategies and warranty reserve planning.



### 33) Calculate the average time to warranty claim per category and rank them 
Subscription Service with the  lowest average time to warranty claim of 143 days.
Accessory with the highest  average time to warranty claim of 196 days.

![Slide33](https://github.com/user-attachments/assets/d23f395f-06b2-4ca1-84d4-f82f0efd9912)

#### 💼 Business Use Case:
Category-Level Warranty Claim Response Monitoring

#### 🧩 Business Problem Solved:
Surfaces which product categories tend to experience faster or delayed warranty issues.

Reveals hidden product quality trends at a category level.

Informs resource allocation for support teams by category priority.

#### 🌟 Strategic Benefit:
Drives continuous product improvement by targeting underperforming categories.

Improves category-specific quality assurance processes.

Enables targeted risk mitigation, reducing potential brand damage and support costs.
