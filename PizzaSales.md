# Pizza Sales Analysis
A year's (2015) worth of sales from a fictitious pizza place, including the date and time of each order and the pizzas served, with additional details on the type, size, quantity, price, and ingredients.
- You can see the queries on BigQuery:
https://console.cloud.google.com/bigquery?sq=868099434214:346eb9d9db694076a16cb1ff9aad49aa
- First you will see the recommendations then the queries with the outputs.
## Some of the answered questions:
- How many customers do we have each day? Are there any peak hours?
- How many pizzas are typically in an order? Do we have any bestsellers?
- How much money did we make this year? Can we indentify any seasonality in the sales?
- Are there any pizzas we should take of the menu, or any promotions we could leverage?

## Recommendations:
1) Monthly sales trend:
      - July: Summer vacations, outdoor activities, and gatherings could lead to increased demand for pizza.
      - May: Graduations, end-of-school-year parties, and warmer weather might contribute to higher pizza sales.
      - March: Spring break, sports events, and the transition from winter to spring could drive more pizza orders.
      - November: Thanksgiving and the start of the holiday season might lead to families ordering pizzas for convenience during busy times.
  these months are good for promotions
2) The middle of the day is the peak hours so it would be good to increase the labor force from 11 hr to 13 hr
3) Friday, Saturday and Thursday are the peak days of the week with 7300+ orders, and the normal days are between 5900 and 6800 and that will be good to launch campaigns and promotions for the normal day to increase the sales
4) XL and XXL sizes are rarely ordered, customers prefer the normal sizes from S to L so we should avoid promotions on L+ sizes
5) Our bestsellers are The Classic Deluxe Pizza, The Barbecue Chicken Pizza, The Hawaiian Pizza, The Pepperoni Pizza and The Thai Chicken Pizza promotion on them will lead to more sales
6) We should take of The Brie Carre Pizza from the menu as it's rarely ordered 
### Count, Min, Avg and Max prices for each pizza category:

```sql
SELECT b.category AS category,
  COUNT(a.pizza_id) AS pizzas, MIN(a.price) AS min_price,
  ROUND(AVG(a.price), 2) AS avg_price,
  MAX(a.price) AS max_price 
FROM `Pizza_Sales.pizzas` a
JOIN `Pizza_Sales.pizza_type` b
  ON a.pizza_type_id = b.pizza_type_id
GROUP BY 1;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/a8dccecf-2187-4f0f-b127-467d371883d3)

### How many pizzas per order?

```sql
SELECT ROUND(AVG(quantity), 2) AS avg_pizzas_per_order
FROM `Pizza_Sales.order_details`;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/7470c43d-aef9-42a2-920e-810dabaf7fc7)

### Top 5 Selling Pizzas:

```sql
SELECT d.name, SUM(b.quantity) AS total_sold FROM `Pizza_Sales.orders` a
JOIN `Pizza_Sales.order_details` b
  ON a.order_id = b.order_id
JOIN `Pizza_Sales.pizzas` c
  ON b.pizza_id = c.pizza_id
JOIN `Pizza_Sales.pizza_type` d
  ON c.pizza_type_id = d.pizza_type_id
GROUP BY 1 
ORDER BY 2 DESC
LIMIT 5;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/dbaa921e-34b0-42b8-8d69-2b973a8c5e51)

### Total sold pizzas for each category:

```sql
SELECT d.category, SUM(b.quantity) AS total_sold FROM `Pizza_Sales.orders` a
JOIN `Pizza_Sales.order_details` b
  ON a.order_id = b.order_id
JOIN `Pizza_Sales.pizzas` c
  ON b.pizza_id = c.pizza_id
JOIN `Pizza_Sales.pizza_type` d
  ON c.pizza_type_id = d.pizza_type_id
GROUP BY 1 
ORDER BY 2 DESC;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/69ca65be-aec3-4b54-bdd9-3edc4ed58ddf)

### Most popular pizza sizes: 

```sql
SELECT c.size AS size, SUM(b.quantity) AS total_sold
FROM `Pizza_Sales.order_details` b
JOIN `Pizza_Sales.pizzas` c
  ON b.pizza_id = c.pizza_id
GROUP BY 1
ORDER BY 2 DESC;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/801ec05c-9fd1-4dbf-b90d-cf1878e64f6e)

### Peak hours:

```sql
SELECT EXTRACT(hour FROM time) AS order_hour, COUNT(a.order_id) AS num_of_orders
FROM `Pizza_Sales.orders` a
JOIN `Pizza_Sales.order_details` b 
  ON a.order_id = b.order_id
GROUP BY 1
ORDER BY 2 DESC;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/3da167a8-93a4-42a8-a497-3d72a0959a7e)

### Weekdays trend:

```sql
SELECT FORMAT_DATE('%A', date) AS order_day, COUNT(a.order_id) AS num_of_orders
FROM `Pizza_Sales.orders` a
JOIN `Pizza_Sales.order_details` b 
  ON a.order_id = b.order_id
GROUP BY 1
ORDER BY 2 DESC;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/5f748e8a-aafe-44e0-8e38-99f2f0572417)

### Total sales for the year:

```sql
SELECT ROUND(SUM(order_total), 2)
FROM(
  SELECT a.price * b.quantity AS order_total
  FROM `Pizza_Sales.pizzas` a
  JOIN `Pizza_Sales.order_details` b 
  ON a.pizza_id = b.pizza_id)
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/5c39a5b4-6fa3-4308-bd2e-56a3e8edb0fb)

### Monthly sales trend:

```sql
WITH order_total AS (
SELECT b.order_id, a.price * b.quantity AS order_total
  FROM `Pizza_Sales.pizzas` a
  JOIN `Pizza_Sales.order_details` b 
  ON a.pizza_id = b.pizza_id)

SELECT FORMAT_DATE('%B', date) AS month, ROUND(SUM(b.order_total)) AS total_sales
FROM `Pizza_Sales.orders` a
JOIN order_total b
  ON a.order_id = b.order_id
GROUP BY 1
ORDER BY 2 DESC;
```

![image](https://github.com/MohamedWageh09/Pizza-Sales-Analysis-in-BigQuery/assets/120044385/ac5cf9e1-677c-464f-a936-6515d8f86ab4)




