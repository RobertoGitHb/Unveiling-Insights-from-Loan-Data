# Unveiling Insights from Loan Data

## Scenario :
In this case scenario I am a Data Analyst being approched by a financial company called ABC Financial Services. 

## The Company :
ABC Financial Services is a leading financial institution that offers a wide range of loan products, including personal loans, home purchase loans, and education loans.

# ASK
## Business Task :
ABC Financial Services has observed fluctuations in their loan default rates and has tasked me with conducting a comprehensive analysis to understand the factors influencing these trends. The company is particularly interested in identifying high-risk customer segments and optimizing their loan approval criteria to minimize defaults and improve profitability.
Below the key questions to address that will serve as guideline for the business task:

                 1. Correlation Between Default Rate and Customer Age:
                    Investigate how the default rate varies across different age groups. 
                    Is there a specific age group that is more likely to default on loans?
   
                 2. Loan Requests and Customer Income Levels:
                    Analyze how loan requests vary according to customer income levels. 
                    Are higher-income customers requesting larger loans, and how does this impact their repayment behavior?

                 3. Loan Intents and Default Rates:
                    Examine whether different loan purposes, such as home purchase, education, or personal expenses, are associated with varying default rates. 
                    Which loan intents carry higher risks? 

                 4. Employment Duration as a Predictor of Loan Repayment:
                    Assess whether the duration of a customer’s employment is a significant predictor of their ability to repay loans. 
                    Does longer employment correlate with lower default rates?   

                 5. Relationship Between Customer Age and Loan Interest Rates:
                    Explore the relationship between customer age and the interest rates offered on loans. 
                    Are younger or older customers being offered different rates, and how does this affect their likelihood to default?
 
                 6. Identifying High-Risk Loan Intents:
                    Identify which specific loan intents are associated with the highest default rates. 
                    Are there particular types of loans that are consistently riskier, and what recommendations can be made to mitigate these risks?  

## Expected Outcome:
My analysis will provide ABC Financial Services with critical insights into the factors driving loan defaults. By understanding the relationships between customer demographics, loan characteristics, and repayment behavior, the company can refine its loan approval process, adjust interest rates, and tailor loan products to better align with customer risk profiles. Ultimately, this will help ABC Financial Services reduce default rates, optimize loan portfolios, and enhance overall financial performance.

# DATA SET DESCRIPTION
To perform this analysis, I utilized a dataset provided by ABC Financial Services. The dataset consists of 32,587 rows and 13 columns, capturing a wide range of information on the company’s loan portfolio. The data includes key variables such as customer demographics (age, income, employment duration), loan characteristics (amount, interest rate, loan intent), and repayment behavior (default status, payment history).

# Cleaning Process
First thing first I will create a copy of the original dataset
```
CREATE TABLE loanstage
LIKE `loandataset - loansdatasest`;

INSERT loanstage
SELECT * 
FROM `loandataset - loansdatasest`;
```

I will now check for duplicates and I will remove them
```
SELECT *,
ROW_NUMBER() OVER( 
PARTITION BY customer_id, customer_age, customer_income, home_ownership, employment_duration, loan_intent, loan_grade, loan_amnt, loan_int_rate, term_years, historical_default, cred_hist_length, Current_loan_status) AS row_numb
FROM loanstage;

WITH duplicate_cte AS 
(
SELECT *,
ROW_NUMBER() OVER( 
PARTITION BY customer_id, customer_age, customer_income, home_ownership, employment_duration, loan_intent, loan_grade, loan_amnt, loan_int_rate, term_years, historical_default, cred_hist_length, Current_loan_status) AS row_numb
FROM loanstage
)
SELECT *
FROM duplicate_cte
WHERE row_numb > 1;

CREATE TABLE `loanstage2` (
  `customer_id` int DEFAULT NULL,
  `customer_age` int DEFAULT NULL,
  `customer_income` int DEFAULT NULL,
  `home_ownership` text,
  `employment_duration` int DEFAULT NULL,
  `loan_intent` text,
  `loan_grade` text,
  `loan_amnt` text,
  `loan_int_rate` double DEFAULT NULL,
  `term_years` int DEFAULT NULL,
  `historical_default` text,
  `cred_hist_length` int DEFAULT NULL,
  `Current_loan_status` text, 
  `row_numb` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;


INSERT INTO loanstage2
  SELECT *,
ROW_NUMBER() OVER( 
PARTITION BY customer_id, customer_age, customer_income, home_ownership, employment_duration, loan_intent, loan_grade, loan_amnt, loan_int_rate, term_years, historical_default, cred_hist_length, Current_loan_status) AS row_numb
FROM loanstage;

SELECT * 
FROM loanstage2
WHERE row_numb > 1;


DELETE 
from loanstage2
WHERE row_numb > 1;

ALTER TABLE loanstage2
DROP COLUMN row_numb;
```

Customer age looks for some records above 100 years. I will delete these records. 
```
SELECT DISTINCT customer_age
FROM loanstage2
WHERE customer_age >= 84
GROUP BY 1;
```

Other records shows customer age as 3, 6, and 8 (3 records). I will also delete them 
```
SELECT DISTINCT customer_age, count(*) as count
FROM loanstage2
group by customer_age
ORDER BY 1;
```

```
DELETE
FROM loanstage2
WHERE customer_age = 8;
```

