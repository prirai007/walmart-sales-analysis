Project Overview

This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. We utilize Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions. The project is ideal for data analysts looking to develop skills in data manipulation, SQL querying, and data pipeline creation.

Project Steps
1. Set Up the Environment
Tools Used: Visual Studio Code (VS Code), Python, SQL (MySQL and PostgreSQL)
Goal: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.
2. Set Up Kaggle API
API Setup: Obtain your Kaggle API token from Kaggle by navigating to your profile settings and downloading the JSON file.
Configure Kaggle:
Place the downloaded kaggle.json file in your local .kaggle folder.
Use the command kaggle datasets download -d <dataset-path> to pull datasets directly into your project.
3. Download Walmart Sales Data
Data Source: Use the Kaggle API to download the Walmart sales datasets from Kaggle.
Dataset Link: Walmart Sales Dataset
Storage: Save the data in the data/ folder for easy reference and access.
4. Install Required Libraries and Load Data
Libraries: Install necessary Python libraries using:
pip install pandas numpy sqlalchemy
Loading Data: Read the data into a Pandas DataFrame for initial analysis and transformations.
5. Explore the Data
Goal: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
Analysis: Use functions like .info(), .describe(), and .head() to get a quick overview of the data structure and statistics.
6. Data Cleaning
Remove Duplicates: Identify and remove duplicate entries to avoid skewed results.
Handle Missing Values: Drop rows or columns with missing values if they are insignificant; fill values where essential.
Fix Data Types: Ensure all columns have consistent data types (e.g., dates as datetime, prices as float).
Currency Formatting: Use .replace() to handle and format currency values for analysis.
Validation: Check for any remaining inconsistencies and verify the cleaned data.
7. Feature Engineering
Create New Columns: Calculate the Total Amount for each transaction by multiplying unit_price by quantity and adding this as a new column.
Enhance Dataset: Adding this calculated field will streamline further SQL analysis and aggregation tasks.
8. Load Data into MySQL and PostgreSQL
Set Up Connections: Connect to MySQL and PostgreSQL using sqlalchemy and load the cleaned data into each database.
Table Creation: Set up tables in both MySQL and PostgreSQL using Python SQLAlchemy to automate table creation and data insertion.
Verification: Run initial SQL queries to confirm that the data has been loaded accurately.
9. SQL Analysis: Complex Queries and Business Problem Solving
Business Problem-Solving: Write and execute complex SQL queries to answer critical business questions, such as:
Revenue trends across branches and categories.
Identifying best-selling product categories.
Sales performance by time, city, and payment method.
Analyzing peak sales periods and customer buying patterns.
Profit margin analysis by branch and category.
Documentation: Keep clear notes of each query's objective, approach, and results.
10. Project Publishing and Documentation
Documentation: Maintain well-structured documentation of the entire process in Markdown or a Jupyter Notebook.
Project Publishing: Publish the completed project on GitHub or any other version control platform, including:
The README.md file (this document).
Jupyter Notebooks (if applicable).
SQL query scripts.
Data files (if possible) or steps to access them.
Requirements
Python 3.8+
SQL Databases: MySQL, PostgreSQL
Python Libraries:
pandas, numpy, sqlalchemy
Kaggle API Key (for data downloading)
Getting Started
Clone the repository:
git clone <repo-url>
Install Python libraries:
pip install -r requirements.txt
Set up your Kaggle API, download the data, and follow the steps to load and analyze.
Project Structure
|-- data/                     # Raw data and transformed data
|-- notebooks/                # Jupyter notebooks for Python analysis
|-- README.md                 # Project documentation
|-- requirements.txt          # List of required Python libraries
|-- main.py                   # Main script for loading, cleaning, and processing data
Results and Insights
This section will include your analysis findings:

Sales Insights: Key categories, branches with highest sales, and preferred payment methods.
Profitability: Insights into the most profitable product categories and locations.
Customer Behavior: Trends in ratings, payment preferences, and peak shopping hours.

