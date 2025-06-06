# Archived note (from Jeff)
This repository was originally created to facilitate tracing in sockets. A change was made that [negated the requirement](https://github.com/geneva/geneva/pull/4176) for this fork, so it was archived on 5/12/2025.

[![Go Reference](https://pkg.go.dev/badge/github.com/mna/redisc.svg)](https://pkg.go.dev/github.com/mna/redisc)
[![Build Status](https://github.com/mna/redisc/actions/workflows/test.yml/badge.svg?branch=master)](https://github.com/mna/redisc/actions)

# redisc

Package redisc implements a redis cluster client built on top of the [redigo package][redigo]. See the [documentation][godoc] for details.

## Installation

    $ go get [-u] [-t] github.com/mna/redisc

## Releases

* **v1.3.2** : Export the `HashSlots` constant to make it nicer to write the `Cluster.LayoutRefresh` function signature.

* **v1.3.1** : Fix closing/releasing of connections used in `Cluster.EachNode`.

* **v1.3.0** : Add `Cluster.EachNode` to call a function with a connection for each known node in the cluster (e.g. to run diagnostics commands on each node or to collect all keys in a cluster); add optional Cluster function field `BgError` to receive notification of errors happening in background topology refreshes and on closing of `RetryConn` after following a redirection to a new connection; add optional Cluster function field `LayoutRefresh` to receive the old and new cluster slot mappings to server address(es); prevent unnecessary cluster layout refreshes when the internal mapping is the same as the redirection error; better handling of closed Cluster; move CI to Github Actions; drop support for old Go versions (currently tested on 1.15+); enable more static analysis/linters; refactor tests to create less separate clusters and run faster.

* **v1.2.0** : Use Go modules, fix a failing test due to changed error message on Redis 6.

* **v1.1.7** : Do not bind to a random node if `Do` is called without a command and the connection is not already bound (thanks to [@tysonmote][tysonmote]).

* **v1.1.6** : Append the actual error messages when a refresh returns "all nodes failed" error.

* **v1.1.5** : Add `Cluster.PoolWaitTime` to configure the time to wait on a connection from a pool with `MaxActive` > 0 and `Wait` set to true (thanks to [@iwanbk][iwanbk]).

* **v1.1.4** : Add `Conn.DoWithTimeout` and `Conn.ReceiveWithTimeout` to match redigo's `ConnWithTimeout` interface (thanks to [@letsfire][letsfire]).

* **v1.1.3** : Fix handling of `ASK` replies in `RetryConn`.

* **v1.1.2** : Remove mention that `StartupNodes` in `Cluster` struct needs to be master nodes (it can be replicas). Add supporting test.

* **v1.1.1** : Fix CI tests.

* **v1.1.0** : This release builds with the `github.com/gomodule/redigo` package (the new import path of `redigo`, which also has a breaking change in its `v2.0.0`, the `PMessage` type has been removed and consolidated into `Message`).

* **v1.0.0** : This release builds with the `github.com/garyburd/redigo` package, which - according to its [readme][oldredigo] - will not be maintained anymore, having moved to [`github.com/gomodule/redigo`][redigo] for future development. As such, `redisc` will not be updated with the old redigo package, this version was created only to avoid causing issues to users of redisc.

## Documentation

The [code documentation][godoc] is the canonical source for documentation.

The design goal of redisc is to be as compatible as possible with the [redigo][] package. As such, the `Cluster` type can be used as a drop-in replacement to a `redis.Pool` when moving from a standalone Redis to a Redis Cluster setup, and the connections returned by the cluster implement redigo's `redis.Conn` interface. The package offers additional features specific to dealing with a cluster that may be needed for more advanced scenarios.

The main features are:

* Drop-in replacement for `redis.Pool` (the `Cluster` type implements the same `Get` and `Close` method signatures).
* Connections are `redis.Conn` interfaces and use the `redigo` package to execute commands, `redisc` only handles the cluster part.
* Support for all cluster-supported commands including scripting, transactions and pub-sub (within the limitations imposed by Redis Cluster).
* Support for READONLY/READWRITE commands to allow reading data from replicas.
* Client-side smart routing, automatically keeps track of which node holds which key slots.
* Automatic retry of MOVED, ASK and TRYAGAIN errors when desired, via `RetryConn`.
* Manual handling of redirections and retries when desired, via `IsTryAgain` and `ParseRedir`.
* Automatic detection of the node to call based on the command's first parameter (assumed to be the key).
* Explicit selection of the node to call via `BindConn` when needed.
* Support for optimal batch calls via `SplitBySlot`.

Note that to make efficient use of Redis Cluster, some upfront work is usually required. A good understanding of Redis Cluster is highly recommended and the official Redis website has [good documentation that covers this](https://redis.io/topics/cluster-spec). In particular, [Migrating to Redis Cluster](https://redis.io/topics/cluster-tutorial#migrating-to-redis-cluster) will help understand how straightforward (or not) the migration may be for your specific case.

## Alternatives

* [redis-go-cluster][rgc].
* [radix v1][radix1] provides a cluster package.
* [radix v2][radix2] provides a cluster package.

## Support

There are a number of ways you can support the project:

* Use it, star it, build something with it, spread the word!
* Raise issues to improve the project (note: doc typos and clarifications are issues too!)
  - Please search existing issues before opening a new one - it may have already been adressed.
* Pull requests: please discuss new code in an issue first, unless the fix is really trivial.
  - Make sure new code is tested.
  - Be mindful of existing code - PRs that break existing code have a high probability of being declined, unless it fixes a serious issue.
* Sponsor the developer
  - See the Github Sponsor button at the top of the repo on github

## License

The [BSD 3-Clause license][bsd].

[bsd]: http://opensource.org/licenses/BSD-3-Clause
[godoc]: https://pkg.go.dev/github.com/mna/redisc
[redigo]: https://github.com/gomodule/redigo
[oldredigo]: https://github.com/garyburd/redigo
[rgc]: https://github.com/chasex/redis-go-cluster
[radix1]: https://github.com/fzzy/radix
[radix2]: https://github.com/mediocregopher/radix.v2
[letsfire]: https://github.com/letsfire
[iwanbk]: https://github.com/iwanbk
[tysonmote]: https://github.com/tysonmote
