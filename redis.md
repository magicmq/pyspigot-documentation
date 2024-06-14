# Redis Manager

PySpigot includes a Redis manager to allow you to connect to and interact with redis servers. Under the hood, PySpigot utilizes [lettuce](https://lettuce.io/) for interfacing with remote Redis server instances. There are two ways the RedisManager allows you to interact with redis servers: by issuing commands (I.E. interacting with the redis database) and by publishing and subscribing to redis' [pub/sub messaging system](https://redis.io/docs/latest/develop/interact/pubsub/). These are implemented in PySpigot via the RedisCommandClient and RedisPubSubClient, respectively. PySpigot also includes a generic ScriptRedisClient that you may create if you would like to work with other aspects of Redis such as Redis Sentinel or Redis Cluster.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the redis manager into your script.

This is not a comprehensive guide to working with SQL or MongoDB. Please seek out the appropriate tutorials/information if you're unsure how to use SQL or MongoDB.

## Using the Redis Manager

