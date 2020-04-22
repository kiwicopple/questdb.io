---
id: backup
title: BACKUP
sidebar_label: BACKUP
---

## Syntax
![alt-text](assets/backup.svg)

## Description
Creates a backup for one, several, or all database tables. 

## Required configuration
`BACKUP TABLE` requires a `backup directory` which is set using the [configuration key](serverConf.md) `cairo.sql.copy.root` in the [server.conf](rootDirectoryStructure.md#serverconf) file.

Example:
```shell script
cairo.sql.backup.root=/Users/UserName/Desktop
```

> The `backup directory` can be on a local disk to the server, on a remote disk, or a remote filesystem. QuestDB will 
>enforce that the backup are only written in a location relative to the `backup directory`. This is a security feature to disallow 
>random file access by QuestDB.

## Files location
The tables will be written in a directory with today's date. By default, the format is `yyyy-MM-dd`, for example `2020-04-20`. 
If you would like to use a custom format, you can define it using the following [configuration key](serverConf.md) `cairo.sql.backup.dir.datetime.format` like the example below
```sql
cairo.sql.backup.dir.datetime.format=yyyy-dd-MM
```
The data and meta files will be written following the 
[db directory structure](rootDirectoryStructure.md#db)
```filestructure
'backup directory/'
2020-04-20
├── table1 
├── table2  
└── ... 
```

If a user performs several backups on the same date, each backup will be written a new directory. Subsequent backups on the same date 
will look as follows:
```filestructure
'backup directory/'
├── 2020-04-20      'first'
├── 2020-04-20.1    'second'
├── 2020-04-20.2    'third'
├── 2020-04-21      'first new date'
├── 2020-04-21.1    'first new date'
└── ... 
```

## Examples

#### Backup - Single table
```sql
BACKUP TABLE myTable;
```

#### Backup - Multiple table
```sql
BACKUP TABLE table1, table2, table3;
```

#### Backup - All tables
```sql
BACKUP DATABASE;
```