# Case Study #2 - Pizza Runner

[Case Study #2](https://8weeksqlchallenge.com/case-study-2/)

![Pizza](https://8weeksqlchallenge.com/images/case-study-designs/2.png)

## A. Pizza Metrics
1. How many pizzas were ordered?


```sql
SELECT COUNT(pizza_id) as "Total pizzas ordered"
FROM pizza_runner.customer_orders
```


| Total pizzas ordered |
| --- |
| 14  |

---

2. How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT(order_id)) as "Number of unique orders"
FROM pizza_runner.customer_orders
```

| Number of unique orders |
| ----------------------- |
| 10                      |

---
