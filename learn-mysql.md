# mysql-learn
  
## 创建数据库

创建数据库 `CREATE DATABASE demo;`  
  
查看数据库用 `SHOW DATABASES;`  

选中数据库 `USE demo;`  

查看数据表 `SHOW TABLES;`  

## 创建用户&数据库授权

### 账号管理

`CREATE USER learn@localhost IDENTIFIED BY 'demo';`  
创建了一个叫 `learn` 的用户，限制其只能从本地（`localhost`）登录，并设置其密码为 `demo`  

重命名  `RENAME USER ben TO bforta;`  

删除用户  `DROP USER bforta;`  

更改密码  `SET PASSWORD FOR bforta = Password('Totie_O');`  

`SET PASSWORD = Password('Totie_O');`  
在不指定用户名时，SET PASSWORD更改当前登陆用户的口令

### 权限管理

`GRANT ALL ON demo.* TO learn@localhost;`  
赋予该用户对 `demo` 数据库的完整权限（用户缺省是什么权限都没有的）  

特定的表  `ON database.table`

赋予用户select权限  `GRANT SELECT ON demo.* TO learn@localhost;`  
撤销用户select权限  `REVOKE SELECT ON demo.* TO learn@localhost;`  

查看权限  `SHOW GRANT FOR bforta;`

`FLUSH PRIVILEGES;`  
要求服务器刷新权限库，令前面的设置生效。  

## python连接sqlite

```python
import sqlite3
import pandas as pd

try:
    conn = sqlite3.connect('assets/flights.db')
    cursor = conn.cursor()
    cursor.execute('SELECT id, name, city, code FROM airports WHERE country="China";')

    print(cursor.fetchone())
    print(cursor.fetchone())
    print(cursor.fetchmany(5))
    
finally:
    cursor.close()
    conn.close()
```

sqlite3的数据库连接，只需要**conn**指定db的文件保存路径  
通过这个**conn**建立一个**cursor**  
通过**cursor**这个激光笔取数，execute(执行sql命令)  

通过**fetchone**、**fetchmany**对返回的cursor进行取数

## python连接Mysql

```python
import mysql.connector as mysql  # 需要pip install mysql.connector
import pandas as pd

conn = mysql.connect(
    host = 'localhost',
    user = 'learn',
    passwd = 'demo',
    database = 'demo' # 也可以不指定数据库
)

sql = 'SELECT * FROM CUSTOMERS;'

cursor = conn.cursor()
cursor.execute(sql)
print(cursor.fetchmany(5)) #  返回是个list对象，里面每行数据都是tuple
cursor.close()
```

## 通过python创建table

```python
cursor.execute('CREATE TABLE newusers (id INT(11) NOT NULL AUTO_INCREMENT, username VARCHAR(255),firstname VARCHAR(255), lastname VARCHAR(255), PRIMARY KEY(id));')
```

需要先定义table，然后再插入数据  
table定义与在Mysql的定义是一致的  

## 删除table

```python
sql = "DROP TABLE customers"

sql = "DROP TABLE IF EXISTS customers" # if exits
cursor.execute(sql)
```

### 两个主键

```sql
CREATE TABLE orderitems
(
  order_id   int          NOT NULL AUTO_INCREMENT,
  order_num  int          NOT NULL ,
  order_item int          NOT NULL ,
  prod_id    char(10)     NOT NULL ,
  quantity   int          NOT NULL ,
  item_price decimal(8,2) NOT NULL ,
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;
```

## 插入数据

```python
# 插入单行数据
cursor.execute('INSERT INTO newusers (username, firstname, lastname) VALUES ("neo", "Neo", "Lee");') 

# 插入多行数据
sql = 'INSERT INTO newusers(username, firstname, lastname) VALUES (%s, %s, %s)'
values = [
    ('trinity', 'Carrie-Anne', 'Moss'),
    ('morpheus', 'Laurence', 'Fishburne'),
    ('smith', 'Hugo', 'Weaving')
]
cursor.executemany(sql, values)

conn.commit()
```

**%s**是占位符，取代传递进来的值  
插入完数据需要提交确认  

如果代码太长，可以使用反转义字符 `/`进行换行

## 修改数据

```python
cursor.execute('UPDATE newusers SET firstname="Keanu", lastname="Reeves" WHERE username="neo"')

sql = "UPDATE customers SET address = 'Canyon 123' WHERE address = 'Valley 345'"

conn.commit()
```

## 删除数据

```python
sql = "DELETE FROM customers WHERE address = 'Mountain 21'"
cursor.execute(sql)
```

## 修改表格

### 添加唯一属性

```python
cursor.execute('ALTER TABLE newusers ADD CONSTRAINT constraint_username UNIQUE (username)')
conn.commit()
```

### 添加列

`ALTER TABLE Customers ADD Email varchar(255);`

### 删除列

`ALTER TABLE table_name DROP COLUMN column_name;`

### 修改列

`ALTER TABLE Persons MODIFY COLUMN DateOfBirth year;`

### 添加外键

修改orders表格，添加约束 指定表格外键 指定另外表格的列

```SQL
ALTER TABLE orders ADD CONSTRAINT fk_orders_customers FOREIGN KEY (cust_id) REFERENCES customers (cust_id);
```


## 事务回滚

```python
try:
    cursor.execute('INSERT INTO newusers (username, firstname, lastname) VALUES ("oracle", "Gloria", "Foster")')
    cursor.execute('INSERT INTO newusers (username, firstname, lastname) VALUES ("smith", "Robert", "Taylor")')
    conn.commit()
except:
    conn.rollback()
```

## 把数据转换成df

```python
import mysql.connector as mysql
import pandas as pd

conn = mysql.connect(
    host = 'localhost',
    user = 'learn',
    passwd = 'demo',
    database = 'demo'
)

sql = 'SELECT * FROM CUSTOMERS;'

df = pd.read_sql(sql, conn)
```

## 表连结
```python
# 内联结
sql = "SELECT \
  users.name AS user, \
  products.name AS favorite \
  FROM users \
  INNER JOIN products ON users.fav = products.id"

# 左联结
sql = "SELECT \
  users.name AS user, \
  products.name AS favorite \
  FROM users \
  LEFT JOIN products ON users.fav = products.id"

# 右联结
sql = "SELECT \
  users.name AS user, \
  products.name AS favorite \
  FROM users \
  RIGHT JOIN products ON users.fav = products.id"
```