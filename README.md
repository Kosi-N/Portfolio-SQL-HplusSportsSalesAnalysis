# E-commerce: H+ Sport Sales insights

### The goal of this project is to investigate the sales performance at H+sport in order to create recommendations on budget allocation for future marketing campaigns.

## Table of contents
- [Project overview](#project-overview)
- [Data sources](#data-sources)
- [Tools](#tools)
- [Data Preparation](#data-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data analysis](#data-analysis)
- [Recommendations](#recommendations)

### Project overview
H+Sport  was founded in 2006  and operates an online store that sells nutritional products such as supplements, energy bars and rehydration solutions , along with active wear to meet the needs of active lifestyle consumers. You can navigate  to the H+ sport website by clicking [here](https://hplussport.com)

The board of directors wants to gain a better understanding of the H+ Sports mineral water 

### Data sources
The dataset consisted of five tables including information about the  products,customer demographics, orders, order items and saleperson information.
<img width="915" alt="dbg" src="https://github.com/user-attachments/assets/29dc6d78-5bf7-4bc1-b3c3-eb6c65f566b5">



The data sources can be found here :
Customer Data:  The primary dataset used for this analysis is the "sales_data.csv" file, containing detailed information about each sale made by the company.

### Tools
- Excel- Data Cleaning
  - [Download here](https://microsoft.com)
- SQL Server- Data analysis
- Tableau- Creating reports

### Data Preparation
In the initial data preparation phase , I performed the following tasks:
1. Data loading and inspection
2. Handling missing values
3. Data cleaning and formatting
   
```sql
-- Find null values in Customer table
SELECT *  
FROM Customer
WHERE Email is NULL OR  LastName is NULL OR Email is NULL OR Phone Is NULL

-- Remove null values in Customer table
SELECT FirstName,
LastName,
Email,
Phone 
FROM Customer
WHERE Email IS NOT NULL AND 
Phone IS NOT NULL;
``` 

### Exploratory Data Analysis

The EDA focused on analyzing the sales data to address the following key questions:

1. **Product Size Popularity:** Which mineral water size (20 ounces or 32 ounces) is more popular among customers? This analysis will help determine whether both sizes should continue to be offered or if the focus should be on one size.
```sql
SELECT p.Size,
SUM(Quantity) as TotalQuantity
FROM OrderItem
LEFT OUTER JOIN Product p
USING (ProductID)
GROUP BY Size
ORDER BY TotalQuantity DESC;
``` 
2. **Top-Selling Products:** What are the top-selling products, and do certain flavors or sizes stand out? This insight will guide the marketing team in deciding which products to prioritize in the upcoming campaign.
 ```sql
SELECT Variety,Size,o.ProductID,sum(Quantity) as TotalQuantity
FROM OrderItem o
LEFT JOIN Product p ON o.ProductID=p.ProductID
GROUP BY o.ProductID
ORDER BY TotalQuantity DESC;
```

3. **Sales Seasonality:** During which month are sales the highest, and when would be the optimal time to launch a new product? This will be determined by analyzing sales trends over time.
```sql
SELECT
MONTHNAME(CreationDate) as MonthName,
COUNT(o.OrderID) as TotalOrders,
SUM(Quantity) as TotalQuantity,
SUM(TotalDue) as TotalAmount
FROM Orders o
JOIN OrderItem oi USING  (OrderID)
GROUP BY MonthName
ORDER BY MonthName,TotalQuantity; 
```

4. **Geographic Sales Distribution:** Which states have the highest product purchase volumes? Identifying these states will help in targeting regional marketing efforts.
```sql
SELECT State,
Count(DISTINCT OrderID) as TotalOrders,
SUM(Quantity) as TotalQuantity
FROM Orders
LEFT JOIN OrderItem using (OrderID)
LEFT JOIN Customer Using (CustomerID)
GROUp BY State
order by TotalQuantity desc
LIMIT 10
```

5. **State-Specific Preferences**: What is the most popular product in each state? Knowing this will enable more tailored marketing efforts by region
```sql
WITH RankedVarieties AS (
    SELECT 
        c.State, 
        p.Variety, 
        SUM(oi.Quantity) AS TotalQuantitySold,
        ROW_NUMBER() OVER (PARTITION BY c.State ORDER BY SUM(oi.Quantity) DESC) AS rn
    FROM 
        Orders o
    JOIN Customer c ON o.CustomerID = c.CustomerID
    JOIN OrderItem oi ON o.OrderID = oi.OrderID
    JOIN  Product p ON oi.ProductID = p.ProductID
    GROUP BY 
        c.State, p.Variety
)
SELECT 
    State, 
    Variety, 
    TotalQuantitySold
FROM 
    RankedVarieties
WHERE 
    rn = 1;
```
   
6. **Product Pairing Trends:** What products are frequently purchased together? Understanding these combinations will assist in bundling and cross-marketing strategies.
```sql
SELECT 
    p1.ProductID AS Product1_ID,
    p1.Variety AS Product1_Name,
    p2.ProductID AS Product2_ID,
    p2.Variety AS Product2_Name,
    COUNT(*) AS Frequency
FROM  OrderItem oi1
JOIN  OrderItem oi2 ON oi1.OrderID = oi2.OrderID AND oi1.ProductID < oi2.ProductID
JOIN  Product p1 ON oi1.ProductID = p1.ProductID
JOIN Product p2 ON oi2.ProductID = p2.ProductID
GROUP BY p1.ProductID, p1.Variety, p2.ProductID, p2.Variety
ORDER BY 
    Frequency DESC
LIMIT 5;
```

7. **Customer Loyalty Ratio:** What is the ratio of repeat customers? Specifically, what proportion of customers have made at least two purchases within a given time period? This metric will help in evaluating customer loyalty and retention.
```sql
WITH Repeat_Customers as
(
  SELECT
CustomerID as Repeat_Cus
FROM Orders
GROUP BY CustomerID
HAVING COUNT(OrderID) > 1
)
SELECT 
(COUNT(DISTINCT Repeat_Cus)/
COUNT(DISTINCT CustomerID))*100 as CustomerRepeatRate
FROM Orders
LEFT OUTER JOIN Repeat_Customers
ON Orders.CustomerID = Repeat_Customers.Repeat_Cus;
```

### Results/findings

The analysis results summarized as follows:
1. the company sales have been steadyly increasing, peaks in holiday season
2. product a is the best performing
3. customer segments with high life time value should be targeted for marketing efforts

### Recommendations
Based on the analysis,i recommend the following actions
1. invest in marketing an dpromotion during the peak sales seasons to maximize revenue
2. Focus on expanding and promotion products in category A

### limitations 
I had to remove all zero values from budget and revenue colomns because they would have affected the accuracy of my conclusions from the analysis. There are still a few outliers even after the ommision but even then we can still see a positive correlation between budget and th enumber of votes with revenue.

   






   

