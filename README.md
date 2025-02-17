# coredis


[![docs](https://readthedocs.org/projects/coredis/badge/?version=stable)](https://coredis.readthedocs.org)
[![codecov](https://codecov.io/gh/alisaifee/coredis/branch/master/graph/badge.svg)](https://codecov.io/gh/alisaifee/coredis)
[![Latest Version in PyPI](https://img.shields.io/pypi/v/coredis.svg)](https://pypi.python.org/pypi/coredis/)
[![ci](https://github.com/alisaifee/coredis/workflows/CI/badge.svg?branch=master)](https://github.com/alisaifee/coredis/actions?query=branch%3Amaster+workflow%3ACI)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/coredis.svg)](https://pypi.python.org/pypi/coredis/)

---

coredis is an async redis client with support for redis server, cluster & sentinel.

- The client API uses the specifications in the [Redis command documentation](https://redis.io/commands/) to define the API by using the following conventions:

    - Arguments retain naming from redis as much as possible
    - Only optional variadic arguments are mapped to variadic positional or keyword arguments.
      When the variable length arguments are not optional (which is almost always the case) the expected argument 
      is an iterable of type [Parameters](https://coredis.readthedocs.io/en/latest/api.html#coredis.typing.Parameters) or `Mapping`. 
    - Pure tokens used as flags are mapped to boolean arguments
    - `One of` arguments accepting pure tokens are collapsed and accept a [PureToken](https://coredis.readthedocs.io/en/latest/api.html#coredis.tokens.PureToken)

- Responses are mapped as closely from RESP <-> python types as possible.

- For higher level concepts such as Pipelines, LUA Scripts, PubSub & Streams
  abstractions are provided to simplify interaction requires pre-defined sequencing of redis commands (see [Command Wrappers](https://coredis.readthedocs.io/en/latest/api.html#command-wrappers))
  and the [Handbook](https://coredis.readthedocs.io/en/latest/handbook/index.html).

> **Warning**
> The command API does NOT mirror the official python [redis client](https://github.com/redis/redis-py). For details about the high level differences refer to [Divergence from aredis & redis-py](https://coredis.readthedocs.io/en/latest/history.html#divergence-from-aredis-redis-py)
---

<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Installation](#installation)
- [Feature Summary](#feature-summary)
	- [Deployment topologies](#deployment-topologies)
	- [Application patterns](#application-patterns)
	- [Server side scripting](#server-side-scripting)
	- [Miscellaneous](#miscellaneous)
- [Quick start](#quick-start)
	- [Single Node client](#single-node-client)
	- [Cluster client](#cluster-client)
- [Compatibility](#compatibility)
	- [Supported python versions](#supported-python-versions)
	- [Redis-like Backends](#redis-like-backends)
- [References](#references)

<!-- /TOC -->

## Installation

To install coredis:

```bash
$ pip install coredis
```

## Feature Summary

### Deployment topologies
- [Redis Cluster](https://coredis.readthedocs.org/en/latest/handbook/cluster.html#redis-cluster)
- [Sentinel](https://coredis.readthedocs.org/en/latest/api.html#sentinel)

### Application patterns
- [Connection Pooling](https://coredis.readthedocs.org/en/latest/handbook/connections.html#connection-pools)
- [PubSub](https://coredis.readthedocs.org/en/latest/handbook/pubsub.html)
- [Sharded PubSub](https://coredis.readthedocs.org/en/latest/handbook/pubsub.html#sharded-pub-sub) [`>= Redis 7.0`]
- [Stream Consumers](https://coredis.readthedocs.org/en/latest/handbook/streams.html)
- [Pipelining](https://coredis.readthedocs.org/en/latest/handbook/pipelines.html)
- [Client side caching](https://coredis.readthedocs.org/en/latest/handbook/caching.html)

### Server side scripting
- [LUA Scripting](https://coredis.readthedocs.org/en/latest/handbook/scripting.html#lua_scripting)
- [Redis Libraries and functions](https://coredis.readthedocs.org/en/latest/handbook/scripting.html#library-functions) [`>= Redis 7.0`]

### Miscellaneous
- Public API annotated with type annotations
- Optional [Runtime Type Validation](https://coredis.readthedocs.org/en/latest/handbook/typing.html#runtime-type-checking) (via [beartype](https://github.com/beartype/beartype))

## Quick start

### Single Node or Cluster client

```python
import asyncio
from coredis import Redis, RedisCluster

async def example():
    client = Redis(host='127.0.0.1', port=6379, db=0)
    # or with redis cluster
    # client = RedisCluster(startup_nodes=[{"host": "127.0.01", "port": 7001}]) 
    await client.flushdb()
    await client.set('foo', 1)
    assert await client.exists(['foo']) == 1
    assert await client.incr('foo') == 2
    assert await client.incrby('foo', increment=100) == 102
    assert int(await client.get('foo')) == 102
    
    assert await client.expire('foo', 1)
    await asyncio.sleep(0.1)
    assert await client.ttl('foo') == 1
    assert await client.pttl('foo') < 1000
    await asyncio.sleep(1)
    assert not await client.exists(['foo'])

asyncio.run(example())
```

### Sentinel

```python
import asyncio
from coredis.sentinel import Sentinel

async def example():
    sentinel = Sentinel(sentinels=[("localhost", 26379)]) 
    primary = sentinel.primary_for("myservice")
    replica = sentinel.replica_for("myservice")
    
    assert await primary.set("fubar", 1)
    assert int(await replica.get("fubar")) == 1

asyncio.run(example())
```

To see a full list of supported redis commands refer to the [Command
compatibility](https://coredis.readthedocs.io/en/latest/compatibility.html)
documentation

## Compatibility

coredis is tested against redis versions `6.0.x`, `6.2.x` & `7.0.x`. The
test matrix status can be reviewed
[here](https://github.com/alisaifee/coredis/actions/workflows/main.yml)

coredis is additionally tested against:

-   ` uvloop >= 0.15.0`


### Supported python versions

-   3.7
-   3.8
-   3.9
-   3.10
-   3.11
-   PyPy 3.7
-   PyPy 3.8
-   PyPy 3.9


### Redis-like backends

**coredis** is known to work with the following databases that have redis protocol compatibility:

- [KeyDB](https://docs.keydb.dev/)
- [Dragonfly](https://dragonflydb.io/)

## References

- [Documentation (Stable)](http://coredis.readthedocs.org/en/stable)
- [Documentation (Latest)](http://coredis.readthedocs.org/en/latest)
- [Changelog](http://coredis.readthedocs.org/en/stable/release_notes.html)
- [Project History](http://coredis.readthedocs.org/en/stable/history.html)


[redis-py]: https://github.com/redis/redis-py "Official redis python client"
[aredis]: https://github.com/NoneGG/aredis "aredis"
