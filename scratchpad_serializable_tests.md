# Scratchpad Serializable Tests


## SFU wtih implicit

```sql

root@:26257/serial> select * from [show cluster setting all] where variable like '%implicit_%';
                     variable                     | value | setting_type | public |                                                                    description
--------------------------------------------------+-------+--------------+--------+----------------------------------------------------------------------------------------------------------------------------------------------------
  sql.defaults.implicit_select_for_update.enabled | true  | b            | false  | default value for enable_implicit_select_for_update session setting; enables FOR UPDATE locking during the row-fetch phase of mutation statements
  ```

  By default, it is enabled...

```sql
root@:26257/serial> set cluster setting sql.defaults.implicit_select_for_update.enabled=false;
SET CLUSTER SETTING
```


## Retry / Restart Transaction Errors

Show the **RETRY_WRITE_TOO_OLD** and **ABORT_REASON_CLIENT_REJECT** errors...

```sql

root@:26257/serial  OPEN> UPDATE alerts SET cstatus=cstatus, updated_at=now()
WHERE customer_id=9743;
ERROR: restart transaction: TransactionRetryWithProtoRefreshError: TransactionRetryError: retry txn (RETRY_WRITE_TOO_OLD - WriteTooOld flag converted to WriteTooOldError): "sql txn" meta={id=f51ff9b6 key=/Table/95/3/9743 pri=0.02812915 epo=0 ts=1600720890.957582000,1 min=1600720889.685238000,0 seq=98} lock=true stat=PENDING rts=1600720890.957582000,1 wto=false max=1600720890.185238000,0
SQLSTATE: 40001
root@:26257/? ERROR> rollback;
ROLLBACK


root@:26257/serial  OPEN> UPDATE alerts SET cstatus=cstatus, updated_at=now()
WHERE customer_id=9743;
UPDATE 98

Time: 6.455ms

root@:26257/serial  OPEN> commit;
ERROR: restart transaction: TransactionRetryWithProtoRefreshError: TransactionAbortedError(ABORT_REASON_CLIENT_REJECT): "sql txn" meta={id=c7911e6b key=/Table/95/3/9743 pri=0.00864421 epo=0 ts=1600720620.927338000,0 min=1600720619.861270000,0 seq=98} lock=true stat=PENDING rts=1600720620.927338000,0 wto=false max=1600720620.361270000,0
SQLSTATE: 40001


```





## Batching Issues /w Wide tables
Restructure Queries to update directly by PK.... This was a solution from a customer, but not sure it shows the concept real well..... **WORK IN PROGRESS!!!**


Example below:

```sql

CREATE TABLE IOT (
    id INT,
    customer_id INT,
    value1 int,
    value2 int,
    ...
    value50 int
    PRIMARY KEY (id, customer_id)
);

db.Exec(
  "BEGIN;

  DELETE FROM customers WHERE id = 1;

  DELETE orders WHERE customer = 1;

  COMMIT;"
)

BEGIN;

 DELETE FROM customers WHERE id = 1;

 DELETE orders WHERE customer = 1;

COMMIT;

```

