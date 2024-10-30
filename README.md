# Case-Study-1-Danny's-Diner
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## ERD Diagram
<img src=https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png>

## Case Study Questions with Solutions

**1. What is the total amount each customer spent at the restaurant?**
````sql
SELECT sales.customer_id,
      SUM(menu.price) AS total_amount
FROM sales
JOIN menu ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_amount DESC 
````
#### Solution:
| customer_id | total_amount |
| ----------- | -----------  |
| A           | 76           |
| B           | 74           |
| C           | 36           |

**2. How many days has each customer visited the restaurant?**

````sql
SELECT customer_id,
      COUNT(DISTINCT order_date) AS visit_num
FROM sales
GROUP BY customer_id
ORDER BY visit_num DESC
````

#### Solution:
| customer_id | visit_num  |
| ----------- | -----------|
| B           | 6          |
| A           | 4          |
| C           | 2          |

**3. What was the first item from the menu purchased by each customer?**

````sql
WITH first_order AS (
  SELECT 
    sales.customer_id, 
    menu.product_name,
    ROW_NUMBER() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS row_num
  FROM sales
  INNER JOIN menu
    ON sales.product_id = menu.product_id
)

SELECT 
  customer_id, 
  product_name
FROM first_order
WHERE row_num = 1
````

#### Solution:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| B           | curry        | 
| C           | ramen        |


**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql

````


#### Solution:
| most_purchased | product_name | 
| ----------- | ----------- |
|             |              |


**5. Which item was the most popular for each customer?**

````sql

````


#### Solution:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |


**6. Which item was purchased first by the customer after they became a member?**

```sql

```


#### Solution:
| customer_id | product_name |
| ----------- | ---------- |
|             |           |
|            |           |


**7. Which item was purchased just before the customer became a member?**

````sql

````


#### Solution:
| customer_id | product_name |
| ----------- | ---------- |
|            |         |
|            |         |


**8. What is the total items and amount spent for each member before they became a member?**

```sql

```

#### Solution:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
|              |    |         |
|              |    |         |


**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql

```


#### Solution:
| customer_id | total_points | 
| ----------- | ---------- |
|            |  |
|            |  |
|            |  |


**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

```sql

```


#### Solution:
| customer_id | total_points | 
| ----------- | ---------- |
|            |  |
|            |  |


## BONUS QUESTIONS

**Join All The Things**

**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

```sql

```
 
#### Solution: 
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |


***

**Rank All The Things**

**Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.**

```sql

```

#### Solution: 
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |


***

## Data Source & Inspiration
- [Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)
- [Katie Huang](https://github.com/katiehuangx) - GitHub Project inspiration
