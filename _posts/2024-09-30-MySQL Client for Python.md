---
layout:     post
title:      Client for Python
subtitle:   
date:       2024-09-30
author:     Deng You
header-img: img/post-bg-mysql.jpeg
catalog: true
tags:
    - python
---

# Client for Python

## 基本使用

```python
import pymysql

conn = pymysql.connect(
    host="127.0.0.1",
    database="demodb",
    user="root",
    password="rootPwd",
    port=3306,
    charset="utf8mb4",
)

cur = conn.cursor()
try:
    cur.execute('insert into `author` (`id`, `name`, `country`) value ("3", "Sum", "USA")')
    conn.commit()
finally:
    cur.close()
    conn.close()
```

## with语法

使用 with 语句，是代码看起来更优雅。

```python
import pymysql

conn = pymysql.connect(
    host="127.0.0.1",
    database="demodb",
    user="root",
    password="rootPwd",
    port=3306,
    charset="utf8mb4",
)
try:
    with conn.cursor() as cur:
        cur.execute('insert into `author` (`id`, `name`, `country`) value ("7", "Ben", "USA")')
        cur.execute('insert into `author` (`id`, `name`, `country`) value ("8", "July", "USA")')
        # 可一次提交多个事务
        conn.commit()
finally:
    conn.close()
```

## 查询返回元祖

无论使用 fetchone 还是fetchmany 都会返回全部的结果给客户端，性能很差，所以sql语句中建议添加 `limit` 语句或者 `where` 语句。

```python
import pymysql

conn = pymysql.connect(
    host="127.0.0.1",
    database="demodb",
    user="root",
    password="rootPwd",
    port=3306,
    charset="utf8mb4",
)
try:
    with conn.cursor() as cur:
        results = cur.execute('select * from author')
        print(results) # result 返回受影响的row

        # result_tuple = cur.fetchall() # 以元祖的方式返回结果
        # print(result_tuple)
        # 
        result_many = cur.fetchmany(3) # 返回指定的数字的行
        print(result_many)

        # result_one = cur.fetchone() 返回单条结果
        # print(result_one)
finally:
    conn.close()
```

## 查询返回字典

```python
import time
import pymysql
from pymysql.cursors import DictCursor

conn = pymysql.connect(
    host="127.0.0.1",
    database="demodb",
    user="root",
    password="rootPwd",
    port=3306,
    charset="utf8mb4",
    cursorclass=DictCursor
)
try:
    with conn.cursor() as cur:
        results = cur.execute('select * from author')
        print(results)  # result 返回受影响的row
        result_many = cur.fetchmany(3)
        print(result_many)
finally:
    conn.close()
```
