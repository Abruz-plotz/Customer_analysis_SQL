# Customer Churn Analysis Project(using SQL)

<details>
  <summary> <h2>  A) Inroduction </h2> </summary>

This project focuses on E-Commerce Customer Churn Analysis to understand customer behavior and identify key factors influencing churn. Using SQL, the dataset was cleaned, transformed, and analyzed to uncover insights into customer preferences, purchase patterns, and retention strategies. The analysis helps businesses improve customer engagement, reduce churn rates, and enhance overall satisfaction.
</details>

<details>
  <summary> <h2> B) Dataset </h2> </summary>
	

The dataset contains 5630 rows and 20 columns.The MySQL Workbench shows only 1000 rows.

```sql
select * from customer_churn;
```
#### Output 

##### B1) Using Python 🟢🟢:- 

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/First_Output_Py.png)
<!-- Note :- ![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/blob/main/Scrnshts/First_Output_Py.png) does not work since "blob/" does
not work with "raw.githubusercontent"-->

##### B2) MySQL output link 🟢🟢:-

<!--<div style="margin: 10px 0;">
  <a href="https://github.com/Abruz-plotz/Customer_analysis_SQL/blob/main/Customer_Analysis.csv" target="_blank"
     style="
       display: inline-block;
       background-color: #4CAF50;
       color: white;
       padding: 10px 20px;
       text-decoration: none;
       border-radius: 5px;
       font-weight: bold;
     ">
    📁 Click to view MySQL Workbench Output(CSV)
  </a>
</div>-->

<a href="https://github.com/Abruz-plotz/Customer_analysis_SQL/blob/main/Customer_Analysis.csv">
  📁  Click to view the original dataset (CSV)
</a><br>

*****Remarks:- This is the original dataset before Data preprocessing.*****
</details>

<details>
  <summary> <h2> C) Data Cleaning </h2> </summary>
	

##### C1) Handling missing values 🟢🟢

```sql
set sql_safe_updates = 0;

update customer_churn
join (select round(avg(WarehouseToHome),0) as avg_WarehouseToHome from customer_churn)
a on 1=1 set customer_churn.WarehouseToHome = a.avg_WarehouseToHome
where customer_churn.WarehouseToHome is null;

update customer_churn
join (select round(avg(HourSpendOnApp),0) as avg_HourSpendOnApp  from customer_churn)
a on 1=1 set customer_churn.HourSpendOnApp = a.avg_HourSpendOnApp
where customer_churn.HourSpendOnApp is null;
    
update customer_churn
join (select round(avg(OrderAmountHikeFromlastYear),0) as avg_OrderAmountHikeFromlastYear from customer_churn)
a on 1=1 set customer_churn.OrderAmountHikeFromlastYear=a.avg_OrderAmountHikeFromlastYear
where customer_churn.OrderAmountHikeFromlastYear is null;
    
update customer_churn
join (select round(avg(DaySinceLastOrder),0) as avg_DaySinceLastOrder from customer_churn)
a on 1=1 set customer_churn.DaySinceLastOrder=a.avg_DaySinceLastOrder
where customer_churn.DaySinceLastOrder is null;
    
update customer_churn
set Tenure = (select Tenure from
(select Tenure,count(Tenure) as value_occurrence from customer_churn group by
Tenure order by value_occurrence desc limit 1) as mode_Tenure)
where Tenure is null;   
    
update customer_churn
set CouponUsed = (select CouponUsed from
(select CouponUsed,count(CouponUsed) as value_occurrence from customer_churn group by
CouponUsed order by value_occurrence desc limit 1) as mode_CouponUsed)
where CouponUsed is null;   
     
update customer_churn
set OrderCount = (select OrderCount from
(select  OrderCount,count(OrderCount) AS value_occurrence from customer_churn group by
OrderCount order by value_occurrence desc limit 1) as mode_OrderCount)
where OrderCount is null;
```
*****Remarks:- The null values in numerical columns like WarehouseToHome, HourSpendOnApp, OrderAmountHikeFromlastYear, DaySinceLastOrder are imputed using average
and the null values in categorical columns like Tenure,CouponUsed,OrderCount are imputed using mode.*****



