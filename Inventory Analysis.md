# Mint Classics Inventory Analysis

Mint Classics Company, a retailer of classic model cars and other vehicles, is looking to close one of its storage facilities. To make a data-based business decision, the company wants suggestions and recommendations for reorganizing or reducing inventory while maintaining timely customer service. As a data analyst, you will use MySQL Workbench to familiarize yourself with the general business by examining the current data and isolating and identifying those parts of the data that could be useful in deciding how to reduce inventory.

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Results and Findings](#results-and-findings)
- [Recommendations](#recommendations)

  
### Project Overview
---

Conducted exploratory analysis to investigate if there were any patterns or themes that may influence the reduction or reorganization of inventory in the Mint Classics storage facilities, and provided analytic insights and data-driven recommendations. 

### Data Sources

* [Mint Classics Dataset](https://d3c33hcgiwev3.cloudfront.net/2oM-7bPhQAK0DX4vqIQ_JQ_596ae3ede6934608af481acc56cb18f1_mintclassicsDB.sql?Expires=1701993600&Signature=dShvrtGJLsQD2jDNaa~4YrY9RMnItBt9vtQaxiN6PeFpDNgNMmnV3eu8q6RRgu66Mo9YmvipbRfHsgXCuLKOvraKCq7vbGuQN664xB5X8ot0~N-txScgkRcM5d2OYhUdoKy1jy6RCkTKQNX1afuYxThRPKe-klWcSlNfuyCuuf0_&Key-Pair-Id=APKAJLTNE6QMUY6HBC5A)
* [EER Diagram](https://github.com/phelpsbp/Projects/blob/main/Mint%20Classics%20Inventory%20Analysis/EER.jpg)

### Tools

Tools Used:
* MySQL Workbench - Data Analysis
* Excel - Pivot Tables and Visualizations
---
## Data Cleaning and Preparation

In the initial data preparation phase, we performed the following tasks:
1. Import the Classic Model Car Relational Database.
2. Examine the database structure via the EER to build familiarity with the business process.

## Exploratory Data Analysis

EDA involved exploring the products currently in inventory to answer key questions, such as:

- Which products have the lowest and highest order rates?
- Which product lines are most in demand?
- Which warehouses have the highest and lowest inventory stocks?
- Who are the top customers? 

## Data Analysis


### Order Frequencies
#### Individual Products
To start, let's look at the overall stock counts. Using the `mintclassics.products` and `mintclassics.orderdetails` tables, I'll take a look at the differences in inventory-to-order counts.

```sql
SELECT
	productCode,
	productName,
	quantityInStock,
	totalOrdered,
	(quantityInStock - totalOrdered) As inventoryDifference
FROM
	( SELECT
		prd.productCode,
        prd.productName,
        prd.quantityInStock,
        SUM(ord.quantityOrdered) as totalOrdered
	FROM 
		mintclassics.products as prd
    	LEFT JOIN 
		mintclassics.orderdetails as ord on prd.productCode = ord.productCode
	GROUP BY
		prd.productCode,
        prd.productName,
        prd.quantityInStock
	) as inventory_summary
WHERE 
	(quantityInStock - totalOrdered) > 0 
ORDER BY
	inventoryDifference DESC;
```
The query filters the results to show only those records where the inventory difference is greater than zero, indicating a positive difference (more in stock than ordered). The results are then ordered in descending order based on the inventory difference.

![top5_products_in_stock](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/bfd5de57-7efd-4342-9df6-e08b530271ac)



It looks like 2002 Suzuki XREO has the largest inventory difference, with 9997 currently in stock and only 1028 ordered overall. 
In the top 5 individual products with the highest stock-to-order difference, 4 of the products are cars and 1 is an airplane. 
Let's dig deeper into that. Are there any product lines that underperform or contribute to overstocking? 

#### Product Lines
Let's look at the product lines and their total inventories, quantity ordered, total earnings, and their stock-to-order ratios. 
```sql
SELECT
    	prd.productLine,
    	prl.textDescription as productLineDescription,
    	SUM(prd.quantityInStock) as inventoryTotal,
    	SUM(ord.quantityOrdered) as totalOrdered,
    	SUM(ord.priceEach * ord.quantityOrdered) as totalEarnings,
    	(SUM(ord.quantityOrdered) / SUM(prd.quantityInStock)) * 100 as inventoryDifferencePercentage
FROM
	mintclassics.products as prd
LEFT JOIN
	mintclassics.productlines as prl on prl.productLine = prd.productLine
LEFT JOIN
	mintclassics.orderdetails as ord on prd.productCode = ord.productCode
GROUP BY 
	prd.productLine, prl.textDescription
order by 
	inventoryDifferencePercentage desc;
```
The order of these results is based on the order frequencies of the product line as whole, helping to identify which have a higher percentage of excess inventory or unmet demand. 


![Screenshot 2023-12-14 113748](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/7fcb0a35-91b2-46e0-bae9-3631d8c1e42b)



It appears that while 4 of the 5 products with the highest amount of stock were classic cars, they are also the most ordered product line with significantly higher earnings than any other product line, indicating their inventory numbers are meeting demands. 
Ships and trains have very low order rates, with trains making up only 2.67% of all orders. The low amount of orders coupled with a 0.62% stock-to-order rate suggests that, despite the relatively low demand indicated by the quantity of orders, there is a possibility of an overstock of trains. 
<img src="https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/69ce3af4-86ee-4cd0-bcbe-f96299bfb1a7" width="425"/><img src="https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/9f159d52-baf7-4403-af3b-a5b7cae67925" width="425"/>
### Warehouses

Let's take a look at data related to product inventory across different warehouses. I'll be looking at which warehouses have both the highest and lowest inventories.
#### Individual Products in Each Warehouse

```sql
SELECT
	prd.productName,
    	wh.warehouseName,
    	SUM(prd.quantityInStock) AS inventoryTotal
FROM 
	mintclassics.products AS prd
JOIN
	mintclassics.warehouses AS wh ON prd.warehouseCode = wh.warehouseCode
GROUP BY 
	prd.productName, wh.warehouseName
ORDER BY
	inventoryTotal asc;
```
The results show a list of products along with their corresponding warehouse names and the total quantity in stock for each product in each warehouse. The ordering is based on the ascending total inventory, meaning that the products with the lowest inventory across all warehouses appear first.

| ![Warehouse_PivotTable](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/b8896a34-990d-44b3-977a-747e574b6b1c)| 
|:--:| 
| Pivot Table from the results of the query examining individual product stocks in each warehouse. The Pivot Table shows stock totals by warehouse |

It looks like the Southern Warehouses have the lowest inventory overall. 
To validate the pivot table findings, I'm going to run a query for warehouse stock totals overall.
#### Overall Stock Totals
```sql
SELECT
	wh.warehouseCode,
    	wh.warehouseName,
    	SUM(prd.quantityinStock) as stockTotal
FROM
	mintclassics.warehouses as wh
LEFT JOIN 
	mintclassics.products as prd on wh.warehouseCode = prd.warehouseCode
GROUP BY 
	wh.warehouseCode,
    	wh.warehouseName
order by 
	stockTotal desc;
```

![WarehouseStock_PieChart](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/5c9b4384-d7fe-4d3a-82c5-7519e9e79afe)![WarehouseStock_Barchart](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/54347c30-c0d0-4583-a8f9-b11fb4fa93de)


The results show how the inventory is distributed across different warehouses.
The query validated that the Southern warehouses have the lowest overall inventory and Eastern Warehouses have the highest. 
Low inventory could allude to: 
1) high demand, quick turnover rates, or efficient supply chain managment.
or
2) slow turnover rate if the products are in low demand.

#### Southern Warehouse
To check for possible underlying reasons, I'm going to create a pivot table showing the specific items in stock in the Southern Warehouse. 
![SouthernWarehouse_PivotTable](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/649b2946-bd8f-4bcd-8ab7-1ea41f78971d)

Looks like the Southern Warehouse has inventory consisting primarily of the three product lines with the slowest turnover rates - Trucks and Buses, Ships, and Trains. 
Let's take a peak at the inventory in the Eastern Warehouse.

#### Eastern Warehouse

![EastWarehouse_PivotTable](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/8758b7f1-c313-4612-8f86-0761ef9f351b)

The Eastern Warehouse holds cars, which we know from previous queries have high inventory due to high demand. 


### Customers vs. Sales Amounts
Who are the top customers? To retrieve insights into customers and their spending habits, I'll be using their customer number, customer name, payment date, and the total earnings for the company associated with each payment. 

```sql
SELECT
	cmr.customerNumber,
	cmr.customerName,
    	pmt.paymentDate,
    	pmt.amount AS totalEarnings
FROM
	mintclassics.customers AS cmr
LEFT Join
	mintclassics.payments as pmt on cmr.customerNumber = pmt.customerNumber
order by
	totalEarnings desc;
```
|![Customers_pivotTable](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/12b54ebf-4f4c-4b0b-802c-977ed198e89e)![CustomerEarnings_pivotTable](https://github.com/phelpsbp/Mint-Classics-Inventory-Optimization/assets/150976820/d922638b-39d5-43e5-bd58-d25bd3b139d9)| 
|:--:| 
| The results are ordered by total earnings in descending order, meaning the customers who have spent the most are listed at the top of the table. These pivot tables show that Euro+ Shopping Channel and Mini Gifts Distributors Ltd. make up a majority of the company's sales |

## Results and Findings
The key findings are summarized as follows:
1. Trucks and Buses, Ships, and Trains were the three least ordered product lines. 
2. Trains made up only 2.67% of all orders made. Coupled with a low order frequency of 0.62%, this alludes to an overstock due to low consumer demands.
3. Southern Warehouses had low overall inventory with the inventory consisting of very slow selling product-lines.
4. The Eastern Warehouse had the highest stock levels consisting of classic model cars that are high in demand. 

## Recommendations
1. **Product Line Adjustment:**
* Consider reducing the inventory of product lines with low demand, such as Trucks and Buses, Ships, and Trains. This could involve offering promotions, bundling, or marketing strategies to boost sales.
* Allocate more resources and space to product lines with higher demand, like Classic Cars, which are top-sellers.
  
2. **Warehouse Optimization**:
* Evaluate the performance of Southern Warehouses and consider redistributing slow-moving products to other locations to ensure efficient space utilization.
* Explore the possibility of consolidating or closing Southern Warehouses if their low inventory is consistently due to slow-selling products.

3. **Customer Loyalty Programs**
* Tailor promotions or incentives that encourage repeat business.
* Consider exclusive offers for high-value customers to maintain their patronage.




---
