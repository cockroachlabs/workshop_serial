# Serial Contention Tests 
This is to show how writers block readers.  Readers may not have to RETRY, but ranges need to be REFRESHED to get the most current value of there are *write intents* on the same rows.

Create and Populate the ALERTS table to be used by the various tests from [Contention_Update_and_Reads.jmx](Contention_Update_and_Reads.jmx) with Jmeter....


## DDL creation of alerts table
```sql

CREATE TABLE alerts (
    id INT NOT NULL DEFAULT unique_rowid(),
    customer_id INT,
    alert_type STRING,
    severity INT,
    cstatus STRING,
    adesc STRING,
    id1 INT,
    id1_desc STRING,
    id2 INT,
    id2_desc STRING,
    created_at TIMESTAMP NOT NULL DEFAULT now(),
    updated_at TIMESTAMP NOT NULL DEFAULT now(),
    PRIMARY KEY (id),
    INDEX alerts_i_idx_1 (cstatus ASC, customer_id ASC, id1 ASC, severity ASC),
    INDEX alerts_i_idx_2 (customer_id ASC, id1 ASC, id1_desc ASC, id2 ASC),
    INDEX alerts_i_idx_3 (id2 ASC, id2_desc ASC, cstatus ASC)
);

insert into alerts
select 
a,
round(random()*10000)::INT,
'ALERT_TYPE',
round(random()*10)::INT,
concat('STATUS-',round(random()*10)::STRING),
'ADESC',
round(random()*1000)::INT,
'ID1_DESCRIPTION',
round(random()*5000)::INT,
'ID2_DESCRIPTION',
now(),
now()
from generate_series(1,1000000) as a;
```

## Queries and DML updates

```sql
-- Simple Select by customer_id (returns ~98 rows)
--
SELECT * FROM alerts 
WHERE customer_id=9743;

-- Update to SAME rows by customer_id  (Implicit)
--
UPDATE alerts SET cstatus=cstatus, updated_at=now() 
WHERE customer_id=9743;
```

```sql
-- Transactions with Priority Settings (Explicit)
--
BEGIN;

-- Priority Settings
--
-- SET TRANSACTION PRIORITY LOW;
-- SET TRANSACTION PRIORITY NORMAL;  -- Default no need to set
SET TRANSACTION PRIORITY HIGH;

-- SFU settings
--
-- SELECT * FROM alerts
-- WHERE customer_id=9743 FOR UPDATE;

UPDATE alerts SET cstatus=cstatus, updated_at=now() 
WHERE customer_id=9743;

-- Induce Sleep if required using pg_sleep()... in seconds
--
--SELECT pg_sleep(0.1);

COMMIT;

```

