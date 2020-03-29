### CTE概念( CTE 只在 mysql 8.0 版本后有支持)
CTE 全称 common table expression 公共表表达式，CTE 是一个命名的临时结果集，仅在单个SQL语句的执行范围内存在（即排除有子查询的SQL语句），与派生表类似，CTE 不作为对象存储，仅在查询执行期间持续。与派生表不同，CTE 可以是自引用，也可以在同一查询中多次引用。此外与派生表相比，CTE 提供了更好的可读性和性能。

### 语法
```
WITH cte_name (column_list) AS (
    query
)  
SELECT * FROM cte_name;
```
请注意，查询中的列数必须与column_list中的列数相同。 如果省略column_list，CTE将使用定义CTE的查询的列列表。
