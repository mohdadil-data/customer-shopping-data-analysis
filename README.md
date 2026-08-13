# customer-shopping-data-analysis
End-to-end customer shopping behavior analysis using Python, SQL, and Power BI to uncover customer insights, product trends, sales patterns, and subscription behavior.
# Customer Shopping Behavior Analysis

An end-to-end **Data Analytics project** that analyzes customer shopping behavior using **Python, Pandas, MySQL, SQL, and Power BI**.

The project transforms raw customer transaction data into meaningful business insights by performing **data loading, exploratory data analysis (EDA), data cleaning, feature engineering, database integration, SQL analysis, customer segmentation, and interactive dashboard development**.

---

## 📌 Project Overview

Understanding customer behavior is important for businesses because it helps them make better decisions about products, pricing, discounts, customer retention, subscriptions, and marketing.

In this project, I analyzed **3,900 customer purchase records with 18 columns** to understand:

* Customer spending patterns
* Product and category performance
* Customer ratings
* Discount behavior
* Subscription behavior
* Shipping preferences
* Customer loyalty
* Repeat purchasing behavior
* Revenue contribution by customer groups
* Top-performing products

The final analysis was presented through an **interactive Power BI dashboard**.

---

## 🎯 Business Objective

The main objective of this project was to answer practical business questions such as:

* Which customer group generates the most revenue?
* Which products have the highest ratings?
* Do subscribers spend more than non-subscribers?
* Which products depend heavily on discounts?
* Who are the New, Returning, and Loyal customers?
* What are the top products in each category?
* Are repeat buyers more likely to subscribe?
* Which age groups contribute the most revenue?
* Does shipping type affect average purchase value?

---

## 🛠️ Technologies & Tools

| Technology           | Purpose                                             |
| -------------------- | --------------------------------------------------- |
| **Python**           | Data loading, cleaning, EDA and feature engineering |
| **Pandas**           | Data manipulation and analysis                      |
| **Jupyter Notebook** | Developing and documenting the analysis             |
| **MySQL**            | Storing and querying customer transaction data      |
| **SQL**              | Business analysis and extracting insights           |
| **SQLAlchemy**       | Connecting Python with MySQL                        |
| **PyMySQL**          | MySQL database connectivity                         |
| **Power BI**         | Interactive dashboard and data visualization        |

---

# 📂 Dataset

The dataset contains **3,900 records and 18 columns**.

### Main Data Categories

### Customer Information

* Customer ID
* Age
* Gender
* Location
* Subscription Status

### Purchase Information

* Item Purchased
* Category
* Purchase Amount
* Season
* Size
* Color

### Shopping Behavior

* Discount Applied
* Promo Code Used
* Previous Purchases
* Frequency of Purchases
* Review Rating
* Shipping Type
* Payment Method

The dataset initially contained **37 missing values in the Review Rating column**.

---

# 🔄 Project Workflow

```text
Raw Dataset
     ↓
Load Data using Python
     ↓
Exploratory Data Analysis
     ↓
Data Cleaning
     ↓
Feature Engineering
     ↓
MySQL Database Integration
     ↓
SQL Business Analysis
     ↓
Customer & Product Insights
     ↓
Power BI Dashboard
     ↓
Business Recommendations
```

---

# 1️⃣ Loading the Dataset in Python

The project starts by loading the customer shopping dataset using **Pandas**.

```python
import pandas as pd

df = pd.read_csv("customer_shopping_behavior.csv")
```

After loading the data, the first few rows were inspected using:

```python
df.head()
```

This helped verify that the dataset was loaded correctly.

---

# 2️⃣ Exploratory Data Analysis (EDA)

Before cleaning the data, I performed basic exploratory analysis to understand its structure and characteristics.

### Dataset Structure

```python
df.info()
```

This was used to inspect:

* Number of rows
* Number of columns
* Data types
* Non-null values
* Missing values

### Statistical Summary

```python
df.describe(include="all")
```

This helped understand:

* Average values
* Minimum and maximum values
* Distribution of numerical fields
* Common categorical values

### Missing Value Analysis

```python
df.isnull().sum()
```

This was used to identify missing values across the dataset.

The main missing-data issue was found in the **Review Rating** column.

---

# 3️⃣ Data Cleaning

Data cleaning was performed before moving to the SQL analysis stage.

## Handling Missing Review Ratings

