# 📊 Exploratory Data Analysis (EDA) on Telecommunication Customer Churn using MySQL

This project showcases my SQL-based Exploratory Data Analysis (EDA) skills using a customer churn dataset from a telecommunication company. The goal was to identify patterns in customer behavior, segment churn trends by contract types, and derive actionable insights for improving customer retention.

## 🎯 Project Objective
As part of my data analytics learning, I explored a telecom dataset using MySQL to understand why customers leave the service (churn). I focused on contract types to see which group had the highest churn rate and how retention strategies could be improved.

## 📂 Dataset Summary
Realistic customer data from a telecom company

Includes fields such as:

* customerID
* Service Availability
* Contract (Month-to-month, One year, Two year)

* MonthlyCharges, Churn, tenure, etc.

## ⚙️ Tools & Technologies Used
* MySQL for data querying and transformation

* CTEs (Common Table Expressions) for modular analysis

* JOINs and Aggregations for combining insights

## 🔍 My Analysis Approach
I divided my SQL analysis into multiple logical steps using CTEs to make it clean and efficient.

✅ Step-by-step SQL Logic:
* Counted total number of customers per Contract type.

* Calculated how many customers churned (Churn = 1) and how many were retained (Churn = 0).

* Used LEFT JOINs to merge CTEs into a single output.

* Calculated churn rate (%) using SQL arithmetic.
* Summarized the output into a table for insights.

| Contract Type  | Total Customers | Churned Customers | Retained Customers | Churn Rate (%) |
| -------------- | --------------- | ----------------- | ------------------ | -------------- |
| Month-to-month | 3875            | 1655              | 2220               | 42.73%         |
| One year       | 1470            | 174               | 1296               | 11.83%         |
| Two year       | 1695            | 131               | 1564               | 7.73%          |


## 📸 MySQL Screenshot

Below is the actual result of my SQL query executed in MySQL Workbench. It shows remove duplicate fro data celaning, total customers,phone service availablity,Multiple Line Availability,customer and their totalchagers with respect to contract, churned customers, retained customers, and churn rate by contract type:

### check duplicate value:
```mysql
# check dulipcate customer id 
select customerID, count(customerID) from telco_customer_data
group by customerID
having count(customerID)>1
```
<img width="222" height="118" alt="t1" src="https://github.com/user-attachments/assets/60b804fc-cbaa-415e-812c-2255bad5333e" />

For delete the duplicate customerID, first i add a temp_id column and set as primary_key
step1:
add a temp_id column
```mysql
ALTER TABLE telco_customer_data ADD COLUMN temp_id INT NOT NULL AUTO_INCREMENT PRIMARY KEY;
```

--- step2: delete the duplicate customerID with the help of temp_id

```mysql
WITH cte AS (
  SELECT temp_id, customerID,
         COUNT(customerID) OVER (PARTITION BY customerID) AS count_customerID,
         ROW_NUMBER() OVER (PARTITION BY customerID ORDER BY temp_id) AS rn
  FROM telco_customer_data
)
DELETE FROM telco_customer_data
WHERE temp_id IN (
    SELECT temp_id FROM cte WHERE rn > 1
);
```
<img width="211" height="113" alt="t2" src="https://github.com/user-attachments/assets/80eb1abd-5ed5-474b-a744-684049631c4a" />

Now delete the temp_id table
```
ALTER TABLE telco_customer_data DROP COLUMN temp_id;
```

### count of unique customerID
```
select count(customerID) from telco_customer_data
```
<img width="138" height="105" alt="t3" src="https://github.com/user-attachments/assets/d8bc3191-25eb-4b01-b401-59f8ac0c8b96" />

### count of phone service availability
```
select  count(PhoneService) as count_phone_service,
       case 
       when PhoneService=1 then 'Phone Service Available'
       when PhoneService=0 then 'Phone service not available'
end as phone_service_availability
from telco_customer_data
group by phone_service_availability;
```
<img width="295" height="80" alt="t4" src="https://github.com/user-attachments/assets/cdc413a1-3fbb-453e-b44d-86b8b8226fdb" />


