---
layout: post
title: "[GO] Orchid Database"
date: 2025-09-19
permalink: /orchid-database/
author: Nate Maxwell
categories:
    - "Go"
tags:
    - "database" 
    - "go"
    - "golang"
    - "data"
    - "structure"
    - "algorithm"
    - "application"
---

Purely driven by curiosity I have spent the last couple of weeks learning about
the inner workings of databases. Not how to set one up using an existing type
like sql, but what is actually happening in the `.db` file and how they're
interacted with. Like all things in computer science, I've found that it's
actually simple in explanation and very cool in implementation. Databases are
primarily driven by two concepts: data structures, and optimizing memory
layout.

To study for this I bought and read Alex Petrov's [Database Internals](https://www.amazon.com/Database-Internals-Deep-Distributed-Systems/dp/1492040347/ref=sr_1_1?crid=1BPDQVPCG2VHZ&dib=eyJ2IjoiMSJ9.Vqd8I6ia0xVn9mpdu96stg.P40u58TrjmMvjYuRcKnaX71rIK6oMylRQXth9KeWCKw&dib_tag=se&keywords=database+internals+alex+petrov&qid=1758314510&sprefix=database+internals%2Caps%2C151&sr=8-1).
This helped immensely with learning the primary underlying data structure and
the higher level concepts surrounding most databases. Mainly this gave me a
frame of reference for how to lay out the entire project using Petrov's layered
architecture

# Data Structure and Memory

Databases are primarily just blocks of bytes stored in a file. Shocking, I
know. Typically, these are segments of 4 KiB blocks, although sometimes are 8
or 16 KiB blocks and are referred to as "pages". The most common size is 4 KiB
as that is the same size as the standard operating system memory page, making
it the fastest and most space efficient amount of contiguous data the operating
system can load at once. Some databases use 8 or 16 KiB pages which still load
neatly into 2 or 4 OS memory pages with the advantage performing less read
operations from disk. The trade-off of larger database pages is that they often
end up with larger amounts of empty space when a user inserts a piece of data
smaller than the page size.

Having a consistent size means that the storage engine can simply open a file
and read `pageNum * pageSize` bytes into the file to get the start of a page.
Almost always developers will mark the start of a page with a "magic number"
which is a sequence of bytes that denotes the start of a page. This is to
verify that the page offset was correctly determined and that no drifting
occurred when inserting/updating pages. It would also be a massive coincidence
if another program placed the exact byte values you were looking for at the
exact offset you were reading, giving us some form of file validation.

A page is just an array of bytes. Anything can be stored in a page, although
there are some common configurations that many databases use. The most common
types of pages are 'meta' pages that act like table of contents for the file,
'freelist' pages which track pages that are no longer used, and the primary
data page that houses user-inserted data, as well as pointers to child pages.

<img src="https://i.imgur.com/EzDb1p9.png">

Sometimes there are 'overflow' pages which occur when data that is
larger than the size of a page is inserted and the extra data must be stored in
an additional page.

The goal is to minimize the number of times a query must read from the primary
data source and databases commonly use B-Tree or LSM tree data structures
(depending on if they persist to disk or are primarily stored in memory) to
optimize the number of disk hops to get to the requested data.

For my database I chose to persist data to disk, meaning the B-Tree was better,
and also easier to implement. B-Trees are a tree data structure that is
a self-balancing tree like binary trees, but with reduced height by storing
multiple values in a node. Internal nodes store t number values where t is the
degrees of the tree. This makes it more ideal for high latency systems such as
disk storage.

Traversing a B-Tree involves starting at the root, compare the target key with
the keys in the current node. If it matches, return it. If it’s smaller (or
falls between two keys), follow the corresponding child pointer. Repeat until
the key is found or a leaf is reached.

If inserting a key into a full node, create two new half-full nodes by moving
the median key up into the parent. Keys less than the median stay in the left
node, keys greater go in the right node. If the parent is also full,
recursively split it, possibly creating a new root.

<img src="https://i.imgur.com/ZfOUEBx.png">

Now from this diagram, you can see in the final tree, if we want to request 5
we only have 3 disk hops to perform to get to the requested value.

Most databases persist data to disk, but some only store data in memory. These
are referred to as 'main memory' databases. Storing purely in memory has the
advantage of being orders of magnitude faster than reading from disk. Main
memory databases may persist data to disk as a recovery feature so that the
tables can be repopulated in the event of a power-loss, although some databases
like Redis are purely ephemeral and do not persist data for any reason.

# Architecture

<img src="https://i.imgur.com/Ktovko1.png">

Diagram taken from Database Internals.

### Transport Layer
Most databases use a client/server model where they manage requests among nodes
or database instances. This sends incoming requests to the query parser, to
other shards on a network, or back to the requesting client.

