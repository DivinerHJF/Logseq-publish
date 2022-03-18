- 排名是数据库中的一个经典题目，实际上又根据排名的具体细节可分为 3 种场景：
	- 连续排名，例如薪水3000、2000、2000、1000 排名结果为 1-2-3-4
	- 同薪同名但总排名不连续，例如同样的薪水分布，排名结果为 1-2-2-4
	- 同薪同名且总排名连续，同样的薪水排名结果为 1-2-2-3
- 不同的应用场景可能需要不同的排名结果，也意味着不同的查询策略。本题的目标是实现第三种排名方式下的第 N 个结果，且是全局排名，不存在分组的问题，实际上还要相对简单一些。
- **思路1：单表查询**
  collapsed:: true
	- 由于本题不存在分组排序，只需返回全局第 `N` 高的一个，所以自然想到的想法是用 `order by` 排序加 `limit` 限制得到。需要注意两个细节：
		- 同薪同名且不跳级的问题：解决办法是用 `group by` 按薪水分组后再 `order by`
		- 排名第 `N` 高意味着要跳过 `N-1` 个薪水，由于无法直接用 `limit N-1`，所以需先在函数开头处理 `N` 为 `N=N-1`。
		  注：这里不能直接用 `limit N-1` 是因为 `limit` 和 `offset` 字段后面只接受正整数（意味着 0、负数、小数都不行）或者单一变量（意味着不能用表达式），也就是说想取一条，`limit 2-1`、`limit 1.1` 这类的写法都是报错的。
		  注：这种解法形式最为简洁直观，但仅适用于查询全局排名问题，如果要求各分组的每个第 `N` 名，则该方法不适用；而且也不能处理存在重复值的情况。
	- ```mysql
	  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
	  BEGIN
	      SET N := N-1;
	    RETURN (
	        # Write your MySQL query statement below.
	        SELECT 
	              salary
	        FROM 
	              employee
	        GROUP BY 
	              salary
	        ORDER BY 
	              salary DESC
	        LIMIT N, 1
	    );
	  END
	  ```
- **思路2：子查询**
  collapsed:: true
	- 排名第 `N` 的薪水意味着该表中存在 `N-1` 个比其更高的薪水
	- 这里的 `N-1` 个更高的薪水是指去重后的 `N-1` 个，实际对应人数可能不止 `N-1` 个
	- 最后返回的薪水也应该去重，因为可能不止一个薪水排名第 `N`
	- 由于对于每个薪水的 `where` 条件都要执行一遍子查询，注定其效率低下
	- ```mysql
	  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
	  BEGIN
	    RETURN (
	        # Write your MySQL query statement below.
	        SELECT 
	            DISTINCT e.salary
	        FROM 
	            employee e
	        WHERE 
	            (SELECT count(DISTINCT salary) FROM employee WHERE salary>e.salary) = N-1
	    );
	  END
	  ```
- **思路3：自连接**
  collapsed:: true
	- 一般来说，能用子查询解决的问题也能用连接解决。
	- ```mysql
	  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
	  BEGIN
	    RETURN (
	        # Write your MySQL query statement below.
	        SELECT 
	            e1.salary
	        FROM 
	            employee e1 JOIN employee e2 ON e1.salary <= e2.salary
	        GROUP BY 
	            e1.salary
	        HAVING 
	            count(DISTINCT e2.salary) = N
	    );
	  END
	  ```
- **思路4：笛卡尔积**
  collapsed:: true
	- 可以很容易将思路 2 中的代码改为笛卡尔积连接形式，其执行过程实际上一致的，甚至 MySQL 执行时可能会优化成相同的查询语句。
	- ```mysql
	  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
	  BEGIN
	    RETURN (
	        # Write your MySQL query statement below.
	        SELECT 
	            e1.salary
	        FROM 
	            employee e1, employee e2 
	        WHERE 
	            e1.salary <= e2.salary
	        GROUP BY 
	            e1.salary
	        HAVING 
	            count(DISTINCT e2.salary) = N
	    );
	  END
	  ```
- **思路5：自定义变量**
  collapsed:: true
	- 以上方法 2-4 中均存在两表关联的问题，表中记录数少时尚可接受，当记录数量较大且无法建立合适索引时，实测速度会比较慢，用算法复杂度来形容大概是 `O(n^2)` 量级（实际还与索引有关）。那么，用下面的自定义变量的方法可实现 `O(2*n)` 量级，速度会快得多，且与索引无关。
		- 自定义变量实现按薪水降序后的数据排名，同薪同名不跳级，即 3000、2000、2000、1000 排名后为 1、2、2、3；
		- 对带有排名信息的临时表二次筛选，得到排名为 `N` 的薪水；
		- 因为薪水排名为 `N` 的记录可能不止 1 个，用 `distinct` 去重
	- ```mysql
	  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
	  BEGIN
	    RETURN (
	        # Write your MySQL query statement below.
	        SELECT 
	            DISTINCT salary 
	        FROM 
	            (SELECT 
	                  salary, @r:=IF(@p=salary, @r, @r+1) AS rnk,  @p:= salary 
	              FROM  
	                  employee, (SELECT @r:=0, @p:=NULL)init 
	              ORDER BY 
	                  salary DESC) tmp
	        WHERE rnk = N
	    );
	  END
	  ```
- **思路6：窗口函数**
  collapsed:: true
	- 实际上，在 mysql8.0 中有相关的内置函数，而且考虑了各种排名问题：
		- `row_number()`: 同薪不同名，相当于行号，例如 3000、2000、2000、1000 排名后为 1、2、3、4
		- `rank()`: 同薪同名，有跳级，例如 3000、2000、2000、1000 排名后为 1、2、2、4
		- `dense_rank()`: 同薪同名，无跳级，例如 3000、2000、2000、1000 排名后为 1、2、2、3
		- `ntile()`: 分桶排名，即首先按桶的个数分出第一二三桶，然后各桶内从 1 排名，实际不是很常用
	- ```mysql
	  CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
	  BEGIN
	    RETURN (
	        # Write your MySQL query statement below.
	          SELECT 
	              DISTINCT salary
	          FROM 
	              (SELECT 
	                  salary, dense_rank() over(ORDER BY salary DESC) AS rnk
	               FROM 
	                  employee) tmp
	          WHERE rnk = N
	    );
	  END
	  ```