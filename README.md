**<h1 align="center">MARKET BASKET ANALYSIS WITH SQL</h1>**
## Overview

Market basket analysis (or in short: MBA) is a data mining technique used by retailers to increase sales by better understanding customer purchasing patterns. It involves analyzing large data sets, such as purchase history, to reveal product groupings, as well as products that are likely to be purchased together.

Amazon's website uses a well-known example of market basket analysis. On a product page, Amazon presents users with related products, under the headings of "Frequently bought together" and "Customers who bought this item also bought

## MBA with The Bread Basket Database

This is the bakery data with the list of items bought by customers. The dataset provides transaction details of all items purchased between 2016 and 2017 from the bakery online. The dataset has 20507 entries over 9000 transactions, and 4 columns.

## Description
In this market basket analysis, I use two main attributes to perform the analysis: _Transaction_ and _Item_

_1. Transaction_: Transaction id which is unique for each order

_2. Item_: List of items to be ordered/placed by customer
```sql
Select Top 30 
	[Transaction], Item 
From
	MBA.dbo.bread_basket
  ```
## 1. Select the transactions having at least two different items.
```sql
Select 
	[Transaction], COUNT(Item) as Number_Of_Products
From 
	MBA.dbo.bread_basket
Group By 
	[Transaction] 
Having COUNT(Item) >= 2
```
We have the following result:

![1](https://user-images.githubusercontent.com/126461113/226678909-877cc319-cff0-4e8c-912e-23fdc54bc094.png)

## 2. List out the transaction and item of orders having at least two items.
```sql 
SELECT
	OrderList.[Transaction],
	BrB.Item
FROM
	(Select 
	[Transaction], COUNT(Item) as Number_Of_Products
From 
	MBA.dbo.bread_basket
Group By 
	[Transaction] 
Having COUNT(Item) >= 2) AS OrderList
JOIN MBA.dbo.bread_basket AS BrB 
ON OrderList.[Transaction] = BrB.[Transaction];
```
We have the following result:

![2](https://user-images.githubusercontent.com/126461113/226680482-349ca0d0-a026-4220-9d08-ef4cddd1e5e2.png)

## 3. Create combinations of two items in the same transaction.
```sql
With Info AS (
SELECT
	OrderList.[Transaction],
	BrB.Item
FROM
	(Select 
	[Transaction], COUNT(Item) as Number_Of_Products
From 
	MBA.dbo.bread_basket
Group By 
	[Transaction] 
Having COUNT(Item) >= 2) AS OrderList
JOIN MBA.dbo.bread_basket AS BrB 
ON OrderList.[Transaction] = BrB.[Transaction])

Select 
	Info1.[Transaction],
	Info1.Item AS Item1,
	Info2.Item AS Item2
FROM 
	Info AS Info1
	JOIN Info AS Info2 ON Info1.[Transaction] = Info2.[Transaction]
WHERE Info1.Item != Info2.Item 
AND Info1.Item > Info2.Item
```
This SQL query is using a Common Table Expression (CTE) named "Info" to create a table that contains all transactions with two or more items, and the items in those transactions.

I join the "Info" table with itself on the transaction column to find pairs of items that appear together in the same transaction

The WHERE clause filters out cases where the same item appears twice in a pair, and selects only pairs where the first item is greater than the second item in alphabetical order.

#### Info1.Item != Info2.Item
This makes sure that the items are different
#### Info1.Item > Info2.Item
This avoid duplicate items

This is the result with duplicates: 

![4](https://user-images.githubusercontent.com/126461113/226685157-e0948d44-07f8-49c3-9adb-0212113706fd.png)

This is the result without duplicates:

![3](https://user-images.githubusercontent.com/126461113/226684794-3a026ae1-d9d1-4ab0-b05d-d6322a7ea1e1.png)

## 4. Calculate the frequency of a pair of two items.
```sql
With Info AS (
SELECT
	OrderList.[Transaction],
	BrB.Item
FROM
	(Select 
	[Transaction], COUNT(Item) as Number_Of_Products
From 
	MBA.dbo.bread_basket
Group By 
	[Transaction] 
Having COUNT(Item) >= 2) AS OrderList
JOIN MBA.dbo.bread_basket AS BrB 
ON OrderList.[Transaction] = BrB.[Transaction])

Select 
	Info1.Item AS Item1,
	Info2.Item AS Item2,
	Count (*) AS Frequency
FROM 
	Info AS Info1
	JOIN Info AS Info2 ON Info1.[Transaction] = Info2.[Transaction]
WHERE Info1.Item != Info2.Item 
AND Info1.Item > Info2.Item
GROUP BY 
	Info1.Item,
	Info2.Item 
ORDER BY COUNT(*) DESC
```
This is the result: 

![5](https://user-images.githubusercontent.com/126461113/226685964-99eb60a6-ab6f-423f-8289-574cfde75eae.png)

Look at the result, we can find that when the customers buy _Coffee_, they are most likely to buy _Bread_ and _Cake_.
Therefore, the sellers can recommend their customers to buy these two items would be likely to improve sales revenue.

## Conclusion
Market basket analysis can increase sales and customer satisfaction. Using data to determine that products are often purchased together, retailers can optimize product placement, offer special deals and create new product bundles to encourage further sales of these combinations

## Reference 
- [How to perform Product Association using SQL](https://www.youtube.com/watch?v=2ZR08NJLIL4)
- [Market Basket Analysis](https://www.techtarget.com/searchcustomerexperience/definition/market-basket-analysis#:~:text=Market%20basket%20analysis%20is%20a,likely%20to%20be%20purchased%20together.)



