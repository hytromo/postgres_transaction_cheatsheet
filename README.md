# Postgres Transaction Cheatsheet

## How SELECT, UPDATE and DELETE work

<h3 align="center">SELECT</h3>

Takes a snapshot of the entire db and searches in the snapshot. This means that 2 selects in same transaction have their own snapshot and can see different data.

<h3 align="center">UPDATE / DELETE</h3>

<p align="center">
    <img src="https://raw.githubusercontent.com/hytromo/postgres_transaction_cheatsheet/master/img/update_delete.png" />
</p>

## Problems to deal with

<a><h3 align="center">#P1 - Non-repeatable reads</h3></a>

When a transaction reads the same row twice but finds different data, because of some other transaction.

<a><h3 align="center">#P2 - Lost update</h3></a>

Consider the following example:

1. Fetch row from db and read value (e.g. value == 1)
2. Increment a field (server side) (value++ -> value == 2)
3. Save back to db (SET value = 2)

If the above example is run concurrently from 2 apps, then the final value will be 2, not 3.

<a><h3 align="center">#P3 - Phantom read</h3></a>

The same transaction may see different number of rows in the same table because of INSERTs of other transactions.

<a><h3 align="center">#P4 - Skipped modification</h3></a>

Due to how UPDATEs and DELETEs work, a row can be true for the WHERE clause and still not be updated.
Consider the following example: 

```
DB tables->
[name:ints]
 n
---
 1 -> row#1
 2 -> row#2

T1: BEGIN
T1: UPDATE ints SET n = n+1;

T2: BEGIN
T2: DELETE from ints WHERE n = 2; -> blocks since delete is trying to modify rows being updated

T1: COMMIT

T2: COMMIT

DB tables->
[name:ints]
 n
---
 2
 3
 ```

No row is being deleted. Because: 
1. DELETE finds #row2 having the value '2'
2. It waits for it to become unlocked
3. Once unlocked, it re-evaluates the row, but now it has the value '3', and it's no longer true.
4. DELETE skips the row

The catch is that row#1 is not re-evaluated after the first transaction commits. Consult the UPDATE / DELETE image if you can't wrap your head around this, and you'll realize that `T2` blocks only on #row2.

Sources

http://malisper.me/postgres-transactions-arent-fully-isolated/

http://malisper.me/postgres-row-level-locks/

http://malisper.me/postgres-transaction-isolation-levels/