### Multiple Line Availability
```
SELECT 
	COUNT(MultipleLines) AS count_multiple_lines,
	CASE 
		WHEN MultipleLines = 1 THEN 'Multiple Available'
		WHEN MultipleLines = 0 THEN 'Multiple Lines Not Available'
        END AS multiple_lines_availability
    FROM telco_customer_data
    GROUP BY multiple_lines_availability;
```

<img width="293" height="67" alt="t5" src="https://github.com/user-attachments/assets/0a5bd8e8-1914-4e61-937d-93f1f21f7052" />

### customer which internet service they used 
```
select distinct InternetService, count(customerID) as count_customer from telco_customer_data
group by InternetService;
```

<img width="217" height="82" alt="t6" src="https://github.com/user-attachments/assets/07406f24-2b8a-41bc-9160-f48ea08bb33a" />

### customer and their totalchagers with respect to contract
```
select contract, count(customerID) as total_customer, round(sum(TotalCharges),0) as total_contract_charges from telco_customer_data
group by contract
order by total_contract_charges desc;
```
<img width="341" height="95" alt="t7" src="https://github.com/user-attachments/assets/8d2c1f88-b911-479d-acc1-666dbe215bc9" />

### customer their payment method and average monthlycharges
```
select PaymentMethod, count(customerID), round(avg(MonthlyCharges),2) from telco_customer_data
group by PaymentMethod
```
<img width="443" height="106" alt="t8" src="https://github.com/user-attachments/assets/ec3ce158-19e7-4cae-93f9-73b7550ead57" />

### Customer and their retentation rate
```
WITH 
cte_1 AS (
    SELECT COUNT(customerID) AS total_customer FROM telco_customer_data
),
cte_2 AS (
    SELECT COUNT(customerID) AS leave_customer 
    FROM telco_customer_data
    WHERE Churn = 1
),
cte_3 AS (
    SELECT COUNT(customerID) AS retain_customer 
    FROM telco_customer_data
    WHERE Churn = 0
)
SELECT 
    (leave_customer / total_customer) * 100 AS churn_rate,
    (retain_customer / total_customer) * 100 AS retention_rate
FROM cte_1, cte_2, cte_3;
```
<img width="188" height="63" alt="t9" src="https://github.com/user-attachments/assets/787ee5fe-8dda-436c-97d2-ada994012f7c" />

### customer retentation rate on contract bases
```
WITH 
cte_total AS (
    SELECT contract, COUNT(customerID) AS total_customer 
    FROM telco_customer_data
    GROUP BY contract
),
cte_leave AS (
    SELECT contract, COUNT(customerID) AS leave_customer 
    FROM telco_customer_data
    WHERE Churn = 1
    GROUP BY contract
),
cte_retain AS (
    SELECT contract, COUNT(customerID) AS retain_customer 
    FROM telco_customer_data
    WHERE Churn = 0
    GROUP BY contract
)

SELECT 
    total.contract,
    COALESCE(leave_customer, 0) / total_customer * 100 AS churn_rate,
    COALESCE(retain_customer, 0) / total_customer * 100 AS retention_rate
FROM cte_total AS total
LEFT JOIN cte_leave AS `leave` ON total.contract = `leave`.contract
LEFT JOIN cte_retain AS retain ON total.contract = retain.contract;
```


<img width="283" height="91" alt="t10" src="https://github.com/user-attachments/assets/aa6ec748-d58d-4231-885a-9e57d50c765f" />

## 📈 Insights & Findings
* Month-to-month contract customers have the highest churn rate — over 42%

* Customers on One year and Two year contracts have significantly lower churn

* Customers with long-term contracts are more loyal, possibly due to commitment or discounted plans

## 💡 Business Impact
* Telecom companies can focus retention efforts on month-to-month customers

* Offering discounts, loyalty programs, or upgrades might reduce churn

* Strategic segmentation like this helps improve customer lifetime value


🙋‍♂️ About Me
My name is Asif Ali, and I'm passionate about data analytics in business and telecom environments.
This project reflects my ability to apply SQL for real-world data analysis and support strategic decision-making.

