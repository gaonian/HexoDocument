## 1. 数据库基础

- 数据库管理软件 （DBMS）

- 数据库 （database）

- 表 （table）

- 列 （column）

- 行 （row）

- 主键 （primary key）

  应该总是定义主键，方便以后数据操作和管理。

  表中任何列都可以作为主键，需要满足以下条件:

  - 任意两行都不具有相同的主键值
  - 每一行都必须具有一个主键值 (主键列不允许NULL值)
  - 主键列中的值不允许修改或更新
  - 主键值不能重用（如果某行从表中删除，它的主键不能赋予给以后的新行）

  也可以多列一起使用作为主键



## 2. 检索数据

- SELECT

  ```sql
  // 从Products表中查找prod_name列
  SELECT prod_name 
  FROM Products;
  
  // 多列
  SELECT prod_id, prod_name, prod_price
  FROM Products;
  
  // 所有列
  SELECT *
  FROM Products;
  ```

- DISTINCT

  只检索不同的值

  ``` sql
  SELECT DISTINCT vend_id
  FROM Products;
  ```

- LIMIT OFFSET

  ```sql
  // 检索从第5行起的5行数据
  SELECT prod_name
  FROM Products
  LIMIT 5 OFFSET 5;
  ```



## 3. 排序检索数据

- ORDER BY

  order by 必须要保证是select语句中最后一条字句

  ``` sql
  SELECT prod_name
  FROM Products
  ORDER BY prod_price, prod_name;
  ```

  ``` sql
  SELECT prod_id, prod_price, prod_name
  FROM Products
  ORDER BY 2, 3;
  ```

- DESC

  降序排列

  ``` sql
  SELECT prod_id, prod_price, prod_name
  FROM Products
  ORDER BY prod_price DESC;
  ```



## 4. 过滤数据

- WHERE

  ``` sql
  SELECT prod_name, prod_price FROM Products
  WHERE prod_price = 3.49;
  ```

  ``` sql
  SELECT prod_name, prod_price
  FROM Products
  WHERE prod_price BETWEEN 5 AND 10;
  ```

  ``` sql
  SELECT prod_name, prod_price
  FROM Products
  WHERE prod_price IS NULL;
  ```

  

  ![sql1](./sql_image/sql1.png)



## 5. 高级数据过滤

- AND

  ``` sql
  SELECT prod_name, prod_price
  FROM Products
  WHERE vend_id = 'DLL01' AND prod_price <= 4;
  ```

- OR

  ``` sql
  SELECT prod_name, prod_price
  FROM Products
  WHERE vend_id = 'DLL01' OR vend_id = ‘BRS01’;
  ```

  在使用AND OR 组合的时候，应使用小括号明确分组操作符

- IN

  IN 操作符用来指定条件范围，范围中的每个条件都可以进行匹配，IN 取值由都好分割，括在圆括号中的合法值。

  where子句中用来指定要匹配值得清单的关键字，功能与OR相当

  ``` sql
  SELECT prod_name, prod_price
  FROM Products
  WHERE vend_id IN ( 'DLL01', 'BRS01' )
  ORDER BY prod_name;
  ```

- NOT

  否定其后跟着的任何条件

  ``` sql
  SELECT prod_name
  FROM Products
  WHERE NOT vend_id = 'DLL01' 
  ORDER BY prod_name;
  ```



## 6. 用通配符进行过滤

- LIKE

  - %

    %表示任何字符出现任意次数，例如寻找Fish开头的所有产品，可以用以下sql语句 %告诉DBMS接受Fish之后的任意字符，不管有多少字符

    ``` sql
    SELECT prod_id, prod_name
    FROM Products
    WHERE prod_name LIKE 'Fish%';
    ```

    ``` sql
    SELECT prod_name
    FROM Products
    WHERE prod_name LIKE 'F%y';
    ```

    ``` sql
    SELECT prod_id, prod_name
    FROM Products
    WHERE prod_name LIKE '%bean bag%';
    ```

  - _

    下划线 _ 只匹配单个字符，而不是多个字符

    ``` sql
    SELECT prod_id, prod_name
    FROM Products
    WHERE prod_name LIKE '__ inch teddy bear';
    ```

