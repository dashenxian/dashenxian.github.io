---
title: "Oracle全表update恢复"
publishDate: 2022-11-07 00:26:00 +0800
date: 2022-11-07 00:14:08 +0800
categories: Oracle windows dotnet csharp
position: problem
---

oracle执行update没有带where条件，造成全表更新。

---

<div id="toc"></div>

## 问题

oracle执行update没有带where条件，造成全表更新。

## 解决

```sql
1.select * from V$SQL where SQL_TEXT like '%%'--根据修改语句查出你需要恢复的时间点

2.create table new_table as select * from table as of timestamp to_timestamp('2020-09-10 11:44:25','yyyy-mm-dd            hh24:mi:ss');

--new_table :新建表的名； table ：误操作的表名；  2020-09-10 11:44:25：保存这个时间点的数据到新表。

3.delete table ;--将原表的数据全部删除

4.insert into table select * from new_table ;--把恢复的数据保存到原表。
```

---

**参考资料**

- [oracle误操作（update）数据后怎么恢复到之前--超详细](https://blog.csdn.net/l_s_r/article/details/108511501)