##### C2) Handling outliers 🟢🟢

```sql
Select * from customer_churn where WarehouseToHome > 100;
delete from customer_churn where WarehouseToHome > 100;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Dist_100_todelete.png)

*****Remarks:- There are 2 outliers present.After deleting both rows the resulting dataset has dimension 5,628 X 20.*****


</details>

<details>
  <summary> <h2> D) Dealing with Inconsistencies </h2> </summary>



```sql   
   update customer_churn 
   set PreferredPaymentMode = case
   when PreferredPaymentMode = 'cc' then 'Credit Card'
   when PreferredPaymentMode = 'cod' then 'Cash on Delivery'
   else PreferredPaymentMode
   end,
   PreferredLoginDevice = case
   when PreferredLoginDevice = 'Phone' then 'Mobile Phone'
   else PreferredLoginDevice
   end,
   PreferedOrderCat = case 
   when PreferedOrderCat = 'Mobile' then 'Mobile Phone'
   else PreferedOrderCat 
   end;
   
Select * from customer_churn;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Mobile_Phone.png)

*****Remarks:- The Mobile/Phone in Preferred Login Device column is standardized as Mobile Phone,'cc' and 'cod' in PreferredPaymentMode 
is Normalized as 'Credit Card' and 'Cash On Delivery' respectively.*****


</details>

<details>
  <summary> <h2> E) Data Transformation: </h2> </summary>


##### E1) Column Renaming and Creating New Columns 🟢🟢

```sql 
alter table customer_churn
rename column PreferedOrderCat to PreferredOrderCat,
rename column HourSpendOnApp to HoursSpentOnApp,
add column ComplaintReceived varchar(50),
add column ChurnStatus varchar(50);

update customer_churn 
set ComplaintReceived  = case
when Complain = 1 then 'Yes'
else 'No'
end,
ChurnStatus = case
when churn = 1 then 'Churned'
else 'Active'
end;
```
*****Remarks:- The column names of 2 columns are corrected and updated the columns 'ComplaintReceived' and 'ChurnStatus' to display descriptive text (“Yes”/“No”, “Churned”/“Active”) instead of numeric indicators (1/0).*****

 

##### E2) Column Dropping 🟢🟢

```sql
alter table customer_churn
drop column churn,
drop column complain;

select * from customer_churn;
```
![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/E_Results.png) 

##### E3) MySQL output link 🟢🟢

<a href="https://github.com/Abruz-plotz/Customer_analysis_SQL/blob/main/Custmr_Anlys_AfterPreProcess.csv">
  📁 Click to view the complete Preprocessed Data (CSV)
</a><br>

*****Remarks:-Before analysing the dataset, the data was cleaned, transformed, reduced, and integrated to ensure it was error-free. Two outliers (CustomerID: 51310 and 54125) were removed, as their distances from the warehouse were unusually high. The final dataset now has dimensions of 5,628 × 20.***** 

<br><br><br>
</details>

<details>
  <summary> <h2> F) Data Exploration and Analysis </h2> </summary>
	

**F1) Analysis No.1 🟧🟧 :-** Count of churned and active customers from the dataset.

```sql 
select  sum(case when churnStatus = 'Churned' then 1 else 0 end) as Count_of_Churned,
		 sum(case when churnStatus = 'Active' then 1 else 0 end) as Count_of_Active 
from customer_churn;
```


![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_1.png)

*****Observation:- After removing the outliers, there are 4680 active customers and 948 churned customers.*****

<br><br><br>
**F2) Analysis No.2 🟧🟧 :-** Average tenure and total cashback amount of customers who churned.