sql的通配符很有用。但这种功能是有代价的，通配符搜索一般比其他搜索会消耗更长的处理事件。

- 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符
- 在确实需要使用通配符时，也尽量不要把他们用在搜索模式的开始，把通配符用在开始处，搜索起来是最慢的
- 仔细注意通配符的位置，如果放错地方，可能不会返回想要的数据



## 7. 创建计算字段

- ||

  如果想要使返回的值拼接在一起，在sqlite中可以使用 || 进行拼接两个列

  ``` sql
  SELECT vend_name || ' (' || vend_country || ')'
  FROM Vendors
  ORDER BY vend_name;
  ```

- RTRIM()函数

  使用RTRIM函数去掉值右边的所有空格

  ``` sql
  SELECT RTRIM(vend_name) || ' (' || RTRIM(vend_country) || ')' FROM Vendors
  ORDER BY vend_name;
  ```

- AS

  别名（alias）。 通过拼接的字段并没有名字，对于客户端来说没法引用。

  为了解决这个问题，sql支持列别名，别名是一个字段或值得替换名。别名用AS关键字赋予

  别名有时也叫导出列（derived column）

  ``` sql
  SELECT RTRIM(vend_name) || ' (' || RTRIM(vend_country) || ')' AS vend_title
  FROM Vendors
  ORDER BY vend_name;
  ```

- 执行 + - * / 算术计算

  ``` sql
  SELECT prod_id,
         quantity,
  			 item_price,
         quantity*item_price AS expanded_price
  FROM OrderItems
  WHERE order_num = 20008;
  ```



## 8. 使用函数处理数据

### 文本处理函数

![](./sql_image/sql2.png)

### 数值处理函数

![](./sql_image/sql3.png)

### 日期处理函数



## 9. 汇总数据

- AVG()

  AVG返回某列的平均值。也可以用来返回特定列或行的平均值

  ``` sql
  SELECT AVG(prod_price) AS avg_price
  FROM Products;
  ```

- COUNT()

  COUNT()函数进行计数，可利用count()确定表中行的数目或符合特定条件的行的数目

  COUNT()函数有两种使用方式

  - 使用 COUNT(*) 对表中行的数目进行计数，不管表列中包含的是空置（null）还是非空值

    ```sql
    SELECT COUNT(*) AS num_cust
    FROM Customers;
    ```

  - 使用 COUNT(column) 对特定列中具有值的行进行计数，忽略NULL值

    ``` sql
    SELECT COUNT(cust_email) AS num_cust
    FROM Customers;
    ```

- MAX()

  返回指定列中的最大值。MAX()要求指定列名。忽略列值为null的行

  ``` sql
  // 返回Products表中最贵物品的价格
  SELECT MAX(prod_price) AS max_price
  FROM Products;
  ```

- MIN()

  返回指定列的最小值，MIN()要求指定列名

  ``` sql
  SELECT MIN(prod_price) AS min_price
  FROM Products;
  ```

- SUM()

  SUM()用来返回指定列值得和(总计)

  ``` sql
  SELECT SUM(quantity) AS items_ordered FROM OrderItems
  WHERE order_num = 20005;
  ```

- DISTINCT

  ``` sql
  SELECT AVG(DISTINCT prod_price) AS avg_price FROM Products
  WHERE vend_id = 'DLL01';
  ```

  DISTINCT不用用于count(*)

- 可组合函数使用

  ``` sql
  SELECT COUNT(*) AS num_items,
         MIN(prod_price) AS price_min,
         MAX(prod_price) AS price_max,
         AVG(prod_price) AS price_avg
  FROM Products;
  ```



## 10. 分组数据

