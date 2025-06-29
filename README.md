# NOTE-Transaction-Management-ACID-Principles-in-Spring-SQL

⚙️ TRANSACTION STEPS

BEGIN → NEW ID IS ASSIGNED TO TRANSACTION

EXECUTE → CHANGES ARE MADE BUT ARE STORED IN RAM AND CACHE.

WAL LOG → BEFORE WRITING ANYTHING TO THE DISK, CHANGES ARE WRITTEN IN THE WAL LOGS AND ARE BEING SAVED IN RAM.

COMMIT → DB CONFIRMS THAT ALL WAL LOGS ARE IN RAM.

APPLY → AFTER COMMIT, CHANGES ARE WRITTEN FROM RAM TO ACTUAL SSD DB TABLES.

WAL LOG CLEAN → AFTER DATA IS WRITTEN TO DISK. WAL LOGS ARE CLEANED FROM RAM.



🔐 ACID

ATOMICITY – ENSURES WHETHER BOTH FAIL OR BOTH SUCCEED.

CONSISTENCY – MAINTAINS VALID DATA, WHEN ACC HAS 500$, 1000$ CAN NOT BE WITHDRAWN

ISOLATION – TWO THREAD TRIES TO BOOK THE LAST SEAT. SYSTEMS LETS ONLY ONE BOOK. SECOND GETS SEATS ARE FULL MESSAGE.

DURABILITY –

IF USING DURABILITY, WAL LOGS ARE SAVED ON THE SSD INSTEAD OF RAM.

SO, IF USING DURABILITY: TRANSACTION STARTS AND COMMITS, WRITES ALL WAL LOGS TO DISK INSTEAD OF RAM. THEN THE SYSTEM CRASHES.

BUT WHEN IT'S RESTARTED, DB RETRIEVES THE WAL LOGS FROM DISK AND WRITES THEM TO THE DATABASE TABLES ON THE DISK.

IF NOT USING DURABILITY: WAL LOGS ARE WRITTEN ON THE RAM, NOT ON THE DISK.

SO WHEN A TRANSACTION IS COMMITTED BUT CHANGES ARE NOT APPLIED TO TABLES YET, AND THE SYSTEM CRASHES.

THE COMMITTED TRANSACTION AND ITS LOGS ARE LOST.



🔁 @Transactional() Propagation

❗️TRANSACTIONAL METHOD MUST BE PUBLIC ❗️

‼️ SEPARATE CLASS AND SEPARATE METHOD CALLED FROM SEPARATE CLASS ‼️

Propagation.REQUIRED (DEFAULT) - We have one method. In that method, we have 2 DB actions. The method opens a transaction, and both DB actions execute in that transaction. If the first action fails second action is not executed. If the first succeeds and the second fails, then the first is being rollbacked too. So the atomicity principle is applied here. 

Propagation.REQUIRES_NEW - always starts a new independent transaction. So if one or both db actions failed in the transferMoney method, but we want to still save the log whether they fail or not, we define a separate class and a separate method, logTransfer, which has Propagation.REQUIRES_NEW and call it within our transferMoney method. 

Propagation.NESTED - similar to Propagation.REQUIRES_NEW, but when done within a method which has other db action too, it does not suspend the existing transaction and does not open a new one, but uses the existing transaction, and if it fails, just rollbacks the certain db action which had the NESTED annotation. so the only difference is it doesnot suspend the existing transaction, but uses it to do its action. 

Propagation.SUPPORTS - if the SUPPORTS annotated method is called within a method that has opened a transaction, it uses that transaction. If there is no open transaction, it runs without a transaction. Good for read-only or optional DB actions.

Propagation.NOT_SUPPORTED - db action annotated with NOT_SUPPORTED, MUST NOT RUN WITHIN ANY TRANSACTION. If there is any existing transaction, it suspends its execution and then runs without a transaction. 

propagation = Propagation.NEVER - same like NOT_SUPPORTED, it MUST NOT RUN WITHIN ANY TRANSACTION if there is any existing transaction, it throws an IllegalTransactionStateException, instead of just suspending the transaction.


propagation = Propagation.MANDATORY - method with MANDATORY must run within an already created transaction. It does not create a transaction on its own, and if no transaction exists, it throws an IllegalTransactionStateException.



🧪 SQL ISOLATION LEVELS

READ_UNCOMMITTED - Dirty Read: A transaction with READ_UNCOMMITTED can read data that is being processed by another transaction but has not been committed yet. so the problem is that if the other transactions failed and rollbacked. It will read the data that is not committed and simply does not exist. Non-Repeatable Read: when we make a READ_COMMITTED transaction to read the quantity and at the same time another transaction edits the quantity table, we will see the edited quantity. Phantom Read: if a new row is added to the table by another transaction same time, we will also see the new row.

Dirty Read: ✅ YES

Non-Repeatable Read: ✅ YES

Phantom Read: ✅ YES

READ_COMMITTED (default) - transaction can only read data which is already commited by other transaction. No dirty read. But when we make a READ_COMMITTED transaction to read quantity and at the same time another transaction edited the quantity table and commits it, we will see the edited quantity. Phantom Read: if a new row is added to the table by another transaction same time and was commited, we will also see the new row.

Dirty Read: ❌ NO

Non-Repeatable Read: ✅ YES

Phantom Read: ✅ YES

REPEATABLE_READ - transaction can only read data which is already commited by other transaction. No dirty read. Also, if we want to read a quantity table and same time other transaction edits that table, we will not see that edit. We will see the previous value of quantity before it was edited, so there is no Non-Repeatable Read. but phandom read is still there. Phantom Read: if a new row is added to the table by other transaction same time and was commited, we will also see the new row.

Dirty Read: ❌ NO

Non-Repeatable Read: ❌ NO

Phantom Read: ✅ YES

SERIALIZABLE - complete isolation, no dirty read, no Non-Repeatable Read, and no phantom read.

Dirty Read: ❌ NO

Non-Repeatable Read: ❌ NO

Phantom Read: ❌ NO



🔒 Locking Levels

Row-Level Locking - locks only the specific row from other transactions from touching it.

Page-Level Locking - 100 users, 1-40 - 1 page, 41 - 80 - 2 page. If user 29 is being modified, 1 - 40 users rows will be blocked.

Table-Level Locking - locks the entire table for one transaction.



🧷 @Transaction - PARAMETERS

Propagation


Isolation


timeout - Specifies the timeout (in seconds) for the transaction. If the transaction takes longer than this, it will be rolled back.

readOnly - Indicates whether the transaction is read-only. This can help with optimization and tells the transaction manager and database that no data modification will occur.

rollbackFor - Specifies which exceptions should cause a rollback of the transaction. By default, only unchecked exceptions (RuntimeException and Error) cause rollback.

rollbackForClassName - Same as rollbackFor but specifies exception class names as Strings.


noRollbackFor - Specifies exceptions that should not trigger a rollback. Overrides default behavior.

noRollbackForClassName - Same as noRollbackFor but with exception class names as Strings.



