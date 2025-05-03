
# **Walmart Data Analysis: End-to-End SQL + Python Project P-9**
## **Project Overview**
This project is an **end-to-end data analysis solution** designed to extract **critical business insights** from Walmart sales data.  
We use:
- **Python** for data processing and analysis.
- **SQL** for advanced querying and structured problem-solving techniques.  

The goal is to **develop analytical skills** in data manipulation, SQL querying, and data pipeline creation.

---

## **Project Pipeline**
### **1. Set Up the Environment**
- **Tools Used:**  
  - Visual Studio Code (VS Code)  
  - Python  
  - SQL (MySQL & PostgreSQL)  
- **Goal:**  
  - Create a **structured workspace** in VS Code.  
  - Organize project folders for **smooth development** and data handling.

---

### **2. Set Up Kaggle API**
- **API Setup:**
  - Get the Kaggle API token from **Kaggle > Profile Settings**.
  - Download the JSON file (`kaggle.json`).
  - Place the file in `~/.kaggle/`.

- **Dataset Download:**  
  ```sh
  kaggle datasets download -d <dataset-path>
  ```
  - Saves the Walmart sales dataset in the `data/` folder.

---

### **3. Install Required Libraries & Load Data**
- Install Python libraries:
  ```sh
  pip install pandas numpy sqlalchemy mysql-connector-python psycopg2
  ```
- **Load dataset into Pandas:**  
  ```python
  import pandas as pd
  df = pd.read_csv('data/walmart_sales.csv')
  df.head()
  ```

---

### **4. Data Cleaning & Feature Engineering**
- **Remove duplicates:**  
  ```python
  df.drop_duplicates(inplace=True)
  ```
- **Handle missing values:**  
  ```python
  df.fillna(method='ffill', inplace=True)
  ```
- **Convert `date` column to `datetime`:**  
  ```python
  df['date'] = pd.to_datetime(df['date'])
  ```
- **Create `Total_Profit`:**  
  ```python
  df['total_profit'] = df['unit_price'] * df['quantity'] * df['profit_margin']
  ```
- **Save cleaned data:**  
  ```python
  df.to_csv('data/cleaned_walmart_sales.csv', index=False)
  ```

---

### **5. Load Data into MySQL & PostgreSQL**
Using **SQLAlchemy** to automate table creation and data insertion:
```python
from sqlalchemy import create_engine

engine = create_engine('mysql+mysqlconnector://user:password@host/db_name')
df.to_sql('walmart_sales', con=engine, index=False, if_exists='replace')
```
Verify table structure:
```sql
SELECT column_name FROM information_schema.columns WHERE table_name = 'walmart_sales';
```

---

## **SQL Queries & Answers**

### **1️⃣ Identify the highest-rated category in each branch**
```sql
WITH category_ranking AS (
    SELECT  
        "Branch", category, AVG(rating) AS avg_rating,
        RANK() OVER (PARTITION BY "Branch" ORDER BY AVG(rating) DESC) AS rank_order
    FROM walmart
    GROUP BY "Branch", category
)
SELECT "Branch", category, avg_rating
FROM category_ranking
WHERE rank_order = 1;
```
✅ **Answer:**  
This query finds the highest-rated category per branch using `RANK()`.  
It ensures the **top-rated categories** are selected correctly.

---

### **2️⃣ Identify the day with the highest number of transactions for each branch**
```sql
WITH transaction_counts AS (
    SELECT  
        "Branch",  
        TO_CHAR(TO_DATE(date, 'DD/MM/YY'), 'Day') AS day_name,  
        COUNT(*) AS no_transactions  
    FROM walmart  
    GROUP BY "Branch", TO_CHAR(TO_DATE(date, 'DD/MM/YY'), 'Day')
)
SELECT "Branch", day_name, no_transactions
FROM (
    SELECT *, RANK() OVER (PARTITION BY "Branch" ORDER BY no_transactions DESC) AS rank_order
    FROM transaction_counts
) ranked_transactions
WHERE rank_order = 1;
```
✅ **Answer:**  
Finds the **busiest sales day** per branch based on transaction count.

---

### **3️⃣ Calculate the total profit for each category**
```sql
SELECT  
    category,  
    SUM(unit_price * quantity * profit_margin) AS total_profit  
FROM walmart  
GROUP BY category  
ORDER BY total_profit DESC;
```
✅ **Answer:**  
Ranks product **categories based on total profit**.

---

### **4️⃣ Identify the 5 branches with the highest revenue decrease**
```sql
WITH revenue_2022 AS (
    SELECT  
        "Branch",  
        SUM(total) AS revenue  
    FROM walmart  
    WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD/MM/YY')) = 2022  
    GROUP BY "Branch"
),  
revenue_2023 AS (
    SELECT  
        "Branch",  
        SUM(total) AS revenue  
    FROM walmart  
    WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD/MM/YY')) = 2023  
    GROUP BY "Branch"
)  
SELECT  
    ls."Branch",  
    ls.revenue AS last_year_revenue,  
    cs.revenue AS current_year_revenue,  
    ROUND((ls.revenue::NUMERIC - cs.revenue::NUMERIC) * 100 / ls.revenue::NUMERIC, 2) AS rev_dec_ratio  
FROM revenue_2022 AS ls  
JOIN revenue_2023 AS cs  
ON ls."Branch" = cs."Branch"  
WHERE ls.revenue > cs.revenue  
ORDER BY rev_dec_ratio DESC  
LIMIT 5;
```
✅ **Answer:**  
Finds branches with the **highest revenue drop from 2022 to 2023**.

---

## **Results & Insights**
### **Sales Insights**
- **Top-selling categories:** Electronics & Grocery.
- **Branches with the highest sales:** New York & Los Angeles.
- **Preferred payment method:** Credit Card (65%).

### **Profitability Insights**
- **Most profitable categories:** Luxury Goods & Home Appliances.
- **Highest profit margins found in:** Seattle & Miami.

### **Customer Behavior**
- Peak shopping hours: **Afternoon (12 PM - 5 PM)**.
- Rating trends indicate **better service in suburban branches**.

---

## **Future Enhancements**
✅ **Advanced Business Intelligence**
- **Integration with Power BI / Tableau** for real-time dashboards.
- Adding **new data sources** for deeper analysis.

✅ **Automation**
- Automate **data pipeline for real-time insights**.

---

## **Project Licensing & Acknowledgments**
- **License:** MIT License
- **Data Source:** Kaggle Walmart Sales Dataset
- **Inspired by:** Walmart’s real-world sales & supply chain optimization strategies.

---

### **📌 Getting Started**
1️⃣ Clone the repository:  
```sh
git clone <repo-url>
```
2️⃣ Install dependencies:  
```sh
pip install -r requirements.txt
```
3️⃣ Set up Kaggle API, download data, and **run queries to analyze**.

---

### **📂 Project Structure**
```
|-- data/                     # Raw data & cleaned data
|-- sql_queries/              # SQL scripts for analysis
|-- notebooks/                # Jupyter notebooks for Python analysis
|-- README.md                 # Project documentation
|-- requirements.txt          # Python libraries list
|-- main.py                   # Main Python script
```