Instead of removing rows with missing ratings, I filled missing ratings using the **median rating of the corresponding product category**.

```python
df["Review Rating"] = (
    df.groupby("Category")["Review Rating"]
      .transform(lambda x: x.fillna(x.median()))
)
```

This helps preserve the existing records while using category-specific information.

---

## Standardizing Column Names

The original column names were converted into a cleaner format.

```python
df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(" ", "_")

df = df.rename(
    columns={"purchase_amount_(usd)": "purchase_amount"}
)
```

For example:

```text
Purchase Amount (USD)
        ↓
purchase_amount
```

This makes SQL queries easier to write and maintain.

---

# 4️⃣ Feature Engineering

Feature engineering was performed to create additional fields that could provide better business insights.

## Age Group

Customers were divided into four age groups using `qcut()`:

```python
labels = [
    "Young Adult",
    "Adult",
    "Middle-aged",
    "Senior"
]

df["age_group"] = pd.qcut(
    df["age"],
    q=4,
    labels=labels
)
```

This makes it easier to compare purchasing behavior across customer age groups.

---

## Purchase Frequency in Days

The original purchase-frequency values were converted into numerical day intervals.

```python
frequency_mapping = {
    "Fortnightly": 14,
    "Weekly": 7,
    "Monthly": 30,
    "Quarterly": 90,
    "Bi-Weekly": 14,
    "Annually": 365,
    "Every 3 Months": 90
}

df["purchase_frequency_days"] = (
    df["frequency_of_purchases"].map(frequency_mapping)
)
```

This converts categorical information into a more analysis-friendly numerical field.

---

# 5️⃣ Data Consistency Check

I checked whether `discount_applied` and `promo_code_used` contained the same information.

```python
(df["discount_applied"] == df["promo_code_used"]).all()
```

After checking the redundancy, `promo_code_used` was removed:

```python
df = df.drop("promo_code_used", axis=1)
```

This reduced unnecessary duplication in the dataset.

---

# 6️⃣ MySQL Database Integration

After preparing the data, I connected Python with **MySQL** using:

* SQLAlchemy
* PyMySQL

Required packages:

```bash
pip install pymysql sqlalchemy pandas
```

Database connection was created using SQLAlchemy:

```python
from sqlalchemy import create_engine

engine = create_engine(
    "mysql+pymysql://username:password@host:port/database"
)
```

The customer data was then loaded into a MySQL table named:

```text
customer
```

The table was created/replaced using:

```python
df.to_sql(
    "customer",
    con=engine,
    if_exists="replace",
    index=False
)
```

I also verified the data by reading it back from MySQL:

```python
result = pd.read_sql(
    "SELECT * FROM customer;",
    engine
)
```

---

# 7️⃣ Database Column Standardization

Before the final database upload, the columns were renamed to SQL-friendly names:

```text
customer_id
age
gender
item_purchased
category
purchase_amount
location
size
color
season
review_rating
subscription_status
shipping_type
discount_applied
promo_code_used
previous_purchases
payment_method
frequency_of_purchases
```

The cleaned structure was then written back to the MySQL `customer` table.

---

# 8️⃣ SQL Business Analysis

After loading the data into MySQL, I used SQL to answer **10 practical business questions**.

## Q1. Revenue by Gender

**Question:**
How much revenue was generated by male and female customers?

```sql
SELECT gender,
       SUM(purchase_amount) AS revenue
FROM customer
GROUP BY gender;
```

**Purpose:**
Compare total revenue contribution by gender.

---

## Q2. High-Spending Discount Users

**Question:**
Which customers used discounts but still spent more than the average purchase amount?

```sql
SELECT customer_id,
       purchase_amount,
       discount_applied
FROM customer
WHERE discount_applied = 'Yes'
AND purchase_amount >
(
    SELECT AVG(purchase_amount)
    FROM customer
);
```

**Purpose:**
Identify customers who respond to discounts but still make relatively high-value purchases.

---

## Q3. Top 5 Products by Rating

**Question:**
Which 5 products have the highest average review rating?

```sql
SELECT item_purchased,
       AVG(review_rating) AS avg_rating
FROM customer
GROUP BY item_purchased
ORDER BY avg_rating DESC
LIMIT 5;
```

**Purpose:**
Identify highly rated products.

---

## Q4. Shipping Type Comparison

**Question:**
How does average purchase amount differ between Standard and Express shipping?

