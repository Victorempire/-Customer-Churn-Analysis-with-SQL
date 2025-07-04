# Customer Churn Analysis with SQL
### This project analyzes customer churn using the BankChurners dataset. The goal is to explore the key factors driving customer attrition in a financial institution and derive actionable business insights. A star schema was designed with fact and dimension tables to optimize query performance and reporting.
# 1. Introduction
This analysis explores key issues affecting customer retention in a retail banking environment. Using the BankChurners dataset, which contains detailed information on over 10,000 clients including demographics, account activity, credit behavior, and attrition status the analysis identifies behavioral patterns and risk indicators linked to customer churn. By applying structured SQL queries within a dimensional data model, the goal is to support data driven strategies that enhance customer engagement, reduce attrition, and improve long-term client value.
# Problem Statement
Customer churn remains a persistent concern for banks, directly affecting profitability and customer lifetime value. To mitigate this risk, banks must understand who is churning, why they are churning, and what behavioral or demographic patterns contribute to attrition.
This analysis investigates customer churn using a banking dataset by answering eight critical business questions focused on:
- Overall churn rate – What percentage of customers have left the bank?
- Age group impact – Are certain age segments more prone to churn?
- Income category analysis – Does income level influence churn likelihood?
- Customer inactivity – Is inactivity a reliable predictor of churn?
- Card category effect – Are some card types associated with higher churn?
- Transactions and income – Do income and transaction patterns correlate with attrition?
- Revolving balance impact – Does a customer's credit behavior signal churn risk?
- Churn risk profiling – Can we identify existing customers likely to churn based on inactivity, income level, and credit behavior?
 The goal is to extract actionable insights from customer demographic, transactional, and behavioral data to support strategic decision-making, reduce attrition, and enhance customer retention efforts.
