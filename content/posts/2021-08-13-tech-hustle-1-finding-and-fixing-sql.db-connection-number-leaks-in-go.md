---
title: "[TECH-HUSTLE] 1 Finding and Fixing sql.DB Connection Number Leaks in Go"
date: 2021-08-13T10:37:25-03:00
draft: true
categories:
  - Golang
  - Tech Hustle
tags:
  - Tech Hustle
  - Technology
  - Database
  - PostgreSQL
  - Golang
featured_image:
---

I always though articles about hairy production problems, really challenging archtecture decisions or game changing performance improvements super awesome and informative. Even though, there's plenty of good content out there, I decided to share some of my personal experiences dealing with production problems and I hope it might help developers out there. I called this series of articles **TECH HUSLTLE**.

---

This posts describes how we found and fixed a memory leak related to Go `sql.DB` connections on PostgreSQL on a syncronous event worker.

This particular event worker is as simple as an AWS SQS messages consumer, running independently on a go routine alongside with a http server.

```go
h := msgqueue.NewHandler(updateUseCase)
go msgqueue.ListenAndHandle(sqsClient, h)

r := http.NewRouter()
http.ListenAndServe(":"+cfg.Port, r)
```

Due to the few amount of messages we're processing, we never put much effort to make this worker async so it gets and processes messages syncronously.

```go
func (l Listener) ListenAndHandle() error {
	for {
		ctx := context.Background()
		msgs, err := l.broker.Messages(ctx)
		if err != nil {
			continue
		}
		for _, msg := range msgs {
			if err := l.handleMessage(ctx, msg); err != nil {
				log.Println("[ERROR]", err)
			}
		}
	}
}
```

Despite its simplicity, when you deployed it to consume aproximatly 60 messages per minute it almost instantaneously took over all available conections on our PostgreSQL database breaking the limit of our db instance on AWS RDS.

<image>

After we're rollbacked the solution, we thought the problem could be related to the fact that our database pool had no configs about
`MaxOpenConns`, `MaxIdleConns` and `ConnMaxLifetime`. Then, we set this configs up and redeployed the sevice.

```go
db, err := database.Open(cfg.DatabaseURL)
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)
db.SetConnMaxLifetime(5 * time.Minute)
```

Also, we reduced the number of messages sent to the server, notifying it only when neeeded.
We noticed that it only slowed down the leak and we endeed up reaching the max number of conns errors after a certain time.

<image>

We assumed it was due to the amount of time of `ConnMaxLifetime` and we reduced it to **1 minute**.

```go
db.SetConnMaxLifetime(1 * time.Minute)
```

We noticed the number of opened connections fall down instantanouly. However, after some time the connections started, slowly, to go up again. We further reduced the `ConnMaxLifetime` to **30 seconds** but the exact same pattern persisted.

After some time brainstorming on possibilities I decided to look up opened connections on the database and I noticied that, even after 30 seconds, the connections remained opened with `wait_type` `ClientRead`. For some reason, the connections got stuck and were never terminated by the `sql.DB` pool configs.

```sql
psql> SELECT * FROM pg_stat_activity;
/* TODO *
```

Then, I decided to take a look on how `sql.DB` handles connections and the internals that make a connection return to the pool, stating from the method we're using to execute the queries `db.QueryContext`.

```go
_, err := db.QueryContext(ctx, "<query>")
```

After reading around the `$GOROOT/src/databse/sql/sql.go` file, I noticed that there was a relation between how connections were deleted from the pool and context being finished via the `channel` `<-ctx.Done()`.

Since we're using `context.Background()` to all our queries comming from the worker, I decided to add I simple **timeout context** and see how the connections on database would behave. Also, I reduced the `ConMaxLifetime` to **3 seconds** to see connections being terminated on database faster.

```go
db.SetConnMaxLifetime(3 * time.Seconds)
```

```go
func (l Listener) handleMessage(ctx context.Context, msg Message) error {
	ctx, cancel := context.WithTimeout(ctx, 1*time.Second)
	defer cancel()
	if err := l.handler.HandleMessage(ctx, []byte(msg.Body)); err != nil {
		return err
	}
	return l.broker.DeleteMessage(ctx, msg.ReceiptHandle)
}
```

And it worked! After **3 seconds** the connection used to run the query was nicely terminated due to our `ConnMaxLifetime` and dissapered as opened connection on the database.

```sql
psql> SELECT * FROM pg_stat_activity;
/* TODO */
```

<image>

---

I hope this history of many attempts, mistakes and investigation can help you out with your own tech hustles.
