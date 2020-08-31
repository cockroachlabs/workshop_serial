# Serializable Test Senarios
This will describe servaral scenarios that can be use for the workshop.


## Read Only :: Ten Rows

Want to show if there is any contention for read-only data.

Use the simple table with only ten rows:

```sql
create database serialtests;
create table tenrows(id int, name string);

insert into tenrows 
select i, lpad(cast(i as string), 64, 'S') from generate_series(1,10) as i;
```

Using Jmeter or `workload run querybench` to show behavior.  Show the following up to 16 threads:
+ 10 random id's (rows) for selects
+ 1 single id (row) for all selects

## Read Write :: Ten Rows

Using the previous `tenrows` table, run the 10 random and single row select with updates.  Use 50/50 read/write ratio to show contention and retries.

Show the following:
+ 10 random ids for select/update
+ 1 single id (row) for all select and updates 

## Writers Blocking Readers :: Breaking down locking

This exercise will show various ways that writers can block reads along with options on how to optimize to provide **read committed** like behavior.

### Full blocking
Using the `tenrows` table update **all** of the names column inside a un-commited transaction.

```sql
root@:26257/serialtests> begin;
Now adding input for a multi-line SQL transaction client-side (smart_prompt enabled).
Press Enter two times to send the SQL text collected so far to the server, or Ctrl+C to cancel.
You can also use \show to display the statements entered so far.
                      -> update tenrows set name='RuffRubyRose' where 1=1;
                      ->
UPDATE 10

Time: 2.474ms
```

In another window try to select values:

```sql
root@:26257/serialtests> select * from tenrows;
...
...  Select will not return
...
```

### Full blocking
Using the `tenrows` table update **all** of the names column inside a un-commited transaction.

```sql
root@:26257/serialtests> begin;
Now adding input for a multi-line SQL transaction client-side (smart_prompt enabled).
Press Enter two times to send the SQL text collected so far to the server, or Ctrl+C to cancel.
You can also use \show to display the statements entered so far.
                      -> update tenrows set name='RuffRubyRose' where 1=1;
                      ->
UPDATE 10

Time: 2.474ms
```


In another window try to select values:

```sql
root@:26257/serialtests> select * from tenrows;
...
...  Select will not return due to an outstanding transaction
...
```

Rollback the transaction in the first window:

```sql
root@:26257/serialtests  OPEN> rollback;
ROLLBACK

Time: 1.623ms
```

Observe the Select Query return with data in the second window:
```sql
root@:26257/serialtests> select * from tenrows;
  id |   name
-----+------------
   1 | RubyRosie
   2 | RubyRosie
   3 | RubyRosie
   4 | RubyRosie
   5 | RubyRosie
   6 | RubyRosie
   7 | RubyRosie
   8 | RubyRosie
   9 | RubyRosie
  10 | RubyRosie
(10 rows)

Time: 2m6.303112s
```

### Unblocking the Reads

You can set the priority of various transactions to allow for reads to proceed over writes.

Begin the transaction again and set the transaction priority low for the update.

```sql
root@:26257/serialtests> set session application_name='UNBLOCK';
SET

Time: 257µs

root@:26257/serialtests> BEGIN;
Now adding input for a multi-line SQL transaction client-side (smart_prompt enabled).
Press Enter two times to send the SQL text collected so far to the server, or Ctrl+C to cancel.
You can also use \show to display the statements entered so far.
                      -> SET TRANSACTION PRIORITY LOW;
                      -> update tenrows set name='RuffRubyRose' where 1=1;

UPDATE 10

Time: 1.396ms
```

Observe the Select Query return with data in the second window:
```sql
root@:26257/serialtests> select * from tenrows;
  id |   name
-----+------------
   1 | RubyRosie
   2 | RubyRosie
   3 | RubyRosie
   4 | RubyRosie
   5 | RubyRosie
   6 | RubyRosie
   7 | RubyRosie
   8 | RubyRosie
   9 | RubyRosie
  10 | RubyRosie
(10 rows)

Time: 2m6.303112s
```

**FOR THE LAB... have them set the TRANSACTION PRIORITY TO HIGH**