4 records shows customer age between 123 and 144. It is clearly an unreal age number therefore I will delete them
```
SELECT * 
FROM loanstage2
WHERE customer_age > 99;
```
```
DELETE
FROM loanstage2
WHERE customer_age > 99;
```

Some values in the "historical_default" column are blank
```
SELECT * 
FROM loanstage2
WHERE historical_default = '';
```

I will then populate them with "Unknown"
```
UPDATE loanstage2
SET historical_default = 'Unknown'
WHERE historical_default = '';
```

The "Loan_amount" column has a special character before the currency character. I will fix it removing it
```
SELECT loan_amnt, TRIM(LEADING 'Â' FROM loan_amnt)
FROM loanstage2;
UPDATE loanstage2
SET loan_amnt = TRIM(LEADING 'Â' FROM loan_amnt);
```

# Analysis
## Is there a specific age group that is more likely to default on loans?
```
SELECT 
case
    WHEN customer_age >=20 AND customer_age <=30 THEN '20 - 30'
    WHEN customer_age >=31 AND customer_age <=60 THEN '31 - 60'
    WHEN customer_age >=61 AND customer_age <=99 THEN '61 - 99'
END AS customer_age_groups, current_loan_status,
count(*) AS Count
FROM loanstage2
where Current_loan_status = 'default'
GROUP BY  customer_age_groups, Current_loan_status
ORDER BY  customer_age_groups;
```

## Are higher-income customers requesting larger loans... 
```
SELECT 
    CASE 
        WHEN customer_income < 59999 THEN 'Low'
        WHEN customer_income BETWEEN 60000 AND 99999 THEN 'Medium'
        ELSE 'High'
    END AS income_level,
    COUNT(*) AS number_of_customers
FROM 
    loanstage2
GROUP BY 
    income_level;
    select * from Loanstage2;
```

##... and how does this impact their repayment behavior?
```
SELECT 
case
    WHEN customer_income >=4000 AND customer_income <=30000 THEN '4000 - 30000'
    WHEN customer_income >=30001 AND customer_income <=60000 THEN '30001 - 60000'
    WHEN customer_income >=60001 AND customer_income <=80000 THEN '60001 - 80000'
    WHEN customer_income >=80001 AND customer_income <=120000 THEN '80001 - 120000'
    WHEN customer_income >=120001 AND customer_income <=2039784 THEN '120001 - 2039784'
END AS income_groups, Current_loan_status,
count(*) AS Count
FROM loanstage2
GROUP BY  income_groups, Current_loan_status
ORDER BY  income_groups;
```

## Which loan intents carry higher risks?
```
SELECT loan_intent, Current_loan_status, count(*) as count
FROM loanstage2
WHERE Current_loan_status = 'DEFAULT'
GROUP BY loan_intent, Current_loan_status
ORDER BY count DESC;
```

## Does longer employment correlate with lower default rates?
```
SELECT Current_loan_status, avg(employment_duration) AS avg_empl_duration
FROM loanstage2
GROUP BY Current_loan_status
ORDER BY avg_empl_duration desc;
```

## Are younger or older customers being offered different rates...
```
SELECT customer_age, AVG(loan_int_rate) as AVG_loan_int_rate
FROM loanstage2
WHERE customer_age >= 20 AND customer_age <=30
GROUP BY customer_age
ORDER BY AVG_loan_int_rate desc;

SELECT customer_age, AVG(loan_int_rate) as AVG_loan_int_rate
FROM loanstage2
WHERE customer_age >= 31 AND customer_age <=50
GROUP BY customer_age
ORDER BY AVG_loan_int_rate desc;
SELECT customer_age, AVG(loan_int_rate) as AVG_loan_int_rate
FROM loanstage2
WHERE customer_age >= 51 AND customer_age <=60
GROUP BY customer_age
ORDER BY AVG_loan_int_rate desc;

SELECT customer_age, AVG(loan_int_rate) as AVG_loan_int_rate
FROM loanstage2
WHERE customer_age >= 61 AND customer_age <=99
GROUP BY customer_age
ORDER BY AVG_loan_int_rate desc;
```

## ... and how does this affect their likelihood to default?
```
SELECT 
case
    WHEN customer_age >=20 AND customer_age <=30 THEN '20 - 30'
    WHEN customer_age >=31 AND customer_age <=60 THEN '31 - 60'
    WHEN customer_age >=61 AND customer_age <=99 THEN '61 - 99'
END AS customer_age_groups, current_loan_status,
count(*) AS Count
FROM loanstage2
where Current_loan_status = 'default'
GROUP BY  customer_age_groups, Current_loan_status
ORDER BY  customer_age_groups;
```

## Are there particular types of loans that are consistently riskier?
```
SELECT loan_intent, avg(loan_int_rate) AS avg_loan_int_rate
FROM loanstage2
GROUP BY loan_intent
ORDER BY avg_loan_int_rate desc;
```

# Sharing Insights with Stakeholders:
To effectively communicate my findings to the stakeholders at ABC Financial Services, I will be using Power BI to create interactive and visually engaging dashboards. Power BI allows for a clear and concise presentation of complex data, making it easier for stakeholders to understand key trends, correlations, and patterns identified in the analysis.

![Data set Loan Project Power BI](https://github.com/user-attachments/assets/eec316b9-5c94-45ec-bc42-586a9044fc8d)



