---
title: "DB Transaction"
categories:
  - Computer Science Background
  - Database
date: 2022-02-08 15:00:25 +0900
tags:
  - database
  - golang
  - postgres
---

# What is Database Transaction?
* It refers to the unit of work performed by changing the state of the database.
* Simply put, it means accessing the database using the following query(SELECT, UPDATE, DELETE, ALTER, etc.).
> Transaction can be figured as multiple queries.

# Why do we need transaction?
* To provide reliable and consistant unit of work, even if system fails.
* To provide isolation between programs that access to database at same time(concurrently).

# ACID(Atomicity, Consistency, Isolation, Durability)
Transaction has 4 characters.
* **Atomicity** : Whether all transactions are reflected in the database or not at all(all or none).
* **Consistency** : The result of transaction processing should always be consistent. When we access to value in database, even if that value is updated, the operation must be performed with the value initially referenced.
* **Isolation** : If one transaction perform calculation in specific value, other transaction can not access to that value.
* **Durability** : When transaction is done successfully, results must be reflected to database.

> Transaction use callback and rollback to maintain 4 characters.

Here is a example how we can use transaction in golang(**Querier** is made by sqlc).
```go
type Store interface {
	Querier
	TransferTx(ctx context.Context, arg TransferTxParams) (TransferTxResult, error)
}

// 쿼리들을 저장
type SQLStore struct {
	*Queries //앞의 Queries 생략하면 *Queries에서 가져와 만들어짐.
	db       *sql.DB
}

// 만들어진 Store 주소반환
func NewStore(db *sql.DB) Store {
	return &SQLStore{
		db:      db,
		Queries: New(db),
	}
}

// 쿼리문을 실행한다. Beigin, Rollback조건, commit로 최종실행 확인
func (store *SQLStore) execTx(ctx context.Context, fn func(*Queries) error) error {
	tx, err := store.db.BeginTx(ctx, nil) // BEGIN
	if err != nil {
		return err
	}
	q := New(tx)    //
	err = fn(q)     // EXECUTE
	if err != nil { // ROLLBACK
		if rbErr := tx.Rollback(); rbErr != nil {
			return fmt.Errorf("tx err: %v, rb err:%v", err, rbErr)
		}
		return err
	}

	return tx.Commit() // COMMIT
}
```
{: file='store.go'}