# Key Insights
- Overall Churn Rate: Approximately 16.1% of customers have churned, indicating a need for stronger retention strategies.
- Age Group Impact: Customers aged 48–58 exhibit the highest churn rates, suggesting that age plays a significant role in attrition behavior.
- Inactivity as a Risk Factor: Customers with 4 or more months of inactivity have a churn rate of 29.89%, while those with 0 months of inactivity exhibit the highest churn rate at 51.72%. Although the latter group represents a smaller customer segment, the sharp attrition rate suggests that both long-term inactivity and sudden churn among active users are strong predictors of customer loss. This indicates that retention strategies should target both highly inactive and seemingly active customers to effectively reduce churn.
# Data Cleaning & Data Modeling 
Before performing any analysis, the original BankChurners dataset was reviewed and structured for relational modeling. Minimal cleaning was required, but key transformations were implemented to enable efficient querying and better data management.
```SQL
-- Check total records
SELECT COUNT(*)  AS Total_records FROM BankChurners;

-- View column names and data types 
EXEC sp_help BankChurners;

-- Check for nulls
SELECT 
  SUM(CASE WHEN CLIENTNUM IS NULL THEN 1 ELSE 0 END) AS Null_ClientNum,
  SUM(CASE WHEN Attrition_Flag IS NULL THEN 1 ELSE 0 END) AS Null_AttritionFlag,
FROM BankChurners;

-- Check for duplicates
SELECT CLIENTNUM, COUNT(*) AS freq
FROM BankChurners
GROUP BY CLIENTNUM
HAVING COUNT(*) > 1;

-- Value distributions
SELECT DISTINCT Education_Level FROM BankChurners;
SELECT COUNT(DISTINCT Income_Category) FROM BankChurners;

--Rename column CLIENTNUM to Client_Number in BankChurners table
EXEC sp_rename'BankChurners.CLIENTNUM','Client_Number','COLUMN';

--- Entity Relationship Diagrams(ERDs) Of BankChurners
--- Step 1 Create Fact Table
Create Table Transactions (
Client_Number varchar(50) primary key,
Total_Revolving_Bal Smallint,
Avg_Open_To_Buy float,
Total_Amt_Chng_Q4_Q1 float,
Total_Trans_Ct tinyint,
Total_Ct_Chng_Q4_Q1 float,
Avg_Utilization_Ratio float,
Total_Transaction_Amt smallint
);
---Step 2 Insert Data From Existing Table
INSERT INTO Transactions(
        Client_Number,
        Total_Revolving_Bal,
	    Avg_Open_To_Buy,
	    Total_Amt_Chng_Q4_Q1,
	    Total_Trans_Ct,
	    Total_Ct_Chng_Q4_Q1,
	    Avg_Utilization_Ratio,
        Total_Transaction_Amt)
 select  Client_Number,
          Total_Revolving_Bal,
	      Avg_Open_To_Buy,
	      Total_Amt_Chng_Q4_Q1,
	      Total_Trans_Ct,
	      Total_Ct_Chng_Q4_Q1,
	      Avg_Utilization_Ratio,
          Total_Trans_Amt
from BankChurners;

---Create Diamension Tables
---Step 1 Create Customer Activities table
Create Table Activities (
Client_Number Varchar(50),
Total_Relationship_Count tinyint,
Months_Inactive_12_mon tinyint,
Contracts_Count_12_Mon tinyint
foreign key (Client_Number) references Transactions(Client_Number));
---Step 2 insert data into existing Table
INSERT INTO  Activities(
              Client_Number,
			  Total_Relationship_Count,
			  Months_Inactive_12_Mon,
			  Contracts_Count_12_Mon)
SELECT    
             Client_Number
		     Total_Relationship_Count,
	         Months_Inactive_12_mon,
		     Contacts_Count_12_mon
FROM BankChurners;

----Step 1 Create Customers Table
Create Table Customers (
Client_Number Varchar(50),
Customer_Age tinyint,
Gender varchar(50),
Dependent_count tinyint,
Education_Level Varchar(50),
Marital_Status Varchar(50),
Foreign key (Client_Number) references Transactions(Client_Number));
----Step 2 insert data into existing Table
INSERT INTO Customers (
            Client_Number,
			 Gender,
			 Customer_Age,
			 Dependent_count,
			 Education_Level,
			 Marital_Status)
SELECT Client_Number,
	   Gender,
	   Customer_Age,
	   Dependent_count,
       Education_Level,
	   Marital_Status
FROM BankChurners;

----- Step 1 Create Churn Status
Create Table [Churn Status] (
Client_Number varchar(50),
Attrition_Flag varchar(50)
foreign key (Client_Number) references Transactions (Client_Number));
---Step 2 Insert Data into existing Table
INSERT INTO [Churn Status](
            Client_Number,
			Attrition_Flag)
SELECT Client_Number,
       Attrition_Flag
from BankChurners;
 
----Account Table
Create Table Account (
Client_Number Varchar(50),
Income_Category varchar(100),
Credit_Limit float,
Months_on_book tinyint,
Card_Category varchar(50));

INSERT INTO Account(
            Client_Number,
		    Income_Category,
		    Credit_Limit,
		    Months_on_book,
		    Card_Category)
SELECT 
          Client_Number,
	      Income_Category, 
	      Credit_Limit,
	      Months_on_book,
	      Card_Category
FROM BankChurners;
```
# Entity Relationship Diagram (ERD)
 This diagram illustrates the relationships between fact and dimension tables used in the analysis, based on the `BankChurners` dataset.
