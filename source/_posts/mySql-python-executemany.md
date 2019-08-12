---
title: mySql_python_executemany
date: 2019-07-31 12:22:26
tags: [mysql, python, executemany]
---

## 使用
　　使用executemany()可以完成批量执行sql语句，比自己写循环执行语句效率高很多。不过它对输入数据有格式要求，需要是一个tuple的list或者一个tuple的tuple。

<!--more-->

```python
import MySQLdb
import traceback

def _get_conn(self):
    conn = None
    counter = 0
    while counter < 10:
        try:
            conn = MySQLdb.connect(
                    user=self._user,
                    passwd=self._passwd,
                    host=self._host,
                    port=self._port,
                    db=self._database,
                    charset="utf8")
            conn.autocommit(1)
            conn.ping()
            break
        except Exception, e:
            print(traceback.format_exc())
            counter += 1
            if counter >= 10:
                raise e
            time.sleep(5)

    return conn

def db_get_res_records(self):
    res_records = []
    try:
        conn = self._get_conn()
        cursor = conn.cursor()
        query = "select * from myTable"
        print("query: %s" % query)
        cursor.execute(query)
        conn.commit()
        res = cursor.fetchall()
        for tmp in res:
            res_records.append(tmp)

    except Exception as e:
        conn.rollback()
        print(traceback.format_exc())
        print("query: %s" % query)
        raise e
    finally:
        cursor.close()
        conn.close()

    return res_records

def db_insert_res_records(self, res_records):
    """
    res_records type is a list/tuple of list/tuple
    which is required by executemany's input
    Like :
    {(1, 4, 5, 6),
     (1, 3, 5, 7),
     (1, 4, 3, 6)}
    or
    ((2, 3, 5, 7),
     (2, 4, 6, 8),
     (2, 3, 6, 9))
    """
    try:
        conn = self._get_conn()
        cursor = conn.cursor()
        query = "replace into myTable (user, date, cost_time) values (%s, %s, %s)"
        print("query: %s" % query)
        cursor.executemany(query, res_audit_records)
        conn.commit()

    except Exception as e:
        conn.rollback()
        print(traceback.format_exc())
        print("query: %s" % query)
    finally:
        cursor.close()
        conn.close()

    return True
```
