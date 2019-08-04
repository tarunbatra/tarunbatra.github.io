
# Delayed triggers using Dead Letter Queues (DLQs)

Execution of computer programs is blazing fast. It's only practical to need some delay in execution. Some of such use cases are:
- scheduling a task for sometime in future;
- retrying a failed task with a backoff stratagy

At smallcase, we place orders to buy or sell equities on clients' behalf. When the order is placed, it's execution status is not immediately known. We need to poll the partner broker to know the status of order on the exchange.

Intraday traders will tell you that stock markets are time sensitive. Any platform that caters to them needs to be fast and deterministic. Polling for order status needs to happen in a timely manner after regular intervals because an endless loading is never a pleasant sight.

!['Loading'][loading-gif]

This blog explains the reliability issues we faced with our legacy system for scheduling polling and how we fixed it.

## Delay mechanisms:
We will discuss the following delay mechanisms we used in our platform:
* [Redis Keyspace Notifications](#Redis-Keyspace-Notifications)
* [In-memory Timers](#In-memoryTimers)
* [Dead Letter Queues](#Dead-Letter-Queues)

### Redis Keyspace Notifications
This approach is very simple. We heavily use Redis in our stack, so it was easy for us to use it for scheduling triggers. Redis allows to set keys with expiry using its [SETEX][redis-setex-doc] command. For instance, setting a key `TESTKEY` with value `TESTVALUE` and expiry of 10 seconds can be achieved with:
```sh
>redis SETEX TESTKEY 10 "TESTVALUE"
```
Parallely, Redis provides [Keyspace notifications][redis-notifications-doc] for data changing events which can be subscribed by clients and can be used to trigger the delayed tasks. Key expiry events can be subscribed using:
```sh
SUBSCRIBE __keyevent@0__:expired
```
#### Pros:
1. **Stateless application code.**
    The application doesn't need to store the tasks to perform and delays in its memory. It is outsourced to Redis.
2. **Arbitrary delays.**
    Delays of arbitrary length are possible without any extra configuration. Just changing the `ttl` paramter of `SETEX` command will work.

#### Cons:
1. **Unscalable.**
    This strategy works only till there's only one instance of the application listening to these events. In case this application is to be horizontally scaled, each instance of the application will receive the event because of its [publish-subscribe nature][pub-sub-ref].
1. **Unreliable.**
    The keyspace notifications can get highly unreliable as the number of keys increase. This is clearly mentioned in the docs:
    > If no command targets the key constantly, and there are many keys with a TTL associated, there can be a significant delay between the time the key time to live drops to zero, and the time the expired event is generated.

This worked fine for us in the beginning. But once the orders increased, a lot of clients started reporting really long waiting periods before they could see the status of their orders. On debugging, we saw delays of the tune of 40x and this was the deal breaker for us.

---

### In-memory Timers
As a quick fix to this we used in-process timers. Timers are defined and executed by the application code itself and can be used to manage delays. Node.js provides a variety of in-built [timer functions][nodejs-timer-doc]. Setting a 10 second timer is as simple as:
```js
setTimeout(() => {
    console.log('This executes after 10 seconds');
}, 10 * 1000);
```
#### Pros:
1. **Arbitrary delays.**
    Delay time is controlled by the time argument only. Passing the required delay time in milliseconds is all that is required.
2. **Reliability.**
    Timers in code are quite reliable. Deviation of only upto few milliseconds is observed.

#### Cons:
1. **Stateful application code.**
    The application adds to its state whenever a timer is set up. It also increases memory footprint of the application.
2. **Unscalable.**
    In-memory timers do not work well with horizontal scaling. Timer will always trigger the instance which set it in the first place, irrespective of the traffic distribution across the instances.

Timers worked well to mask the problem until we had bandwidth to think about the problem and fix it for good. Now we needed a permanent and reliable solution that would scale.

---

### Dead Letter Queues
After researching a bit about this topic, we decided to try Dead letter queues. DLQ is a sophisticated but robust way to delay message delivery used in message queues like Amazon SQS, RabbitMQ and ActiveMQ.

In very basic terms, a message with an expiry is put into a queue. The queue is instructed to take some action if the messages are expired without being read. We can instruct these expired messages to be pushed to a different queue which is actively consumed by the target application and it simulates sdelayed delivery of message. Following diagram explains this concept:

!['DLQ diagram'][dlq-image]

We chose RabbitMQ based on AMQP 0-9-1 for our implementation due to maturity of the protocol and its community.

#### Pros:
1. **Stateless application code.**
    The application doesn't need to store any state. The messages in the queue represent staete in this case.
2. **Scalable.**
    Unlike redis events, message delivery in MQs can be configured to use [worker pattern][rabbitmq-worker-doc] which loadbalances the consumers (subscribing instances of the application)  to deliver the message.

    !['RabbitMQ worker pattern'][rabbitmq-worker-image]

3. **Reliability.**
    Message Queues are expected to be highly reliable in delivery of messages. In our testing with RabbitMQ, the results were found to be close to in-memory timers.

#### Cons:
2. **Fixed delays.**
    In RabbitMQ's implementation of DLQs, the ttl is bound to the queue and applies to all the incoming messages. There is a way to set message level ttl but it has its own [caveats][rabbitmq-msg-ttl-ref]. One of the most serious side-effects of using message level ttl is that a message with high ttl will block the queue and delay processing of messages with low ttl. This leads to uncertainity in processing of message time and thus, doesn't solve the original problem.

We traded flexibility with robustness and reliability by using DLQs to manage the delays in our polling for order statuses. If you've solved a related problem in any other way, do let us know in comments.


[redis-setex-doc]: https://redis.io/commands/setex
[redis-notifications-doc]: https://redis.io/topics/notifications
[pub-sub-ref]: https://redis.io/topics/pubsub
[nodejs-timer-doc]:https://nodejs.org/en/docs/guides/timers-in-node/
[rabbitmq-worker-doc]: https://www.rabbitmq.com/tutorials/tutorial-two-javascript.html
[rabbitmq-msg-ttl-ref]: https://www.rabbitmq.com/ttl.html#per-message-ttl-caveats

[loading-gif]: https://media.giphy.com/media/OQHGckzXXDHZbuH1cN/giphy.gif
[dlq-image]: /data/images/dlqs.png
[rabbitmq-worker-image]: https://www.rabbitmq.com/img/tutorials/python-two.png