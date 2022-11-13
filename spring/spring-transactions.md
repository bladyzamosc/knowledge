## Spring transactions

### 1. Transaction kinds 

- Global - EJB CMT  container manager transaction, for more resources 
- Local - for only one resource like DB or JMS

### 2. Spring transaction strategy - TransactionManager

- **PlatformTransactionManager**:

1. getTransaction() returns TransactionStatus
2. commit
3. rollback
4. TransactionException - unchecked, a child for RuntimeException
5. TransactionStatus can represent a new or new transaction, associated with the current thread

- **ReactiveTransactionManager**:

1. getRactiveTransaction returns Mono<ReactiveTransaction>
2. commit Mono<void>
3. rollback Mono<void>

- **TransactionDefinition**:
1. propagation - require, requires_new, nested, supports (is transaction will be active it will use it, if there is not transaction the method won’t be  invoked inside the transaction), mandatory, never, not_support
2. isolation
3. timeout
4. read-only
5. api - isNewTransaction(), hasSavepoint(), setRolbakconly, isRollbackonly
   , flush(), isCompleted()

### 3. setRollbackOnly() - what causes 

The transaction won't be finished.

### 4. Rollback rules

- unchecked + errors - by default rollback
- checked - by default will be not rollback
- the behaviour can be changed rollbackFor(), noRollbackFor()

### 5. @Transactional

- only for public methods 
- default = proxy
- proxy target class = false by default and standard JDK interface-based proxies are created
- settings default 
1. propagation - PROPAGATION_REQUIRED
2. isolation - ISOLATION_DEFAULT
3. transaction is read-write
4. timeout - of the underlying transaction system
5. RuntimeExceptions and Error triggers rollback and checked exceptions not

