
# How to choose between RabbitMQ and Kafka

RabbitMQ and Kafka are two terms which are likely to be thrown around together a lot in an engineering meeting. When I felt the need to use a queueing system and looked for recommendations, it came down to choose from one of these two. Here's my take on

[redis-setex-doc]: https://redis.io/commands/setex
[redis-notifications-doc]: https://redis.io/topics/notifications
[pub-sub-ref]: https://redis.io/topics/pubsub
[nodejs-timer-doc]:https://nodejs.org/en/docs/guides/timers-in-node/
[rabbitmq-worker-doc]: https://www.rabbitmq.com/tutorials/tutorial-two-javascript.html
[rabbitmq-msg-ttl-ref]: https://www.rabbitmq.com/ttl.html#per-message-ttl-caveats

[loading-gif]: https://media.giphy.com/media/OQHGckzXXDHZbuH1cN/giphy.gif
[dlq-image]: /data/images/dlqs.png
[rabbitmq-worker-image]: https://www.rabbitmq.com/img/tutorials/python-two.png