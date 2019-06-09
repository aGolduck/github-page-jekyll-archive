---
title: "Distinct and Group By"
date: 2019-06-10
tags: [mysql]
---

In MySQL, `DISTINCT` can be replaced by `GROUP BY` in principle.

`question` is a table used in our online mysql server. This is part of the structure of the table.

```sql
SHOW FULL COLUMNS FROM question;
```

| Field | Type                                                                                      |
|----- |----------------------------------------------------------------------------------------- |
| id    | int(10) unsigned                                                                          |
| type  | enum('choice','single\_choice','uncertain\_choice','determine','fill','essay','material') |
| score | float(10,1) unsigned                                                                      |

```sql
SHOW INDEX FROM question;
```

| Table    | Non\_unique | Key\_name | Seq\_in\_index | Column\_name | Index\_type |
|-------- |----------- |--------- |-------------- |------------ |----------- |
| question | 0           | PRIMARY   | 1              | id           | BTREE       |
| question | 1           | type      | 1              | type         | BTREE       |

Let's see results of `DISTINCT` and `GROUP BY` on `type`.

```sql
SELECT DISTINCT type FROM question;
```

| type              |
|----------------- |
| choice            |
| single\_choice    |
| uncertain\_choice |
| determine         |
| fill              |
| essay             |
| material          |

```sql
SELECT type FROM questiton GROUP BY type;
```

| type              |
|----------------- |
| choice            |
| single\_choice    |
| uncertain\_choice |
| determine         |
| fill              |
| essay             |
| material          |

They are exactly the same.

So how MySQL works out these results. Let's `EXPLAIN`.

```sql
EXPLAIN SELECT DISTINCT type FROM question;
```

| id | select\_type | table    | type  | possible\_keys | key  | key\_len | ref  | rows | Extra                    |
|--- |------------ |-------- |----- |-------------- |---- |-------- |---- |---- |------------------------ |
| 1  | SIMPLE       | question | range | type           | type | 1        | NULL | 16   | Using index for group-by |

```sql
EXPLAIN SELECT type FROM question GROUP BY type;
```

| id | select\_type | table    | type  | possible\_keys | key  | key\_len | ref  | rows | Extra                    |
|--- |------------ |-------- |----- |-------------- |---- |-------- |---- |---- |------------------------ |
| 1  | SIMPLE       | question | range | type           | type | 1        | NULL | 16   | Using index for group-by |

Interesting, the plan of `DISTINCT` query is using index for group-by. MySQL basically translates `DINSTINCT` query to `GROUP BY` query.

What about `score` of `question`. Let's explain first.

```sql
EXPLAIN SELECT DISTINCT score FROM question;
```

| id | select\_type | table    | type | possible\_keys | key  | key\_len | ref  | rows  | Extra           |
|--- |------------ |-------- |---- |-------------- |---- |-------- |---- |----- |--------------- |
| 1  | SIMPLE       | question | ALL  | NULL           | NULL | NULL     | NULL | 37424 | Using temporary |

```sql
EXPLAIN SELECT score FROM question GROUP BY score;
```

| id | select\_type | table    | type | possible\_keys | key  | key\_len | ref  | rows  | Extra                           |
|--- |------------ |-------- |---- |-------------- |---- |-------- |---- |----- |------------------------------- |
| 1  | SIMPLE       | question | ALL  | NULL           | NULL | NULL     | NULL | 37424 | Using temporary; Using filesort |

Without index, MySQL scans the whole table first, then `DISTINCT` or `GROUPBY`. But this time, `GROUP BY` uses filesort while `DISTINCT` not.

```sql
SELECT DISTINCT score FROM question;
```

| score |
|----- |
| 1.0   |
| 2.0   |
| 0.0   |
| 3.0   |
| 8.0   |
| 12.0  |
| 6.0   |
| 4.0   |
| 10.0  |
| 14.0  |
| 18.0  |
| 2.5   |
| 15.0  |
| 2.3   |
| 5.0   |
| 1.5   |
| 4.5   |

```sql
SELECT score FROM question GROUP BY score;
```

| score |
|----- |
| 0.0   |
| 1.0   |
| 1.5   |
| 2.0   |
| 2.3   |
| 2.5   |
| 3.0   |
| 4.0   |
| 4.5   |
| 5.0   |
| 6.0   |
| 8.0   |
| 10.0  |
| 12.0  |
| 14.0  |
| 15.0  |
| 18.0  |

See, the result of `GROUP BY` is sorted while `DISTINCT` is not. This is because MySQL always uses `sort` to group rows.

So, yeah, the strategy to choose execute plan of MySQL is quite complicated.
