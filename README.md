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
  JOIN menu
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
SELECT menu.product_name,
      COUNT(sales.product_id) AS purchased_number
FROM menu
JOIN sales ON menu.product_id = sales.product_id
GROUP BY product_name
ORDER BY purchased_number DESC
LIMIT 1
````


#### Solution:
| product_name | purchased_number| 
| ----------- | ----------- |
|ramen        |8              |


**5. Which item was the most popular for each customer?**

````sql
WITH popular_item AS (
  SELECT sales.customer_id,
          menu.product_name,
          COUNT(sales.product_id) AS order_num,
          DENSE_RANK() OVER (
            PARTITION BY sales.customer_id 
            ORDER BY COUNT(sales.product_id) DESC
          ) AS product_rank
  FROM sales
  JOIN menu ON sales.product_id = menu.product_id
  GROUP BY sales.customer_id, menu.product_name
)
SELECT customer_id,
        product_name,
        order_num
FROM popular_item
WHERE product_rank = 1
````


#### Solution:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A	| ramen	| 3 |
| B	| ramen	| 2 |
| B	| curry	| 2 |
| B	| sushi	| 2 |
| C	| ramen	| 3 |

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH become_member AS (
  		SELECT members.customer_id,
  				members.join_date,
  				sales.product_id,
  				menu.product_name,
  				ROW_NUMBER() OVER (
                  PARTITION BY members.customer_id
                  ORDER BY sales.order_date 
  				) AS member_rank
        FROM members
        JOIN sales ON members.customer_id = sales.customer_id
                   AND sales.order_date > members.join_date
        JOIN menu ON sales.product_id = menu.product_id
        )
SELECT customer_id,
		product_name
FROM become_member
WHERE member_rank = 1
```


#### Solution:
| customer_id | product_name |
| ----------- | ---------- |
|    A         |   ramen      |
|    B        |     sushi      |


**7. Which item was purchased just before the customer became a member?**

````sql
WITH become_member AS (
  		SELECT members.customer_id,
  				members.join_date,
  				sales.product_id,
  				menu.product_name,
  				ROW_NUMBER() OVER (
                  PARTITION BY members.customer_id
                  ORDER BY sales.order_date DESC
  				) AS member_rank
        FROM members
        JOIN sales ON members.customer_id = sales.customer_id
                   AND sales.order_date < members.join_date
        JOIN menu ON sales.product_id = menu.product_id
        )
SELECT customer_id,
		product_name
FROM become_member
WHERE member_rank = 1
````


#### Solution:
| customer_id | product_name |
| ----------- | ---------- |
|    A         |   sushi      |
|    B        |     sushi      |


**8. What is the total items and amount spent for each member before they became a member?**

```sql
WITH become_member AS (
  		SELECT sales.customer_id,
  				sales.order_date,
  				COUNT(sales.product_id) AS total_items,
  				menu.price
  		FROM sales
  		JOIN menu ON sales.product_id = menu.product_id
  		GROUP BY sales.customer_id,
  				sales.order_date,
  				menu.price
		)
SELECT become_member.customer_id,
		SUM(become_member.total_items) AS total_items,
        SUM(become_member.price) AS amount_spent
FROM become_member
JOIN members ON become_member.customer_id = members.customer_id
			AND become_member.order_date < members.join_date
GROUP BY become_member.customer_id
ORDER BY total_items, amount_spent DESC
```

#### Solution:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
|       A      |    2	|    25     |
|       B      |    3	|    40     |


**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
WITH menu_points AS(
  		SELECT sales.customer_id,
                menu.product_name,
                menu.price AS total_spent,
                CASE 
                    WHEN menu.product_name = 'sushi' THEN SUM(menu.price) * 20
                    ELSE SUM(menu.price) * 10
                END as total_points
          FROM sales
          JOIN menu ON sales.product_id = menu.product_id
          GROUP BY sales.customer_id,
                menu.product_name,
  				total_spent
  )
  SELECT customer_id,
  		SUM(total_points) AS total_points
  FROM menu_points
  GROUP BY customer_id
  ORDER BY total_points DESC	
```


#### Solution:
| customer_id | total_points | 
| ----------- | ---------- |
|        B    |   940 | 
|       A     |   860 | 
|        C    |   360 | 


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
## Learnings
## 1. Use of DENSE_RANK(), RANK(), ROW_NUMBER()
      - DENSE_RANK(): Best when you want to show all tied top items for each customer without skipping ranks.
            Example - 
            | customer_id | product_name | order_count | rank |
            |-------------|--------------|-------------|------|
            | 1           | curry        | 5           | 1    |
            | 1           | sushi        | 5           | 1    |
            | 1           | ramen        | 5           | 1    |  <-- Rank 1 for all the items
      
      - RANK(): Similar to DENSE_RANK(), but skips ranks after ties.
            Example - 
            | customer_id | product_name | order_count | rank |
            |-------------|--------------|-------------|------|
            | 1           | curry        | 5           | 1    |
            | 1           | sushi        | 5           | 1    |
            | 1           | ramen        | 3           | 3    |  <-- Rank 2 is skipped due to the tie on rank 1
      
      - ROW_NUMBER(): Use when you need unique ranks without ties; not suitable for finding "most popular" items when ties exist.
            Example - 
            | customer_id | product_name | order_count | rank |
            |-------------|--------------|-------------|------|
            | 1           | curry        | 5           | 1    |
            | 1           | sushi        | 5           | 2    |  <-- No ties; each row gets a unique rank
            | 1           | ramen        | 3           | 3    |


## Data Source & Inspiration
- [Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)
- [Katie Huang](https://github.com/katiehuangx) - GitHub Project inspiration
