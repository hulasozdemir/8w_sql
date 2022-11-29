```sql

CREATE TABLE customer_orders_cleaned AS
SELECT order_id, customer_id, pizza_id,
	CASE 
    	WHEN exclusions LIKE 'null' OR exclusions LIKE '' OR exclusions LIKE ' '
        	THEN NULL
        ELSE exclusions
	END AS exclusions,
    
    CASE
    	WHEN extras LIKE 'null' OR extras LIKE ''
        	THEN NULL
        ELSE extras
    END AS extras,
    order_time
FROM pizza_runner.customer_orders
```

```sql
CREATE TABLE runner_orders_cleaned AS
SELECT order_id, runner_id,
    CASE
        WHEN pickup_time LIKE 'null' 
            THEN NULL
        ELSE  pickup_time
    END AS pickup_time,

    CASE
        WHEN distance LIKE 'null' 
            THEN NULL
        ELSE  distance
    END AS distance,

    CASE
        WHEN duration LIKE 'null' 
            THEN NULL
        ELSE  duration
    END AS duration,

    CASE
        WHEN cancellation LIKE 'null' OR cancellation LIKE '' OR cancellation LIKE ' '
            THEN NULL
        ELSE  cancellation
    END AS cancellation
FROM pizza_runner.runner_orders
```