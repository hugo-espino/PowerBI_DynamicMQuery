# Power BI Dynamic M Query parameters.
Example of Dynamic M query parameters in Power BI and BigQuery

In February 2022 update, Power Bi implemented Dynamic M query parameters in DirectQuery connections (previously only supported in Import). With Dynamic M Query Parameters, model authors can let reports use slicers to set the value or values for an M Query Parameters. 

If you want to review the oficial documentation please clic [here](https://docs.microsoft.com/en-us/power-bi/connect-data/desktop-dynamic-m-query-parameters).

## Prepare data in Google BigQuery.

The objective of this report is that the final user can select a specific date to obtain the orders that have been created before the chosen date and have not been paid. It is a 100% dynamic report which calculate the values in real-time with a direct connection to Google Bigquery.  

In this case I created two tables, OrderSales which contains the details of each order, and Payments, which includes the sales order key, date of payment, and amount. 

Sales Orders

![PaymentsTable](https://drive.google.com/uc?export=view&id=1z9775n6bS7fGmnXzeoI7t7OwKFw8ezx-)

Payments

![SalesOrderTable](https://drive.google.com/uc?export=view&id=1r7I_iI1urqxLuNQInZS2f8Zqz0_SLFIJ)

I have a function table that receives a date parameter to calculate the orders on the chosen date. 

```
CREATE OR REPLACE TABLE FUNCTION proyecto1-344420.Sales.fnViewSalesOrders(pDate DATE) AS (
(
(
    (
    --Obtain records from the past, How were the figures in @date?
        SELECT A.SalesOrderKey,
		       A.CustomerKey,
			   A.SalesOrderNumber,
			   A.OrderDate,
			   A.DueDate,
			   CASE WHEN A.DueDate < pDate THEN DATE_DIFF(pDate, DueDate, DAY) ELSE 0 END AS DaysPastDue,
			   CASE WHEN A.DueDate < pDate THEN 
			        CASE WHEN DATE_DIFF( pDate, DueDate, DAY) > 30 THEN '>30'
					     WHEN DATE_DIFF(pDate, DueDate, DAY) > 20 THEN '20-29'
					     WHEN DATE_DIFF(pDate, DueDate, DAY) > 10 THEN '10-19'
					     WHEN DATE_DIFF(pDate, DueDate, DAY) > 0  THEN '1-9' ELSE '' END 
				ELSE 'On Time' END AS RangePastDue,
			    CASE WHEN A.DueDate <pDate THEN 
			        CASE WHEN DATE_DIFF(pDate, DueDate, DAY) > 30  THEN 5
					     WHEN DATE_DIFF(pDate, DueDate, DAY) > 20  THEN 4
					     WHEN DATE_DIFF(pDate, DueDate, DAY) > 10 THEN 3
					     WHEN DATE_DIFF(pDate, DueDate, DAY) > 0   THEN 2 ELSE 0 END 
				ELSE 1 END AS OrderRangePastDue,			            
			   TotalSalesAmount,
			   Sales.FnPayments(SalesOrderKey,pDate) AS PaidAmount, 
			   TotalSalesAmount - IFNULL(Sales.FnPayments(SalesOrderKey,pDate),0) AS BalanceReceivable			  
		FROM Sales.OrderSales A
		WHERE TotalSalesAmount !=0
		      AND (PaymentDate IS NULL OR PaymentDate > pDate) 
			  AND OrderDate <= pDate
    )
)
)
);
```

##Power Query

In Power Query connection it is necessary to create a new Date Parameter 

![NewParameter](https://drive.google.com/uc?export=view&id=1b8mKbEQ5Y-IHI9QDvlUi5tQjl1sWiu_U)

The next step is to replace the new parameter in the Power Query connection. 

![PQConnection](https://drive.google.com/uc?export=view&id=1UVydBJE-U0Q6DHCj2KjCST1RHqYcFTbW)

It is necessary to create a table in Power BI with the same data type in order to bind the parameter and a slicer or filter where the final user can control the instruction to BigQuery. In my case I created a calculate table with DAX

`DateParameters = CALENDAR(MIN(FactSalesOrder[OrderDate]),MAX(FactSalesOrder[OrderDate]))`

Finally the last step is in the model tab select the table field in the bind to parameter option. 
![BindParameter](https://drive.google.com/uc?export=view&id=17iSU7s7EAth8gsAX1SsNybrUJENBvJHP)

Reelevant links:
[Chris Webb's BI Blog](https://blog.crossjoin.co.uk/2020/10/25/why-im-excited-about-dynamic-m-parameters-in-power-bi/)
[Data Monkey Site](https://datamonkeysite.com/2022/03/08/first-look-at-dynamic-m-query-parameter-using-sql-server/)