Future Enhancements
Possible extensions to this project(Future works):
Integration with a dashboard tool (e.g., Power BI or Tableau) for interactive visualization.
Additional data sources to enhance analysis depth.
Automation of the data pipeline for real-time data ingestion and analysis.

Acknowledgments
Data Source: Kaggle’s Walmart Sales Dataset
Inspiration: Walmart’s business case studies on sales and supply chain optimization.

MySQL queries for all business problems mentioned in the pdf 


## SQL Business Problem Solutions

### 1. Analyze Payment Methods and Sales

This query groups the sales data by payment method to count the number of transactions and the total items sold via each payment type. By using `COUNT(*)` for transactions and `SUM(quantity)` for items, we see how often each payment method is used and the associated sales volume. This analysis reveals customer payment preferences and helps Walmart optimize payment processes.

```sql
SELECT payment_method, COUNT(*) AS transaction_count, SUM(quantity) AS items_sold
FROM walmart_clean_data
GROUP BY payment_method;
```

### 2. Identify the Highest-Rated Category in Each Branch

This query computes the average customer rating of each product category per branch, then identifies the category with the highest average rating at each location. We first calculate `AVG(rating)` for each branch-category combination, then join with a subquery that finds the maximum average rating per branch. The result highlights the most highly-rated category at each branch, guiding targeted marketing and inventory decisions.

```sql
SELECT t.branch, t.category, t.avg_rating
FROM (
    SELECT branch, category, AVG(rating) AS avg_rating
    FROM walmart_clean_data
    GROUP BY branch, category
) AS t
JOIN (
    SELECT branch, MAX(avg_rating) AS max_avg_rating
    FROM (
        SELECT branch, category, AVG(rating) AS avg_rating
        FROM walmart_clean_data
        GROUP BY branch, category
    ) AS sub
    GROUP BY branch
) AS m
ON t.branch = m.branch
AND t.avg_rating = m.max_avg_rating;
```

### 3. Determine the Busiest Day for Each Branch

This query finds the day of the week with the most transactions for each branch. We convert the `date` field to a weekday name using `DAYNAME(STR_TO_DATE(date, '%d/%m/%y'))` and count transactions for each branch-day combination. Then we select the day with the maximum count per branch. Identifying each branch’s busiest day helps optimize staffing and inventory management for peak demand periods.

```sql
SELECT d.branch, d.day_of_week, d.transaction_count
FROM (
    SELECT branch,
           DAYNAME(STR_TO_DATE(date, '%d/%m/%y')) AS day_of_week,
           COUNT(*) AS transaction_count
    FROM walmart_clean_data
    GROUP BY branch, day_of_week
) AS d
JOIN (
    SELECT branch, MAX(transaction_count) AS max_count
    FROM (
        SELECT branch,
               DAYNAME(STR_TO_DATE(date, '%d/%m/%y')) AS day_of_week,
               COUNT(*) AS transaction_count
        FROM walmart_clean_data
        GROUP BY branch, day_of_week
    ) AS t
    GROUP BY branch
) AS m
ON d.branch = m.branch
AND d.transaction_count = m.max_count;
```

### 4. Calculate Total Quantity Sold by Payment Method

This query sums the total number of items sold for each payment method. By grouping on `payment_method` and applying `SUM(quantity)`, we get the total item volume transacted via each payment type. The result shows how sales volume is distributed across payment channels, which can inform sales and payment processing strategies.

```sql
SELECT payment_method, SUM(quantity) AS total_items_sold
FROM walmart_clean_data
GROUP BY payment_method;
```

### 5. Analyze Category Ratings by City

This query computes the average, minimum, and maximum customer rating for each product category within each city. We group the data by `city` and `category` and use the aggregate functions `AVG(rating)`, `MIN(rating)`, and `MAX(rating)`. The results reveal how ratings vary by category and region, informing city-level promotions and product strategy to improve customer satisfaction.

```sql
SELECT city, category,
       AVG(rating) AS avg_rating,
       MIN(rating) AS min_rating,
       MAX(rating) AS max_rating
FROM walmart_clean_data
GROUP BY city, category;
```

