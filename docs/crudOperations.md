---
id: crudOperations
title: How to perform CRUD operations
sidebar_label: CRUD operations
---

QuestDB's data store is mostly meant to be immutable. We plan to support out-of-order inserts from 4.3 which 
in effect allows `UPDATE` and `DELETE` operations if needed. However, the most efficient way to use QuestDB is as `append only`.
We propose an efficient model to perform CRUD operations revolving around the use of `LATEST BY`.
This page describes each of CRUD operations ([create](#create), [read](#read), [update](#update), [delete](#delete)) and how to implement them in an append-only scenario at high efficiency.

## (C)reate

The Create operation in QuestDB appends records to bottom of a table. If the table has a [designated timestamp](designatedTimestamp.md), new 
record timestamps must be superior or equal to the latest timestamp. Attempts to add a timestamp in middle of a
table will result in a `timestamp out of order` error. 

If the table is [partitioned](partitions.md), then 
the `timestamp` value determines which partition the record is appended to. In this case, only the last partition can be appended to. 

When a table does not have a `designated timestamp`, records can be added in any timestamp order and the table will only have one partition.

Let's first create table that holds bank balances for customers.

<!--DOCUSAURUS_CODE_TABS-->
<!--SQL-->
```sql
create table balances (
	cust_id int, 
	balance_ccy symbol, 
	balance double, 
	inactive boolean,
	timestamp timestamp
);
```
<!--REST-->
```shell 
curl -G "http://localhost:13005/exec" --data-urlencode "query=
create table balances (
    cust_id int,
    balance_ccy symbol,
    balance double,
    inactive boolean,
    timestamp timestamp
)"
```
<!--Java-->
```java
final String cairoDatabaseRoot = "/tmp";
try (CairoEngine engine = new CairoEngine(
    new DefaultCairoConfiguration(cairoDatabaseRoot))
) {
    try (SqlCompiler compiler = new SqlCompiler(engine)) {
        compiler.compile("create table balances (\n" +
                "    cust_id int, \n" +
                "    balance_ccy symbol, \n" +
                "    balance double, \n" +
                "    inactive boolean, \n" +
                "    timestamp timestamp\n" +
                ")");
    }
}
```
<!--JDBC-->
```java
Properties properties = new Properties();
properties.setProperty("user", "admin");
properties.setProperty("password", "quest");
properties.setProperty("sslmode", "disable");

final Connection connection = 
    DriverManager.getConnection("jdbc:postgresql://localhost:9120/qdb", properties);
PreparedStatement statement = connection.prepareStatement(
    "create table balances (" +
    "    cust_id int," +
    "    balance_ccy symbol," +
    "    balance double," +
    "    inactive boolean," +
    "    timestamp timestamp" +
    ")"
);
statement.execute();
connection.close();

```
<!--END_DOCUSAURUS_CODE_TABS-->

 - `cust_id` is the customer identifier. It uniquely identifies customer.
 - `balance_ccy` balance currency. We use `SYMBOL` here to avoid storing text against each record to save space and increase database performance.
 - `balance` is the current balance for customer and currency tuple.
 - `inactive` is used to flag deleted records.
 - `timestamp` timestamp in microseconds of the record. Note that if you receive the timestamp data as a string, it could also be inserted using [to_timestamp](functionsDateAndTime.md#to_timestamp).

Let's now insert a few records:
<!--DOCUSAURUS_CODE_TABS-->
<!--SQL-->

```sql
insert into balances (cust_id, balance_ccy, balance, timestamp)
values (1, 'USD', 1500.00, 1587571882704665);

insert into balances (cust_id, balance_ccy, balance, timestamp)
values (1, 'EUR', 650.50, 1587571892904234);

insert into balances (cust_id, balance_ccy, balance, timestamp)
values (2, 'USD', 900.75, 1587571963504432);

insert into balances (cust_id, balance_ccy, balance, timestamp)
values (2, 'EUR', 880.20, 1587572314404665);
```
<!--REST-->
```shell 
curl -G "http://localhost:13005/exec" --data-urlencode "query=
insert into balances (cust_id, balance_ccy, balance, timestamp)
	values (1, 'USD', 1500.00, 1587571882704665)
"
```
<!--Java Raw-->
```java
CairoConfiguration configuration = new DefaultCairoConfiguration(".");
try (CairoEngine engine = new CairoEngine(configuration)) {
    try (TableWriter writer = engine.getWriter(AllowAllCairoSecurityContext.INSTANCE, "balances")) {
        TableWriter.Row r;

        r = writer.newRow(1587571882704665); // timestamp
        r.putInt(0, 1); // cust_id
        r.putSym(1, "USD"); // symbol
        r.putDouble(2, 1500.00); // balance
        r.append();

        r = writer.newRow(1587571892904234); // timestamp
        r.putInt(0, 1); // cust_id
        r.putSym(1, "EUR"); // symbol
        r.putDouble(2, 650.5); // balance
        r.append();

        r = writer.newRow(1587571963504432); // timestamp
        r.putInt(0, 2); // cust_id
        r.putSym(1, "USD"); // symbol
        r.putDouble(2, 900.75); // balance
        r.append();

        r = writer.newRow(1587572314404665); // timestamp
        r.putInt(0, 2); // cust_id
        r.putSym(1, "USD"); // symbol
        r.putDouble(2, 880.2); // balance
        r.append();

        writer.commit();
    }
}
```
<!-- JDBC -->
```java
Properties properties = new Properties();
properties.setProperty("user", "admin");
properties.setProperty("password", "quest");
properties.setProperty("sslmode", "disable");

final Connection connection = 
    DriverManager.getConnection("jdbc:postgresql://localhost:9120/qdb", properties);
PreparedStatement insert = connection.prepareStatement(
    "insert into balances (cust_id, balance_ccy, balance, timestamp)"+
     	"values (?, ?, ?, ?)";

)
insert.setInt(1, 1);
insert.setString(2, "USD");
insert.setDouble(3, 1500);
insert.setLong(4, 1587571882704665);

insert.execute();
connection.close();

```

<!--END_DOCUSAURUS_CODE_TABS-->

Our resulting table looks like the following.

|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|USD|1500|FALSE|2020-04-22T16:11:22.704665Z
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z
|2|USD|900.75|FALSE|2020-04-22T16:12:43.504432Z
|2|EUR|880.2|FALSE|2020-04-22T16:18:34.404665Z

## (R)ead

Reading records can be done using `SELECT` or by reading a table directly via the Java API. 
Reading via the [Java API](embeddedJavaAPI.md) (see tab `Java Raw`) iterates over a table and can therefore only access one table at a time.
If you would like to query various tables via the Java API, you can pass SQL to Java and read the resulting table 
(see tab `Java SQL`).

<!--DOCUSAURUS_CODE_TABS-->
<!--SQL-->
```sql
balances; 
```
<!--REST-->
```shell 
curl -G "http://localhost:9000/exec" --data-urlencode "query=select * from balances"
```
<!--Java SQL-->
```java
final String cairoDatabaseRoot = "/tmp";
CairoConfiguration configuration = new DefaultCairoConfiguration(cairoDatabaseRoot);
try (CairoEngine engine = new CairoEngine(configuration)) {
    try (SqlCompiler compiler = new SqlCompiler(engine)) {
        try (RecordCursorFactory factory = compiler.compile("select * from balances").getRecordCursorFactory()) {

            try (RecordCursor cursor = factory.getCursor()) {
                final Record record = cursor.getRecord();
                while (cursor.hasNext()) {
                    record.getInt(0); // cust_id
                    record.getSym(1); // symbol
                    record.getDouble(2); // balance
                    record.getByte(3); // status
                    record.getTimestamp(4); // timestamp
                }
            }
        }
    }
}
```
<!--Java Raw-->
```java
CairoConfiguration configuration = new DefaultCairoConfiguration(".");
try (CairoEngine engine = new CairoEngine(configuration)) {
    try (TableReader reader = engine.getReader(AllowAllCairoSecurityContext.INSTANCE, "balances")) {
        // closing this cursor will close reader too
        // lets close reader explicitly
        final TableReaderRecordCursor cursor = reader.getCursor();
        final Record record = cursor.getRecord();
        while (cursor.hasNext()) {
            record.getInt(0); // cust_id
            record.getSym(1); // symbol
            record.getDouble(2); // balance
            record.getByte(3); // status
            record.getTimestamp(4); // timestamp
        }
    }
}
```
<!--JDBC-->
```java
Properties properties = new Properties();
properties.setProperty("user", "admin");
properties.setProperty("password", "quest");
properties.setProperty("sslmode", "disable");

final Connection connection =
        DriverManager.getConnection("jdbc:postgresql://localhost:8812/qdb", properties);
PreparedStatement statement = connection.prepareStatement("select * from balances");

ResultSet resultSet = statement.executeQuery();

while (resultSet.next()) {
    System.out.println(resultSet.getInt(1)); // cust_id
    System.out.println(resultSet.getString(2)); // symbol
    System.out.println(resultSet.getDouble(3)); // balance
    System.out.println(resultSet.getByte(4)); // status
    System.out.println(resultSet.getTimestamp(5)); // timestamp
}
connection.close();
```
<!--END_DOCUSAURUS_CODE_TABS-->

The results are shown below
|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|USD|1500|FALSE|2020-04-22T16:11:22.704665Z
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z
|2|USD|900.75|FALSE|2020-04-22T16:12:43.504432Z
|2|EUR|880.2|FALSE|2020-04-22T16:18:34.404665Z

You can use [aggregation functions](functionsAggregation.md) to derive information like the average balance
per currency (note the [voluntary omission of redundant GROUP BY](sqlOverview.md#absence-of-group-by) below). 
```sql
select balance_ccy, avg(balance) from balances;
```

| balance_ccy | avg |
|---|---|
|USD|1200.375|
|EUR|765.35|

If we had more data we could get deeper and use [SAMPLE BY](sqlSELECT.md#sample-by) clauses to easily generate aggregates based on time intervals. 
For example, to get the average hourly balance per currency, all we need is to add `SAMPLE BY 1h` to the above query! 


## (U)pdate

Lets update balance of customer `1` in the `balances` table:

```sql
insert into balances (cust_id, balance_ccy, balance, timestamp)
values (1, 'USD', 330.50, 1587572414404997);
```

After this step, our table looks like this

|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|USD|1500|FALSE|2020-04-22T16:11:22.704665Z
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z
|2|USD|900.75|FALSE|2020-04-22T16:12:43.504432Z
|2|EUR|880.2|FALSE|2020-04-22T16:18:34.404665Z
|**1**|**USD**|**330.5**|**FALSE**|**2020-04-22T16:20:14.404997Z**

You might expect an `UPDATE`. QuestDB uses `INSERT` which means each table keeps change history.
In order to select only the latest value, our `SELECT` statement will have to change. 
We use `LATEST BY` to only select last row for the `(1,USD)` tuple for customer 1 and find the updated USD balance.

```sql
balances 
latest by cust_id, balance_ccy 
where cust_id = 1
```
|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z
|1|USD|330.5|FALSE|2020-04-22T16:20:14.404997Z

In the above example QuestDB will execute the `where` clause *before* `latest by`. To execute `where` _after_ `latest by` 
we have to rely on sub-queries. To find out more, check out our [SQL execution order](sqlExecutionOrder.md)
Here is an example of how to select the latest account information, only for balances over 800.

```sql
(balances 
latest by cust_id, balance_ccy) 
where balance > 800
```

|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|USD|1500|FALSE|2020-04-22T16:11:22.704665Z
|2|USD|900.75|FALSE|2020-04-22T16:12:43.504432Z
|2|EUR|880.2|FALSE|2020-04-22T16:18:34.404665Z

> `latest by` performance note: QuestDB will search time series from newest values to oldest. For single `SYMBOL` column in `latest by` clause QuestDB will know all distinct values upfront. Time series scan will end as soon as
> all values are matched. For all other field types, or multiple fields QuestDB will scan entire time series. Although scan is very fast you should be aware that in certain setups, performance will degrade on hundreds of millions of records. 


## (D)elete

Let's assume that `customer 1` closes their `USD` account but keeps their `EUR` account. This can be reflected in the database as follows.

```sql
insert into balances 
(cust_id, balance_ccy, inactive, timestamp) 
values (1, 'USD', true, 1587572423312698));
```

Notice that this sets tue `inactive` boolean flag to `true` for this balance. At this point, the `balances` table looks like the below

|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|USD|1500|FALSE|2020-04-22T16:11:22.704665Z
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z
|2|USD|900.75|FALSE|2020-04-22T16:12:43.504432Z
|2|EUR|880.2|FALSE|2020-04-22T16:18:34.404665Z
|1|USD|330.5|FALSE|2020-04-22T16:20:14.404997Z
|**1**|**USD**|**null**|**TRUE**|**2020-04-22T16:20:23.312698Z**

If we now want to look at the active account balances for `customer 1` we can do so as follows:

```sql
(balances 
latest by balance_ccy 
where cust_id=1) 
where not inactive;
```

The results will exclude deleted records (the USD balance) and only show the latest EUR balance for this customer.

|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z

> Note that in the above sql example, we use brackets. This is because our [SQL execution order](sqlExecutionOrder.md) will execute WHERE 
>clauses before LATEST BY. By encapsulating the query and applying `where not inactive` to the whole result set, we 
>are able to easily remove the inactive accounts. 

In other words, the brackets allow us to get "the latest balance excluding inactive". 
If we were to remove the brackets and use the following query, we would get "the latest non inactive balance" which is slightly different.

```sql
balances 
latest by balance_ccy 
where cust_id=1 
and not inactive;
```

and returns different results (not what we are looking for!). See below.

|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|USD|1500|FALSE|2020-04-22T16:11:22.704665Z
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z

## Traveling through time

This approach to CRUD operations may be unusual for traditional database users. 
A major advantage of this approach is a superior write and seek speed over traditional relational models along with contiguous storage. 
Moreover, it removes the need to maintain separated master and audit tables.

There is another nice advantage. By keeping all change history and leveraging QuestDB's seek speed, you can trivially 
travel through time at incredible speed and reproduce the state of the database at any point in time. You can use 
this to restore a previous state, or simply to produce snapshots. Welcome to the world of fast time travel!

![btff](assets/bttf.jpg)

For example the below query can be used to know the state of all balances at a `15:00:00` snapshot.
```sql
balances 
latest by balance_ccy, cust_id 
where timestamp <= '2020-04-22T16:15:00.000Z' 
and not inactive;
```

|cust_id|balance_ccy|balance|inactive|timestamp
|---|---|---|---|---
|1|USD|1500|FALSE|2020-04-22T16:11:22.704665Z
|1|EUR|650.5|FALSE|2020-04-22T16:11:32.904234Z
|2|USD|900.75|FALSE|2020-04-22T16:12:43.504432Z

What's great about this is that instead of scanning the whole table, QuestDB is capable of very efficiently locating 
the point in time requested and pull the relevant data. It can perform just as efficiently for any timestamp, from a 
few hundred to billions of rows!


