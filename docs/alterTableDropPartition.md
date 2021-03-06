---
id: alterTableDropPartition
title: ALTER TABLE DROP PARTITION
sidebar_label: ALTER TABLE DROP PARTITION
---

## Synopsis

Drops table partition

## Syntax

![alter table](assets/alter-table.svg)
![drop partition](assets/alter-table-drop-partition.svg)

## Description

Drops one or more table partitions. Partition name must match the name of the directory for the given partition.

Just like with columns dropping of partitions is a non-blocking and non-waiting operation. While being atomic for a single partitions, dropping of
multiple partitions is in itself non-atomic. Drop partition will bail on the first failure and will not continue with the list.

> Last (active) partition cannot be removed (not implemented)

Naming convention for partition directories is as follows:

| Table Partition                           | Partition format                                  |
|-------------------------------------------|---------------------------------------------------|
|DAY                                        |'YYYY-MM-DD'                                       |
|MONTH                                      |'YYYY-MM'                                          |
|YEAR                                       |'YYYY'                                             |

## Examples
Drop one partition
```sql
--DAY
ALTER TABLE my_table DROP PARTITION '2019-05-18';
--MONTH
ALTER TABLE my_table DROP PARTITION '2019-05';
--YEAR
ALTER TABLE my_table DROP PARTITION '2019';
```

Drop several partitions
```sql
ALTER TABLE my_table DROP PARTITION '2018','2019';
```
