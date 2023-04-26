# Window Functions

- Window functions operate on a subset of rows but they do not reduce the number of rows returned by the query (similar to the `GROUP BY` function -> would reduce the number of rows)

- operates on a set of rows defined by the contents of the `OVER` clause

## syntax

```sql
window_function_name(expression) OVER ( 
   [partition_defintion]
   [order_definition]
   [frame_definition]
)
```

`PARTITION BY <expression>[{,<expression>...}]`

- breaks up the rows into chunks or partitions
- window functions are performed within partitions

`ORDER BY <expression> [ASC|DESC], [{,<expression>...}]`

- specifies how the rows are ordered within a partition

`frame_unit {<frame_start>|<frame_between>}`

- A frame is defined with respect to the current row, which allows a frame to move within a partition depending on the position of the current row within its partition.



## Functions

### `dense_rank()`

- assigns a rank to each row within a partition or result set with no gaps in ranking values

- The rank of a row is increased by one from the number of distinct rank values which come before the row

- If a partition has two or more rows with the same rank value, each of these rows will be assigned the same rank.

  ```sql
  DENSE_RANK() OVER (
      PARTITION BY <expression>[{,<expression>...}]
      ORDER BY <expression> [ASC|DESC], [{,<expression>...}]
  )
  ```

  `PARTITION BY`: divides the result sets produced by the `FROM` clause into partitions, which the window function would be applied on;

  `ORDER BY`: specifies the order of rows in each partition 