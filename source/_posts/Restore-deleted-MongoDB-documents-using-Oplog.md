---
title: Restore deleted MongoDB documents using Oplog
date: 2018-06-15 01:05:00
tags:
  - data
  - mongo
category:
  - data
photos: data/images/Restore-deleted-MongoDB-documents-using-Oplog/cover.png
---
<sup>Source: [Dilbert][dilbert-url]</sup>

Recently I found myself trying to restore corrupted data to it's original form. A query gone wrong had erased valuable data from a MongoDB collection and it needed to be restored. This blog is a means to note my learnings from that incident for my future self and others.

#### Oplog is amazing

If you're using a MongoDB replica set, you can use MongoDB Oplog to undo almost any type of *recent* data loss. If you never heard of it, go read the docs as I'll skip to the restoring part.

{% blockquote MongoDB https://docs.mongodb.com/manual/core/replica-set-oplog/ Official docs %}
The oplog is a special capped collection that keeps a rolling record of all operations that modify the data stored in your databases.
{% endblockquote %}

## Restore backup

Restore whatever latest backup you have before the the data was corrupted. It'll be used as a base. Make sure the oplog has operations from or before the time the backup was taken, otherwise the restore may not work.

## Find the faulty query/queries

Now you need to browse the contents of the Oplog and look for the query which corrupted the data in the first place. Oplog is a special collection, but still a collection. So, you can query it to narrow down your results like this:
```mongo mongodb shell
Replica:PRIMARY> use local
switched to db local
Replica:PRIMARY> db.oplog.rs.find()
```
Let's say you found the query and it looked like:
``` js
...
...
{ "ts" : Timestamp(1528225054, 1), "t" : NumberLong(1), "h" : NumberLong("3208671813197906204"), "v" : 2, "op" : "d", "ns" : "test.foo", "ui" : UUID("348dd681-e0df-4d6b-bd69-304d21cf8235"), "wall" : ISODate("2018-06-05T18:57:34.245Z"), "o" : { "_id" : ObjectId("5b16dd0d5861982c59fedefe") } }
...
...
```
Note down the value of field `ts`. It will be needed later.

## Take a dump of oplog

Before you can restore your your from oplog, you need to dump it to a file.
```sh mongodump https://docs.mongodb.com/manual/reference/program/mongodump/ docs
mongodump -d <DBNAME> -c oplog.rs -o oplogD
mkdir oplogR
mv oplogD/<DBNAME>/oplog.rs.bson oplogR/oplog.bson
```
Now you have a dump of the oplog in `oplogR/oplog.bson`.

## Restore from Oplog

You want to restore the data prior to the faulty write operation.

```sh mongorestore https://docs.mongodb.com/manual/reference/program/mongorestore/ docs
mongorestore --oplogReplay --oplogLimit 1528225054:1 oplogR
```
The value of `oplogLimit` paramter is the `ts` property of the faulty query you noted earier.

**Mongo oplog is *idempotent*, i.e. it can be applied multiple times without duplicating data. This is why you need not give `mongorestore` a start time.**

## Selective restore

The previous method helps when the good and bad operations are clearly separated by time. But that's not the case mostly. It's likely that the good and bad writes happened around the same time.

Here, you need to remember that Oplog is a collection too, and in addition to be queried on, it can also be modified. You need to copy oplog to an uncapped collection and delete the bad write operations.

After you've removed all bad operations you can continue the restoration process from [a dump of the modified oplog][oplog-dump-anchor-url]. But this time, no limit argument would be required.

Hope this helps!
<br>

**❤️ code**
<br>

---
<sub>*This post is inpired from resources like [Asya's stackoverflow answer][so-answer-url] which was used to solve the problem at hand.*</sub>

[dilbert-url]: http://dilbert.com/strip/2013-07-05
[mongo-docs-url]: https://docs.mongodb.com/manual/core/replica-set-oplog/
[oplog-dump-anchor-url]: #Take-a-dump-of-oplog
[so-answer-url]: https://stackoverflow.com/a/15451297/2751596
