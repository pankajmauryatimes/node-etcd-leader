etcd-leader
===========

[![Build Status][badge-travis-img]][badge-travis-url]
[![Dependency Information][badge-david-img]][badge-david-url]
[![Code Coverage][badge-cov-img]][badge-cov-url]
[![Code Quality][badge-climate-img]][badge-climate-url]

**Under development, not yet suitable for production use.**

Leader election module built on top of [node-etcd](https://github.com/stianeikeland/node-etcd).

`npm install etcd-leader`

## Usage

```
  var Etcd = require("node-etcd");
  var etcdLeader = require("etcd-leader");

  var etcd = new Etcd("localhost", 4001);

  // First parameter is etcd key to use for election.
  // Second parameter is name of this node.
  // Third parameter is the expiry window for master election.
  var election = etcdLeader(etcd, "/master", "foo", 10).start();

  election.on("elected", function() {
    console.log("I am the MASTER.");
  });

  election.on("unelected", function() {
    console.log("I am no longer the MASTER.");
  });

  election.on("newMaster", function(node) {
    console.log("Master is now " + node);
  });
```

## Algorithm

Important question. Do you trust me to elect your masters?

The leader election algorithm here is based on top of the in-order value strategy [documented by ZooKeeper](https://zookeeper.apache.org/doc/trunk/recipes.html#sc_leaderElection).

How do we do this specifically in etcd? Let's assume we have initialised etcd-leader to use `/master` as the master key, and we have three nodes, with node names `foo`, `bar` and `quux` respectively.

 * `foo` starts up first, it issues a `POST` to `/master` (with a TTL of 10 seconds). It gets a createdIndex of "5".
 * `foo` begins refreshing its value every 5 seconds.
 * `foo` enumerates `/master` to find lowest sorted createdIndex node. Discovers that it's itself.
 * `foo` is now master.
 * `bar` starts up next, it also issues a `POST` to `/master`. It gets a createdIndex of "7".
 * `bar` begins refreshing its value every 5 seconds.
 * `bar` enumerates `/master`, sees that `foo` is the lowest createdIndex. Starts watching that node, waiting for it to disappear.
 * `quux` starts up, issues the POST and gets a createdIndex of "9".
 * `quux` begins refreshing its value every 5 seconds.
 * `quux` enumerates `/master`, sees that `foo` is the lowest createdIndex, and that `bar` is the node that immediately preceeds it.
 * `quux` starts watching `bar`'s node for changes, waiting for it to disappear.

[badge-david-img]: http://img.shields.io/david/samcday/node-etcd-leader.svg?style=flat-square
[badge-david-url]: https://david-dm.org/samcday/node-etcd-leader
[badge-travis-img]: http://img.shields.io/travis/samcday/node-etcd-leader.svg?style=flat-square
[badge-travis-url]: https://travis-ci.org/samcday/node-etcd-leader
[badge-climate-img]: http://img.shields.io/codeclimate/github/samcday/node-etcd-leader.svg?style=flat-square
[badge-climate-url]: https://codeclimate.com/github/samcday/node-etcd-leader
[badge-cov-img]: http://img.shields.io/codeclimate/coverage/github/samcday/node-etcd-leader.svg?style=flat-square
[badge-cov-url]: https://codeclimate.com/github/samcday/node-etcd-leader
