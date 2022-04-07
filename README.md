# Power BI Dynamic M Query parameters.
Example of Dynamic M query parameters in Power BI and BigQuery

In February 2022, update Power Bi implemented Dynamic M query parameters in DirectQuery connections (previously only supported in Import). With Dynamic M Query Parameters, model authors can let reports use slicers to set the value or values for an M Query Parameters. 

If you want review the oficial documentation please qlik [here](https://docs.microsoft.com/en-us/power-bi/connect-data/desktop-dynamic-m-query-parameters).

## Prepare data in Google BigQuery.

I created two tables, OrderSales which contains the details of each order, and Payments, which includes the sales order key, date of payment, and amount. 

Sales Orders
![SalesOrderTable](https://drive.google.com/uc?export=view&id=1r7I_iI1urqxLuNQInZS2f8Zqz0_SLFIJ)

Payments
![PaymentsTable](https://drive.google.com/uc?export=view&id=1z9775n6bS7fGmnXzeoI7t7OwKFw8ezx-)