### Query Processor
This layer has two primary purposes. The first is to parse the custom query
syntax into processable queries. Luckily for me, I've spent quite a bit of time
working with parsers during a long stretch of interpreted language development.

Next is the query optimizer. For databases with more complex ways of
representing the queried data, such as relational or hierarchical databases,
the query optimizer plans out the best strategy read/write to reduce the number
of database pages to iterate over.

### Execution Engine
From here, the assembled queries are given to the execution engine where the
pulls the data from disk locally, or from another shard.

### Storage Engine
Lastly there is the storage engine, the real functional part that we typically
think of as the actual database. This manages transactions (lists of pages
affected by the query), recovery mechanisms, the primary read/write methods,
locking components, and so on. Basically, anything that handles the actual
stored data.

# SQL vs NoSQL
Databases are typically classified into two broad families: Sql and NoSql.

Sql databases, broadly speaking, are any relational database that uses
Structured Query Language to manage tables and rows by creating conditional
table joins to manage an in-memory table that is the result of a query.
Sql-like databases are the most common kind. Think `SQLite`, `MySQL`,
`PostgreSQL`, `Oracle`, etc.

NoSql databases are pretty much all other databases that do not fit the Sql
description.

Some examples are:

* Key-Value stores like `Redis`.
* Hierarchical databases like `MongoDB`, which have a JSON like structure.
* Wide-column stores like `ScyllaDB` or `Cassandra`. These have tables with rows
and columns, but unlike relational databases, the names and format of columns
can vary from row to row.

# Orchid DB

After intensely researching night after night, suddenly realizing it was 3am, I
decided to make an attempt at my own database.

<img src="https://i.imgur.com/BB4lfvi.png">

I chose a KV store as the data structure is simpler, for learning purposes, and
it can reasonably emulate relational table functionality by staggering the data
over multiple KV tables. For example, you might query a user guid from a string
name, then using that guid user ID, query the phone contacts table for their
phone number and the address contacts table for their mailing address, and the
email contacts table for their email address, and so on.

Orchid persists data to disk, but has an in-memory cached table layer that
`GET` requests read from first, before querying disk, as memory is orders of
magnitude faster than disk reads.

Additionally, my recovery mechanism of choice was through Write-Ahead-Logs.
These are logs that get written out, detailing the updates that the query will
attempt. In the event of a power-loss or unexpected shutdown scenario, the logs
are used to replay the last transaction. If the WAL log is missing a successful
log token at the end of the file, then we know that the power-loss occurred
during log writing, and we cannot know the intent of the failed transaction, so
it is completely discarded.

Databases use atomic read/writes, meaning that a query reads or writes all
the data or none at all. Using these Write-Ahead-Logs in combination with Go's
`sync.RWMutex`, we achieve atomicity both in unexpected shutdowns and
preventing 'ghost reads'. These are reads in which a `GET` request was received
but the underlying data was transformed between the time the request was
received and when the data is actually retrieved, making it a query received
for one database state, but then processed under another. This still happens in
the sense that the `sync.RWMutex` creates a queue, but the data cannot
transform in the middle of that request's transaction.

Using the mutex also means that we can have any number of readers at one time
or a single writer, but never both or more than one writer.

Lastly, I chose to make every table its own `.db` file so that I could stick a
"worker" in front of it on its own thread, managing the CRUD methods to that
table/file. Workers have a shutdown timer where if a query isn't sent to the
worker before the timer expires, the worker is shut down as the table is not
likely to be requested frequently. This allows the database to use n number OS
threads where n is the number of active, or hot, tables.

<img src="https://i.imgur.com/YsS0QXS.png">

OrchidDB has its own query syntax, which is pretty simplistic at present. There
are currently only 5 commands:

* `MAKE(table)`
* `DROP(table)`
* `GET(table, key)`
* `PUT(table, key, value)`
* `DEL(table, key)`

These are processed through pretty traditional lexer + parser + evaluators.
There is probably a much more efficient way that databases parse queries, but
this way I know from my previous interpreted language development and is pretty
scalable for additional commands and increasing complexity in the query
grammar.

### WIP
Currently, OrchidDB has all but the in-memory table cache and the table worker
timers. For the time-being I am putting the project down as I am moving for a
new job.

# Distributed Back-End

Now I have my own message broker, [Mycelia](https://nate-maxwell.github.io/go-mycelia-event-broker/),
which I've written about in the past, and my own database. Next I would like to
look into making a simplistic container app to host back-end systems inside.
Then I will have a container to deploy systems in, a database for them to
manage, and a broker for them to communicate, all made entirely by myself.

Perhaps I will add some infrastructural features to each of them, so they work
nicely with each other and release them as an application suite in the future.
