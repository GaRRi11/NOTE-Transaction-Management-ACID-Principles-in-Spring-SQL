# NOTE-Transaction-Management-ACID-Principles-in-Spring-SQL

⚙️ TRANSACTION STEPS

BEGIN → NEW ID IS ASSIGNED TO TRANSACTION

EXECUTE → CHANGES ARE MADE BUT IN THE RAM AND STORED AS CACHE.

WAL LOG → BEFORE WRITING ANYTHING IN THE DISK, CHANGES IS WRITTEN IN THE WAL LOGS AND BEING SAVED IN RAM.

COMMIT → DB CONFIRMS THAT ALL WAL LOGS ARE ON RAM.

APPLY → AFTER COMMIT, CHANGES ARE WRITTEN FROM RAM TO ACTUAL SSD DB TABLES.

WAL LOG CLEAN → AFTER DATA IS WRITTEN TO DISK. WAL LOGS ARE CLEANED FROM RAM.



🔐 ACID

ATOMICITY – ENSURES WHETHER BOTH FAIL OR BOTH SUCCED.

CONSISTENCY – MAINTAINS VALID DATA, WHEN ACC HAS 500$, 1000$ CAN NOT BE WITHDROWEN

ISOLATION – TWO THREAD TRIES TO BOOK THE LAST SEAT. SYSTEMS LETS ONLY ONE TO BOOK. SECOND GETS SEATS ARE FULL MESSAGE.

DURABILITY –

IF USING DURABILITY, WAL LOGS ARE SAVED ON THE SSD INSTEAD OF RAM.

SO IF USING DURABILITY: TRANSACTION STARTS AND COMMITS, WRITES ALL WAL LOGS IN DISK INSTEAD OF RAM. THEN SYSTEM CRASHS.

BUT WHEN ITS RESTARTED, DB RETREIVS THE WAL LOGS FROM DISK AND WRITED THEM ON THE DATABASE TABLES ON THE DISK.

IF NOT USING DURABILITY: WAL LOGS ARE WRITTEN ON THE RAM NOT ON THE DISK.

SO WHEN TRANSACTION IS COMMITED BUT CHANGES ARE NOT APPLIED ON TABLES YET AND SYSTEM CRASHS.

THE COMMITED TRANSACTION AND ITS LOGS ARE LOST.



🔁 @Transactional() Propagation

❗️TRANSACTIONAL METHOD MUST BE PUBLIC ❗️

‼️ SEPARATE CLASS AND SEPARATE METHOD CALLED FROM SEPARATE CLASS ‼️

Propagation.REQUIRED (DEFAULT) - we have one method. in that method we have 2 db actions. the method opens transaction and both db actions execute in that transaction. if first action failed second action is not being executed. if first succedd and second failed, then first is being rollbacked too. so the atomicity principle is applied here. -

Propagation.REQUIRES_NEW - always starts new independed transaction. so if one or both db actions failed in transderMoney method, but we want to still save the log whether they fail or not we define separate class and separate method logTransfer which has Propagation.REQUIRES_NEW and call it within our transferMoney method. 

Propagation.NESTED - same like Propagation.REQUIRES_NEW but when done within method which has other db action too, it doesnot suspend the existing transaction and doesnot open a new, but uses the existing transaction and if it fails just rollbacks the certain db action which had NESTED annotation. so the only difference is it doesnot suspend the existing transaction, but uses it to do its action. 

Propagation.SUPPORTS - if SUPPORTS annotated method is called within a method which has opened a transaction, it uses that transaction. if there is no opened transaction, it runs without transaction. good for read only or optional db actions.

Propagation.NOT_SUPPORTED - db action annotated with NOT_SUPPORTED, MUST NOT RUN WITHIN ANY TRANSACTION if there is any existing transaction it suspends its and then runs without transaction. 

propagation = Propagation.NEVER - same like NOT_SUPPORTED, it MUST NOT RUN WITHIN ANY TRANSACTION if there is any existing transaction it throws an IllegalTransactionStateException, instead of just suspending the transaction.


propagation = Propagation.MANDATORY - method with MANDATORY must run within a already created transaction. it does not create a tranaction on its own, and if no tranaction exist, it throws IllegalTransactionStateException.



🧪 SQL ISOLATION LEVELS

READ_UNCOMMITTED - Dirty Read: transaction with READ_UNCOMMITTED can read data which is being proccesed by other transaction but no commited yet. so problem is that if the other transactions failed and rollbacked. he will read the data which is not commited and simply does not exist. Non-Repeatable Read: when we make a READ_COMMITTED transaction to read cuantity and at the same time other transaction edited the cuantity table and commited it we will the the edited quantity. Phantom Read: if new row is added to table by other transaction same time and was commited, we will also see the new row.

Dirty Read: ✅ YES

Non-Repeatable Read: ✅ YES

Phantom Read: ✅ YES

READ_COMMITTED (default) - transaction can only read data which is already commited by other transaction. no dirty read. but when we make a 
READ_COMMITTED transaction to read cuantity and at the same time other transaction edited the cuantity table and commited it we will the the edited quantity. Phantom Read: if new row is added to table by other transaction same time and was commited, we will also see the new row.

Dirty Read: ❌ NO

Non-Repeatable Read: ✅ YES

Phantom Read: ✅ YES

REPEATABLE_READ - transaction can only read data which is already commited by other transaction. no dirty read. also if we want to read quantity table and same time other transaction edited that table we will not see that edit. we will see the previous value of quantity before it was edited so there is no Non-Repeatable Read. but phandom read is still there Phantom Read: if new row is added to table by other transaction same time and was commited, we will also see the new row.

Dirty Read: ❌ NO

Non-Repeatable Read: ❌ NO

Phantom Read: ✅ YES

SERIALIZABLE - complete isolation no dirty read no Non-Repeatable Read and no phantom read.

Dirty Read: ❌ NO

Non-Repeatable Read: ❌ NO

Phantom Read: ❌ NO



🔒 Locking Levels

Row-Level Locking - locks only the specific row from other transactions from touching it.

Page-Level Locking - 100 users, 1-40 - 1 page, 41 - 80 - 2 page. if user 29 is being modified 1 - 40 users rows will be blocked.

Table-Level Locking - locks entire table for one transaction.



🧷 @Transaction - PARAMETERS

Propagation


Isolation


timeout - Specifies the timeout (in seconds) for the transaction. If the transaction takes longer than this, it will be rolled back.

readOnly - Indicates whether the transaction is read-only. This can help with optimization and tells the transaction manager and database that no data modification will occur.

rollbackFor - Specifies which exceptions should cause a rollback of the transaction. By default, only unchecked exceptions (RuntimeException and Error) cause rollback.

rollbackForClassName - Same as rollbackFor but specifies exception class names as Strings.


noRollbackFor - Specifies exceptions that should not trigger a rollback. Overrides default behavior.

noRollbackForClassName - Same as noRollbackFor but with exception class names as Strings.