### 6. Calculate Total Profit by Category

This query calculates total profit for each product category by using the `profit_margin` and `total` columns. We multiply `total * profit_margin` to get the profit for each transaction, then sum these values per category. The categories are ordered by the resulting profit in descending order, highlighting the most profitable product lines for Walmart to focus on expanding or repricing.

```sql
SELECT category, SUM(total * profit_margin) AS total_profit
FROM walmart_clean_data
GROUP BY category
ORDER BY total_profit DESC;
```

### 7. Determine the Most Common Payment Method per Branch

This query identifies the most frequently used payment method at each branch. We group the data by `branch` and `payment_method` and count the transactions for each. Then we join this result with a subquery that finds the maximum transaction count per branch. The final output lists, for each branch, the payment method used in the most transactions, showing branch-specific payment preferences.

```sql
SELECT t.branch, t.payment_method, t.usage_count
FROM (
    SELECT branch, payment_method, COUNT(*) AS usage_count
    FROM walmart_clean_data
    GROUP BY branch, payment_method
) AS t
JOIN (
    SELECT branch, MAX(usage_count) AS max_count
    FROM (
        SELECT branch, payment_method, COUNT(*) AS usage_count
        FROM walmart_clean_data
        GROUP BY branch, payment_method
    ) AS sub
    GROUP BY branch
) AS m
ON t.branch = m.branch
AND t.usage_count = m.max_count;
```

### 8. Analyze Sales Shifts Throughout the Day

This query classifies each transaction into Morning, Afternoon, or Evening shifts based on its time and counts transactions in each shift. We use a `CASE` expression on the `time` column (converting it with `TIME(time)`) to assign a shift label, then count the transactions per shift. For example, transactions before noon are labeled 'Morning', noon to 4:59pm 'Afternoon', and the rest 'Evening'. Summing these counts reveals peak sales shifts to inform staffing and inventory planning.

```sql
SELECT shift, COUNT(*) AS transaction_count
FROM (
    SELECT CASE
             WHEN TIME(`time`) >= '05:00:00' AND TIME(`time`) < '12:00:00' THEN 'Morning'
             WHEN TIME(`time`) >= '12:00:00' AND TIME(`time`) < '17:00:00' THEN 'Afternoon'
             ELSE 'Evening'
           END AS shift
    FROM walmart_clean_data
) AS shifts
GROUP BY shift;
```

### 9. Identify Branches with Highest Revenue Decline Year-Over-Year

This query finds branches that had the largest drop in revenue compared to the previous year. We first calculate annual revenue per branch by summing `total` grouped by branch and year (`YEAR(STR_TO_DATE(date, '%d/%m/%y'))`). Then we self-join the yearly results on `branch` where the years differ by one to compute the revenue decline (`previous_year_revenue - current_year_revenue`). Sorting this difference in descending order highlights the branches with the steepest revenue declines, indicating where sales have fallen most sharply.

```sql
SELECT curr.branch,
       prev.year AS previous_year,
       curr.year AS current_year,
       (prev.total_revenue - curr.total_revenue) AS revenue_decline
FROM (
    SELECT branch,
           YEAR(STR_TO_DATE(date, '%d/%m/%y')) AS year,
           SUM(total) AS total_revenue
    FROM walmart_clean_data
    GROUP BY branch, year
) AS curr
JOIN (
    SELECT branch,
           YEAR(STR_TO_DATE(date, '%d/%m/%y')) AS year,
           SUM(total) AS total_revenue
    FROM walmart_clean_data
    GROUP BY branch, year
) AS prev
ON curr.branch = prev.branch
AND curr.year = prev.year + 1
ORDER BY revenue_decline DESC;
```

Each of these queries is written for MySQL and uses the column names from the cleaned dataset. They employ aggregation (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`), grouping, and date/time functions (`STR_TO_DATE`, `DAYNAME`, `TIME`) as needed to answer the business questions. Together, these queries can be run against the loaded Walmart sales table (`walmart_clean_data`) to extract actionable insights for Walmart’s management.