![churned](https://github.com/user-attachments/assets/2addeffc-cb54-4a7b-a9ac-e8685e248d4b)
## Data Analysis and SQL Queries
This section presents key business questions and their answers using structured SQL queries. Each
## 1. Overall Churn Rate
```SQL
---Analysis 1 What is the overall churn rate
SELECT 
      ROUND(COUNT( CASE When attrition_Flag ='Attrited Customer' then 1 END)*100.0/COUNT(*),2) AS Churn_rate
FROM [Churn Status];
```
## Result
![question 1](https://github.com/user-attachments/assets/70d9c69a-f433-400e-a30e-86c0d7f797b4)
## Insight:
The overall churn rate is approximately 16.07%, indicating a moderate level of customer attrition. This serves as a baseline metric to assess customer loyalty and retention performance.
 ## 2. Age Group with Highest Churn Rat
```SQL
----Step 1 Create Age Bracket of Customers
WITH Age_Bracket AS ( SELECT CS.Attrition_Flag
                             ,C.Client_Number
                             ,CASE WHEN C.Customer_Age BETWEEN 26 AND 36 THEN '26-36'
						  WHEN C.Customer_Age BETWEEN 37 AND 47 THEN '37-47'
						     WHEN C.Customer_Age BETWEEN 48 AND 58 THEN '48-58'
							   ELSE '59-73'
						END AS Age_Group
					FROM Customers C
					   INNER JOIN [Churn Status] CS ON CS.Client_Number= C.Client_Number                    
)
--- Step 2 Aggregate By Age Group
SELECT Age_Group
       ,COUNT(*) AS Total_Customer
,ROUND(COUNT(CASE WHEN Attrition_Flag = 'Attrited Customer' THEN 1 END)*100.00/COUNT(*),2 ) AS Churned_Rate 
   FROM Age_Bracket
GROUP BY Age_Group
ORDER BY Total_Customer DESC;
```
## Result:
![question 2](https://github.com/user-attachments/assets/51c6ef6d-8a07-4ca0-9eb9-c71f08d2b3ee)
## Insight:
Customers aged 48–58 exhibit the highest churn rate (16.54%), suggesting this age group is more likely to leave the bank and should be prioritized in retention strategies.
## 3. Churn by Income Category
```SQL
--- Analysis 3 Is Income Category Of Attrited Customers An Indicator to churn ?
-- This query calculates churn and retention rates by income category
-- and the total credit limit held by attrited customers.
-- Insight helps assess financial risk associated with churn by income group.
SELECT A.Income_Category
      ,ROUND(COUNT(CASE WHEN CS.Attrition_Flag = 'Attrited Customer' THEN 1 END)*100.00/COUNT(*),1 
	  ) AS Churned_Rate

       ,ROUND(COUNT(CASE WHEN CS.Attrition_Flag = 'Existing Customer'  THEN 1 END)*100.00/COUNT(*),1 
	   ) AS Existing_Customer_Rate

	   ,FORMAT(SUM(CASE WHEN CS.Attrition_Flag='Attrited Customer' THEN A.Credit_Limit ELSE 0 END),'C','NG-US' 
	    ) AS Total_Credit_Limit
	  FROM Account A
	     INNER JOIN [Churn Status] CS ON A.Client_Number=CS.Client_Number

	  GROUP BY A.Income_Category
	     ORDER BY Churned_Rate DESC;
```
## Result:
![question 3](https://github.com/user-attachments/assets/b692b350-c273-47df-86d1-d4505817951d)
## Insights
Customers within the $120K+ and Less than $40K income brackets exhibit higher churn rates. This suggests that both high-income customers (possibly seeking more premium services) and low-income customers (potentially facing financial strain) are at risk, highlighting the need for tailored retention strategies across both ends of the income category.
## 4. Impact of Inactivity on Churn
```SQL
--Analysis 4 Is Customer Inactivity a leading Indicator to Churn ?
--This query analyzes whether the number of inactive months correlates with churn likelihood.

SELECT  A.Months_Inactive_12_mon
      ,COUNT(CASE WHEN CS.Attrition_Flag = 'attrited customer' THEN 1 END 
	    ) AS Churned_Customers

	  ,ROUND(COUNT(CASE WHEN CS.Attrition_Flag = 'attrited customer' THEN 1 END) *100.00/COUNT(*),2 
	   ) AS Churn_Rate

	 ,COUNT(CASE WHEN CS.Attrition_Flag = 'Existing Customer' THEN 1 END
	   ) AS  Existing_Customers

	  ,ROUND(COUNT(CASE WHEN CS.Attrition_Flag = 'Existing Customer' THEN 1 END) *100.00/COUNT(*),2
	   ) AS Existing_customer_Rate

FROM Activities A
   INNER JOIN [Churn Status] CS ON A.Client_Number=CS.Client_Number

GROUP BY Months_Inactive_12_mon
   ORDER BY Months_Inactive_12_mon;
```
## Result:
![question 4](https://github.com/user-attachments/assets/f8a75b00-9a5d-40e4-862a-bd81469f6b6c)
## Insight
Interestingly, both highly active customers 0 inactive months, churn rate 51.72% and inactive customers 4+ months, churn rate 29.89% exhibit high churn. This suggests churn risk is not isolated to inactivity alone extreme engagement levels (either end) require closer attention.