```sql
select avg(Tenure),sum(CashbackAmount) as Total_Cashback,count(CashbackAmount) as Number_of_customers
from customer_churn
where churnStatus = 'Churned';
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Avg_tenure.png)

*****Observation:- The average tenure of churned customers is approximately 3.18 years, indicating that most churners were not long-term users.*****

<br><br><br>
**F3) Analysis No.3 🟧🟧 :-** The percentage of churned customers who complained.

```sql 
Select 
cast((Select count(*) from customer_churn
where churnStatus = 'Churned' and ComplaintReceived = 'Yes') as DECIMAL(10, 2)) * 100 / 
(select count(*) from customer_churn
where churnStatus = 'Churned') as Percentage_of_Churned_Customers_complained;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Per_complnd.png)

*****Observation:-This indicates that over half of the customers who stopped using the service had complained earlier.*****

<br><br><br>
**F4) Analysis No.4 🟧🟧 :-** City tier with the highest number of churned customers whose preferred order category is "Laptop & Accessory".

```sql
  SELECT CityTier,
  COUNT(CustomerID) as Churned_Customer_count FROM customer_churn
  where churnStatus = 'Churned' and PreferredOrderCat='Laptop & Accessory'
  GROUP BY CityTier
  ORDER BY Churned_Customer_count DESC limit 1;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_4.png)

*****Observation:- Among customers who purchased 'Laptop & Accessory', the majority(150) are from City Tier 3, accounts for approximately 15.82% of total churned customers.*****

<br><br><br>
**F5) Analysis No.5 🟧🟧 :-** The most preferred payment mode among active customers.

```sql  
  select PreferredPaymentMode as PreferredPaymentMode_AmongActive, count(CustomerID) AS No_of_Customers_Used from customer_churn
  where churnStatus = 'Active'
  group by PreferredPaymentMode
  order by No_of_Customers_Used DESC limit 1;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_5.png)

*****Observation:- Out of 4680 active customers, 1956 users use debit card which makes it 41.79% of total payment methods. Hence, it is the most preffered payment method.*****

<br><br><br>
**F6) Analysis No.6 🟧🟧 :-** Total order amount hike for all customers who are single and prefer mobile phones for ordering.

```sql
Select avg(OrderAmountHikeFromlastYear) as Total_Order_Amount_Hike from customer_churn
where MaritalStatus='Single' and PreferredOrderCat='Mobile Phone';
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_6_Crct.png)

*****Observation:- There is a 15.65% hike in mobile purchase by single users compared to last year,which highlights a growing spending trend in this segment.*****

<br><br><br>
**F7) Analysis No.7 🟧🟧:-** Average number of devices registered among customers who used UPI as their preferred payment mode.

```sql  
Select avg(NumberOfDeviceRegistered) from customer_churn
where PreferredPaymentMode = 'UPI';
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_7.png) 
 
*****Observations:- Customers who prefer UPI as their payment mode have an average of 3.72 devices registered.This indicates that these customers tends to use multiple devices(probably more than 3 devices) for shopping.*****

<br><br><br>
**F8) Analysis No.8 🟧🟧 :-** Determine the city tier with the highest number of customers.

