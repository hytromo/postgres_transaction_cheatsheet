# Postgres Transaction Cheatsheet

## How select, update and delete work

<h3 align="center">SELECT</h3>

Takes a snapshot of the entire db and searches in the snapshot. This means that 2 selects in same transaction have their own snapshot and can see different data.

<h3 align="center">UPDATE / DELETE</h3>

![Updates and deletes workflow](/img/update_delete.png?raw=true "How updates and deletes work")

Sources

http://malisper.me/postgres-transactions-arent-fully-isolated/

http://malisper.me/postgres-row-level-locks/

http://malisper.me/postgres-transaction-isolation-levels/