```sql
SELECT shipping_type,
       AVG(purchase_amount) AS average_purchase
FROM customer
GROUP BY shipping_type;
```

**Purpose:**
Understand whether shipping preference is associated with different purchase values.

---

## Q5. Subscribers vs Non-Subscribers

**Question:**
Do subscribed customers spend more?

```sql
SELECT subscription_status,
       AVG(purchase_amount) AS average_spend,
       SUM(purchase_amount) AS total_revenue
FROM customer
GROUP BY subscription_status;
```

**Purpose:**
Compare average spending and total revenue between subscribers and non-subscribers.

---

## Q6. Discount-Dependent Products

**Question:**
Which 5 products have the highest percentage of discounted purchases?

The analysis uses conditional aggregation:

```sql
SUM(
    CASE
        WHEN discount_applied = 'Yes'
        THEN 1
        ELSE 0
    END
)
```

and calculates the discount percentage for each product.

**Purpose:**
Identify products where discounts are used frequently.

---

## Q7. Customer Segmentation

Customers were classified into three groups based on previous purchases:

```text
0 purchases       → New
1–5 purchases     → Returning
6+ purchases      → Loyal
```

The SQL logic uses a `CASE` statement:

```sql
CASE
    WHEN previous_purchases = 0 THEN 'New'
    WHEN previous_purchases BETWEEN 1 AND 5 THEN 'Returning'
    ELSE 'Loyal'
END
```

**Purpose:**
Understand customer loyalty and retention patterns.

---

## Q8. Top 3 Products per Category

A `DENSE_RANK()` window function was used to identify the top 3 products within each category.

```sql
DENSE_RANK() OVER (
    PARTITION BY category
    ORDER BY COUNT(*) DESC
)
```

**Purpose:**
Find the most frequently purchased products within each category.

This query demonstrates the use of **window functions**, **aggregation**, and **partitioning**.

---

## Q9. Repeat Buyers and Subscriptions

Customers with more than 5 previous purchases were analyzed to check subscription behavior.

```sql
SELECT subscription_status,
       COUNT(*) AS total_customers
FROM customer
WHERE previous_purchases > 5
GROUP BY subscription_status;
```

**Purpose:**
Check whether repeat buyers are more likely to subscribe.

---

## Q10. Revenue by Age Group

Revenue was calculated for different age ranges using a `CASE` statement.

```sql
CASE
    WHEN age BETWEEN 18 AND 25 THEN '18-25'
    WHEN age BETWEEN 26 AND 35 THEN '26-35'
    WHEN age BETWEEN 36 AND 45 THEN '36-45'
    WHEN age BETWEEN 46 AND 55 THEN '46-55'
    ELSE '56+'
END
```

**Purpose:**
Identify which age groups contribute the most revenue.

---

# 9️⃣ SQL Concepts Demonstrated

This project demonstrates practical SQL concepts including:

* `SELECT`
* `WHERE`
* `GROUP BY`
* `ORDER BY`
* `LIMIT`
* `SUM()`
* `AVG()`
* `COUNT()`
* `CASE`
* Subqueries
* Conditional aggregation
* Window functions
* `DENSE_RANK()`
* `PARTITION BY`

These techniques were used to convert raw transaction records into business insights.

---

# 📊 10️⃣ Power BI Dashboard

After completing the Python and SQL analysis, I created an interactive **Power BI dashboard** to communicate the results visually.

### Dashboard Includes

* Total Number of Customers
* Average Purchase Amount
* Average Review Rating
* Subscription Status
* Revenue by Category
* Sales by Category
* Revenue by Age Group
* Sales by Age Group
* Subscription Status filters
* Gender filters
* Shipping Type filters
* Category filters

The dashboard allows users to interactively filter the data and explore customer behavior from different perspectives.

---

# 📈 Key Insights

The analysis produced several useful business findings.

### Customer Revenue

Male customers generated higher total revenue than female customers in the analyzed dataset.

### Product Ratings

The top five products by average rating were:

1. Gloves
2. Sandals
3. Boots
4. Hat
5. Skirt

### Shipping

Express shipping showed a slightly higher average purchase amount than Standard shipping.

### Customer Segmentation

The customer base was dominated by the **Loyal** segment, followed by **Returning** and **New** customers.

### Discounts

Hat, Sneakers, Coat, Sweater and Pants were among the products with the highest percentages of discounted purchases.

### Categories