```sql 
Select CityTier,
COUNT(CustomerID) as Customer_count from customer_churn
GROUP BY CityTier
ORDER BY Customer_count DESC limit 1;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_8.png)

*****Observations:- Out of 5628 customers(active and churned),3666 resides in city tier 1, accounting for approximately 65% of the total customer base.It is likely due to better accessibility, shorter delivery distances, and higher digital adoption.*****

<br><br><br>
**F9) Analysis No.9 🟧🟧 :-** Gender that utilized the highest number of coupons. 

```sql
Select Gender, 
sum(CouponUsed) as Utilized_coupon_count from customer_churn
group by Gender
ORDER BY Utilized_coupon_count DESC limit 1;
``` 

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_9.png)

*****Observation:- Male customers utilized a total of 5,629 coupons, indicating that they are the most responsive to discounts and coupon-based marketing strategies compared to other gender groups.*****

<br><br><br>
**F10) Analysis No.10 🟧🟧 :-** Number of customers and the maximum hours spent on the app in each preferred order category.

```sql
Select PreferredOrderCat,count(CustomerID) as No_of_Customers,
Sum(HoursSpentOnApp) as Max_hours_Spent from customer_churn
group by PreferredOrderCat
order by PreferredOrderCat,No_of_Customers,Max_hours_Spent;
``` 

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_10.png)

*****Observations:- This indicates that app engagement is directly proportional to number of customers. The Mobile Phone category has the highest customer count (2,078) and total app usage (6,250 hours), while Grocery shows the lowest engagement (410 customers, 1,157 hours).*****

<br><br><br>
**F11) Analysis No.11 🟧🟧 :-** Total order count for customers who prefer using credit cards and have the maximum satisfaction score.       

```sql
Select Sum(OrderCount) as Total_Order_Count,count(OrderCount) as No_Of_Customers from customer_churn
where PreferredPaymentMode = 'Credit Card' 
and SatisfactionScore =(Select max(SatisfactionScore) from customer_churn);
 ```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_11.png)

*****Observations:- High satisfaction levels are positively correlated with higher engagement among Credit Card users. This highly satisfied group, which makes repeat purchases, represents a loyal customer base for the business. Hence, Credit Card buyers can be seen as an indicator of customer trust and convenience.*****

<br><br><br>
**F12) Analysis No.12 🟧🟧 :-** Average satisfaction score of customers who have complained?   

```sql   
Select avg(SatisfactionScore) as Average_Satisfaction_Score
from customer_churn where ComplaintReceived = 'Yes';
 ``` 

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_12.png)

*****Observations:- The average satisfaction score of around 3 shows the customers who complained are unhappy with their experience.From Analysis 3,We already know that half of the customers churned had complained. Hence, dissatisfaction is a strong early indicator of churn risk. By addressing and resolving complaints faster, the business could potentially reduce churn and improve satisfaction.*****

<br><br><br>
**F13) Analysis No.13 🟧🟧 :-** Most Ordered category among customers who used more than 5 coupons.

```sql
Select PreferredOrderCat, COUNT(*) AS CustomerCount, SUM(CouponUsed) AS TotalCouponsUsed
from customer_churn
where CouponUsed > 5
group by PreferredOrderCat
order by CustomerCount DESC;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_13_Crctd.png)

*****Observations:- All categories have customers who used more than 5 coupons.The 'Laptop & Accessories' has the most customers(99) but lower per-customer coupon usage.Meanwhile,the 'grocery' has only 42 customers who used more than 5 coupons but it has the highest average coupon usage per customer with an average of 9.4. Smaller categories like 'Others' have fewer customers but still relatively high average coupon usage.<br><br>
The analysis suggests that customers tend to use more coupons for essentials like groceries compared to luxury items such as laptops and fashion products.This could reflect a middle-class purchasing mindset of those customers.*****

<br><br><br>
**F14) Analysis No.14 🟧🟧 :-** Top 3 preferred order categories with the highest average cashback amount.

```sql
Select PreferredOrderCat as Top_3_category,(select avg(CashbackAmount)) as Average_Cashback from customer_churn
group by PreferredOrderCat
order by Average_Cashback DESC limit 3;
 ``` 
<!-- /* select CustomerID from customer_churn
  group by CustomerID
  having sum(OrderCount) > 500; */-->
  

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_14.png) 
 
*****Observations:- Continuing our analysis 13 here, it is clear that the 'Grocery' category continues to show high engagement with savings-oriented incentives, whether in the form of coupons or cashback.The ‘Others’ category leading in cashback.*****

<br><br><br>
**F15) Analysis No.15 🟧🟧 :-** The preferred payment modes of customers under some condition  
 
 ```sql
 Select PreferredPaymentMode
 from customer_churn
 group by  PreferredPaymentMode
 having avg(Tenure) > 10
 and sum(OrderCount) > 500;        
 ```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_15.png)

*****Observations:- This indicates that E-Wallet users are both loyal and active, showing strong repeat purchase behavior over time.The longer tenure suggests trust and satisfaction with this payment method.*****

