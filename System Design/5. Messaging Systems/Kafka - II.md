
## What is Offset?

#### First understand the problem

Imagine WhatsApp.

Your friend sends you 100 messages.

```
Message 1
Message 2
Message 3
...
Message 100
```

When you reopen WhatsApp, how does it know you already read till Message 68?

It needs to remember

> "Last read position = 68"

Kafka has exactly the same problem.

---

### Kafka Topic

Suppose we have an Order Topic.

```
Order Topic

+---------------------------------------+
| Order-1 |
| Order-2 |
| Order-3 |
| Order-4 |
| Order-5 |
| Order-6 |
+---------------------------------------+
```

Kafka stores messages sequentially.

Each message gets a unique number.

```
Offset

0 -> Order-1
1 -> Order-2
2 -> Order-3
3 -> Order-4
4 -> Order-5
5 -> Order-6
```

Offset is simply

> **The position (index) of a message inside a partition.**

Think of it as a page number in a book.

---

#### Book Analogy

Suppose you're reading Harry Potter.

You stop at page 248.

Tomorrow you don't start from page 1.

You start from page 249.

Why?

Because you remember

```
Last Read Page = 248
```

Kafka remembers

```
Last Read Offset = 248
```

Exactly same idea.

---

#### Real World Example

Amazon Order Service

Producer sends

```
Order Created
```

Topic

```
Offset 0 -> Order A
Offset 1 -> Order B
Offset 2 -> Order C
Offset 3 -> Order D
Offset 4 -> Order E
```

Consumer processes

```
A ✔
B ✔
C ✔
```

Kafka stores

```
Current Offset = 3
```

Meaning

Next message to read

```
Offset 3
```

---

#### Why Offset is Needed?

Without offsets...

Consumer crashes.

After restart...

Where should it begin?

```
Order A ?

Order B ?

Order C ?

Order D ?
```

Impossible to know.

Offset solves this.

---

#### Where are Offsets Stored?

Kafka stores them in a special internal topic.

```
__consumer_offsets
```

Kafka itself manages this automatically.

---

#### Consumer Crash Example

Imagine

```
Offset 0 ✔

Offset 1 ✔

Offset 2 ✔

Offset 3 (processing...)

Offset 4

Offset 5
```

Consumer suddenly crashes.

Question:

Where should it restart?

Answer depends on whether offset was committed.

---

#### What is Commit?

Commit means

> "Kafka, I have successfully processed everything till Offset X."

For example

```
Processed till Offset 2

Commit Offset = 3
```

Kafka remembers

```
Start from Offset 3 next time.
```

---

#### What if Consumer Crashes BEFORE Commit?

Let's see.

Messages

```
0 A
1 B
2 C
3 D
```

Consumer

```
Read A ✔

Read B ✔

Read C ✔

Crash 💥
```

Suppose it never committed.

Kafka still thinks

```
Last committed = Offset 1
```

After restart

Kafka sends again

```
Offset 2
Offset 3
```

Consumer processes

```
C again

D
```

So

```
C processed twice
```

This is called

```
Duplicate Processing
```

Kafka prefers duplicates over losing data.

---

#### What if Offset Was Already Committed?

Consumer

```
Read C

Commit Offset

Crash
```

Restart

Kafka starts

```
Offset 3
```

Everything is fine.

---

#### But What if Consumer Commits Too Early?

Very dangerous.

Suppose

```
Read Order-100

Immediately Commit

Then process...
```

Suddenly

```
Consumer crashes
```

Order was never processed.

Kafka thinks

```
Already done.
```

Message is skipped forever.

Data Lost.

This is why commit timing matters.

---

#### Safe Sequence

Correct order

```
Read Message

↓

Process Message

↓

Save in Database

↓

Commit Offset
```

Never

```
Read

↓

Commit

↓

Process
```

---

#### Why Kafka Doesn't Automatically Remove Messages?

Unlike RabbitMQ.

Kafka keeps everything.

Even after processing.

```
Offset 0
Offset 1
Offset 2
Offset 3
Offset 4
```

Consumer simply moves its offset.

Messages stay.

Another consumer can read from beginning.

---

#### Real Company Example

