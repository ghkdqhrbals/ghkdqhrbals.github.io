---
title: "Banking backend server with Golang(1)"
categories:
  - Server
date: 2022-02-02 06:00:25 +0900
tags:
  - golang
  - backend
---
# What is this blog?

Recently, I felt that my programming skill level is low.

> When I was studying for bachelor degree in computer science, I focus programming and CS **theoretically**.   
> When I'm studying now for master degree, I also focus on blockchain and DL/ML knowledge **theoretically**.

That means, my CS knowledge is based on theory, **not on practical solution**.

So I will use this blog as a note to record not only theoretical knowledge but also **PRACTICAL** knowledge.

Banking backend server with Golang series, as the name suggests, will develop a banking backend server and it will be a long journey.

> As purpose of this blog is just for recording, there might be something missing. All posts are just scribbled, so the grammar will be wrong.
{: .prompt-warning }

Anyway, this is my first post of development. And now I'm going to develop banking backend service from start to finish.

I will use these skills below.

### Skills

| Skills          | Purposes                                            |
| --------------- | --------------------------------------------------- |
| RDS(Postgresql) | storing User,accounts,balance info                  |
| migration       | auto-migration                                      |
| sqlc            | generate Golang interface from sql                  |
| git-workflow    | auto-deploy                                         |
| gin             | HTTP communication                                  |
| bcrypt          | safe way to store PW                                |
| Viper           | auto-server configuration setting                   |
| Gomock          | testing RDS                                         |
| Docker          | auto-setting env and ease for run                   |
| Kubernetes      | auto-scaling and managing pods(docker images)       |
| JWT or PASETO   | TOKEN based authentication(reduce session weight)   |
| JQ              | conversion JSON to txt                              |
| AWS             | get fixed public IP and for automation, maintenance |

# Database design

First I'm going to write DB scheme using [dbdiagram](https://dbdiagram.io).   
We can make general sql code as below.

![img-description](../../assets/p/1/db_diagram.png)

```sql
Table users as U{
  username varchar [pk]
  hased_password varchar [not null]
  full_name varchar [not null]
  email varchar [unique, not null]
  password_change_at timestamptz [not null,default:'0001-01-01 00:00:00Z']
  created_at timestamptz [not null, default: `now()`]
}

Table accounts as A {
  id bigserial [pk]
  owner varchar [ref: > U.username ,not null]
  balance bigint [not null]
  currency varchar [not null]
  created_at timestamptz [not null, default: `now()`]

  Indexes {
    owner
    (owner, currency) [unique]
  }
}

Table entries {
  id bigserial [pk]
  account_id bigint [ref: > A.id, not null]
  amount bigint [not null, note: 'can be negative or positive']
  created_at timestamptz [not null, default: `now()`]

  Indexes {
    account_id
  }
}

Table transfers {
  id bigserial [pk]
  from_account_id bigint [ref: > A.id, not null]
  to_account_id bigint [ref: > A.id, not null]
  amount bigint [not null, note: 'must be positive']
  created_at timestamptz [not null, default: `now()`]

  Indexes {
    from_account_id
    to_account_id
    (from_account_id, to_account_id)
  }
}
```
{: file="sql" }

Now using [dbdiagram](https://dbdiagram.io), we can change sql code above into postgresql code.   

![img-descriptio2](../../assets/p/1/export.png)

```sql
CREATE TABLE "users" (
  "username" varchar PRIMARY KEY,
  "hased_password" varchar NOT NULL,
  "full_name" varchar NOT NULL,
  "email" varchar UNIQUE NOT NULL,
  "password_change_at" timestamptz NOT NULL DEFAULT '0001-01-01 00:00:00Z',
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "accounts" (
  "id" bigserial PRIMARY KEY,
  "owner" varchar NOT NULL,
  "balance" bigint NOT NULL,
  "currency" varchar NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "entries" (
  "id" bigserial PRIMARY KEY,
  "account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "transfers" (
  "id" bigserial PRIMARY KEY,
  "from_account_id" bigint NOT NULL,
  "to_account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE INDEX ON "accounts" ("owner");
CREATE UNIQUE INDEX ON "accounts" ("owner", "currency");
CREATE INDEX ON "entries" ("account_id");
CREATE INDEX ON "transfers" ("from_account_id");
CREATE INDEX ON "transfers" ("to_account_id");
CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

COMMENT ON COLUMN "entries"."amount" IS 'can be negative or positive';
COMMENT ON COLUMN "transfers"."amount" IS 'must be positive';

ALTER TABLE "accounts" ADD FOREIGN KEY ("owner") REFERENCES "users" ("username");
ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");
ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");
ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");
```
{: file="Postgresql"}

So now we got SQL code for migrate to our postgres databse.