The Power BI dashboard shows **Clothing** as the strongest category, followed by **Accessories**, while Footwear and Outerwear contribute less.

These findings support the business recommendations included in the project report.

---

# 💡 Business Recommendations

Based on the analysis, the following recommendations were identified:

### 1. Increase Subscription Adoption

Promote exclusive benefits, offers and loyalty advantages to encourage more customers to subscribe.

### 2. Reward Loyal Customers

Create loyalty programs and rewards for repeat customers.

### 3. Review Discount Strategy

Products with very high discount usage should be monitored to make sure discounts increase sales without unnecessarily reducing profit margins.

### 4. Promote Highly Rated Products

Highly rated and frequently purchased products can be highlighted in marketing campaigns.

### 5. Use Targeted Marketing

Marketing campaigns can focus on high-revenue customer groups, products and shipping segments.

These recommendations are consistent with the project's original business recommendations.

---

# 📁 Suggested Repository Structure

```text
customer-shopping-behavior-analysis/
│
├── data/
│   └── customer_shopping_behavior.csv
│
├── python/
│   └── customer_behavior_analysis.ipynb
│
├── sql/
│   └── customer_behavior_analysis.sql
│
├── powerbi/
│   └── customer_behavior_dashboard.pbix
│
├── reports/
│   └── Customer_Shopping_Behavior_Report.pdf
│
├── assets/
│   └── dashboard.png
│
├── README.md
└── requirements.txt
```

---

# ⚙️ How to Run the Project

## Step 1 — Clone the Repository

```bash
git clone https://github.com/yourusername/customer-shopping-behavior-analysis.git
cd customer-shopping-behavior-analysis
```

## Step 2 — Install Python Libraries

```bash
pip install pandas sqlalchemy pymysql jupyter
```

## Step 3 — Open the Notebook

```bash
jupyter notebook
```

Open:

```text
python/customer_behavior_analysis.ipynb
```

## Step 4 — Configure MySQL

Create a MySQL database:

```sql
CREATE DATABASE customer_shopping_behavior;
```

Update your MySQL username, password, host, port and database name in the Python notebook.

## Step 5 — Run the Python Analysis

Run the notebook cells in order to:

1. Load the dataset
2. Perform EDA
3. Identify missing values
4. Clean the data
5. Create new features
6. Connect to MySQL
7. Load the customer data into MySQL

## Step 6 — Run SQL Analysis

Open:

```text
sql/customer_behavior_analysis.sql
```

Select the database:

```sql
USE customer_shopping_behavior;
```

Then execute the business queries.

## Step 7 — Open Power BI

Open the Power BI file:

```text
powerbi/customer_behavior_dashboard.pbix
```

Refresh the dataset if required and interact with the dashboard filters.

---

# 🧠 What I Learned

Through this project, I gained practical experience in:

* Loading and exploring real-world datasets
* Data cleaning using Pandas
* Handling missing values
* Feature engineering
* Data standardization
* Working with MySQL databases
* Connecting Python with MySQL
* Writing SQL business queries
* Using aggregate functions
* Using subqueries
* Using `CASE` statements
* Using SQL window functions
* Customer segmentation
* Business analysis
* Data visualization
* Power BI dashboard development
* Converting data findings into business recommendations

---

# 🚀 Future Improvements

Possible future improvements include:

* Adding more advanced Power BI measures using DAX
* Creating more detailed customer segmentation
* Adding time-based sales analysis if transaction dates are available
* Building automated data refresh pipelines
* Adding predictive customer churn analysis
* Developing customer lifetime value analysis
* Using machine learning to predict purchasing behavior

---

# ⚠️ Project Notes

The project contains two different approaches for age grouping:

* The Python notebook creates age groups using `pandas.qcut()`.
* The SQL analysis creates fixed age ranges such as `18–25`, `26–35`, `36–45`, `46–55`, and `56+`.

Both approaches are retained because they are part of the original project workflow.

---

# 👨‍💻 Author

**Mohd Adil Khan**

**B.Tech – Computer Science & Engineering**

Aspiring **Data Analyst | Business Analyst**

### Skills Demonstrated

`Python` `Pandas` `SQL` `MySQL` `Power BI` `Data Analysis` `EDA` `Data Cleaning` `Feature Engineering` `Business Intelligence`

---

## ⭐ If you found this project useful

Feel free to explore the repository, review the SQL queries, analyze the notebook, and check out the Power BI dashboard.