Suppose Flipkart.

Topic

```
Orders
```

Consumers

```
Inventory
Payment
Notification
Analytics
```

Each consumer has different offsets.

```
Inventory -> Offset 500

Payment -> Offset 490

Analytics -> Offset 320

Notification -> Offset 500
```

Same topic.

Different progress.

That's possible because offsets are tracked per **consumer group**.

---

## Replication

This is another important concept.

---

#### First Problem

Imagine Kafka has only one server.

```
Broker 1

Orders Topic
```

Suddenly

```
Server crashes 💥
```

All messages gone.

Very bad.

---

Kafka solves this using Replication.

Instead of one copy

Keep multiple copies.

---

Suppose Replication Factor = 3

```
Broker 1

Orders Partition 0
```

Copies

```
Broker 1
Partition 0

Broker 2
Partition 0 Copy

Broker 3
Partition 0 Copy
```

Now we have

```
3 copies
```

---

### Leader and Followers

Among replicas

One becomes

```
Leader
```

Others become

```
Followers
```

Example

```
Broker1
Partition0 (Leader)

Broker2
Partition0 (Follower)

Broker3
Partition0 (Follower)
```

---

#### Why Leader?

Imagine

Three people editing same Google Doc simultaneously.

Chaos.

Instead

One person accepts changes.

Others copy.

Kafka follows the same idea.

Only the leader accepts reads/writes.

Followers replicate from the leader.

---

### Producer Writes

Producer

```
New Order
```

Producer always sends to

```
Leader
```

Never follower.

```
Producer

↓

Leader
```

---

Leader stores

```
Offset 100
```

Then sends

```
Follower 1

Follower 2
```

Followers copy data.

```
Leader

↓

Follower A

↓

Follower B
```

Now all have same data.

---

#### What if Leader Crashes?

Suppose

```
Broker1

Leader

💥 Crash
```

Kafka notices.

It elects one follower.

```
Broker2

New Leader
```

Producer now sends here.

```
Producer

↓

Broker2
```

System continues.

Very little downtime.

---

#### Visual

Before crash

```
Leader

↓

Follower

↓

Follower
```

After crash

```
Leader ❌

Follower

↓

New Leader ✔

↓

Follower
```

---

### How Does Kafka Choose New Leader?

Kafka chooses one of the replicas that is fully caught up (called an in-sync replica, or ISR).

If followers are behind and not fully synchronized, they are usually not chosen to avoid losing acknowledged data.

---

### Real Example

Amazon Orders

Replication = 3

```
Broker1

Leader
```

```
Broker2

Follower
```

```
Broker3

Follower
```

Customer places order.

Producer

```
Order #999
```

Leader stores it.

Followers copy.

Now

```
3 copies exist.
```

Suddenly

```
Broker1 dies.
```

Broker2 becomes leader.

Customers continue placing orders.

Nobody notices.

---

### Complete Flow Together

```
Producer↓Leader Partition↓Followers replicate↓Consumer reads↓Processes↓Commit Offset↓Kafka stores offset
```

---

#### Putting It All Together with a Real E-commerce Example

Imagine an **Orders** topic with one partition:

```
Offset 0 -> Order A
Offset 1 -> Order B
Offset 2 -> Order C
Offset 3 -> Order D
```

1. The **Producer** sends `Order C` to the **leader** partition.
2. The leader writes it to disk and replicates it to the follower brokers.
3. A **Consumer** reads `Order C` (offset 2).
4. The consumer updates the database successfully.
5. The consumer commits the offset (`3`), meaning "start from offset 3 next time."

Now consider failures:

- **Consumer crashes before committing:** Kafka still believes offset 2 hasn't been completed, so `Order C` will be delivered again after restart. This may cause duplicate processing, which is why consumers should make their processing idempotent (safe to repeat).
- **Consumer commits before processing:** If it crashes after committing but before updating the database, Kafka will start from offset 3 on restart, and `Order C` is effectively lost to that consumer.
- **Leader broker crashes:** Since followers already have replicated copies, Kafka elects an up-to-date follower as the new leader. Producers and consumers automatically switch to the new leader and continue working.

-----
-----
----
-----

