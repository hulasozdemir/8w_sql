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


| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        |            | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        |            |        | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        |            | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        |            |        | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        |            |        | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

---

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


| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         |                     |          |            | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  |                         |
| 9        | 2         |                     |          |            | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  |                         |

---