<br><br><br>

**F16) Analysis No.16 🟧🟧 :-** Customer’s order details under some conditions. 

```sql
Select  CustomerID,PreferredOrderCat,OrderAmountHikeFromlastYear,DaySinceLastOrder,CityTier,MaritalStatus,OrderCount
from customer_churn
group by  CustomerID
having MaritalStatus='Married' 
and CityTier = 1 and OrderCount > (select avg(OrderCount) from customer_churn);-- 2.5533;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_17_MySQL.png)

*****Observations:-Married customers in Tier-1 cities are a core high-value group since they have above-average purchase frequency that increases year by year with most orderCount values of customers are between 6 and 15. The tech & lifestyle products lead their spending as their the most frequent PreferredOrderCat are 'Laptop & Accessories', 'Mobile Phone' and 'Fashion'.*****
<br><br><br>
</details>

<details>
  <summary> <h2> G) Combining new table with existing one </h2> </summary>

##### G1) Creation and insertion of data into new table 🟢🟢

```sql
Create table customer_returns( ReturnID  INT PRIMARY KEY,CustomerID INT, 
                              ReturnDate date, RefundAmount int);
                              
Insert into customer_returns(ReturnID,CustomerID,ReturnDate,RefundAmount)
                     values(1001,50022,'2023-01-01',2130),
						   (1002,50316,'2023-01-23',2000),
                           (1003,51099,'2023-02-14',2290),
                           (1004,52321,'2023-03-08',2510),
						   (1005,52928,'2023-03-20',3000),
                           (1006,53749,'2023-04-17',1740),
                           (1007,54206,'2023-04-21',3250),
						   (1008,54838,'2023-04-30',1990);
 
select * from customer_returns;
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_18.png)

##### G2) Linking both tables 🟢🟢

```sql
 Select customer_returns.*,customer_churn.churnStatus,ComplaintReceived
 from customer_churn join customer_returns
 on customer_churn.CustomerID = customer_returns.CustomerID
 where customer_churn.churnStatus = 'Churned' and customer_churn.ComplaintReceived = 'Yes';     
```

![Result](https://raw.githubusercontent.com/Abruz-plotz/Customer_analysis_SQL/main/Scrnshts/Ans_19.png)


</details>

<details>
  <summary> <h2> H) Project Summary & Insights </h2> </summary>

The E-Commerce Customer Churn Analysis provided valuable insights into customer behavior, loyalty patterns and churn indicators using SQL-based data cleaning, transformation, and analytical techniques.After analysis we got some key insights as:-

**1)📌** There are 4680 active customers and 948 churned ones. The active customers significantly outnumber churned ones, but complaints and low satisfaction scores were found to be strong early **indicators of churn risk**. Adressing complains faster can reduce the churn risk.

**2)📌** Married customers from Tier-1 cities show the highest engagement levels, longer tenure, and consistent year-over-year order growth — representing a loyal, high-value segment.

**3)📌** E-Wallet and Debit Card users demonstrate the most consistent and active purchase behavior, reflecting digital adoption and trust in these payment modes.

**4)📌** For essential categories like **Grocery**,customers are highly responsive to savings incentives using Coupons and cashbacks.Hence, Such offers consistently benefits both customers and the business.

**5)📌** Laptop & Accessories, Mobile Phones, and Fashion dominate as **preferred** categories among high-spending customers

**6)📌** A positive correlation between complaints, churn, and product returns was observed.

### I) Action Plan 

**1)⚡⚡** **Enhancing customer satisfaction** through faster complaint resolution could directly reduce churn.

**2)⚡⚡** **Expanding cashback and coupon programs** for essential and high-engagement categories like groceries can drive repeat purchases.

**3)⚡⚡** Personalized retention strategies for loyal Tier-1, married customers can strengthen brand loyalty.

**4)⚡⚡** The **promotion of digital payment incentives** (E-Wallet, Debit Card) can further improve convenience and customer lifetime value.
