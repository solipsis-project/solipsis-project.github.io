---
layout: post
title:  "Mizu Network Protocol, a Primer - How to efficiently pull updates from the Mizu network by using Prolly Trees"
categories: [Mizu]
excerpt: "How can two clients communicating over a network efficiently determine which bits of data they need from each other? See how Mizu solves this problem."
---

([Mizu](https://mizu.stream) is my attempt to build a peer-to-peer network protocol for decentralized applications. It is **not** a blockchain. This blog provides insight into the design process.)

Mizu can be thought of as having three main components, working in tandem:
- A local database, which is a collection of content-addressed records which contain metadata on how the content can be rendered to the user and interacted with.
- User interface tools for interacting with the database via generated web content and interactive elements that create additional transactions, which add one or more records to the database.
- A network layer that allows nodes to request and receive records from other nodes, allowing each database instance to serve as a part of a distributed hash table of records.

Networking efficiently can be tricky: when a requesting node is receiving data from a providing node, both nodes would prefer that the provider not resend any data that the requester already has in its database. Ideally, the amount of data transmitted should scale with the *difference* between the two databases, not with the size of the databases. Synchronizing two large databases should still be fast if the differences between them are minimal. 

In order to accomplish this, the nodes must be able to identify which parts of the database are already the same between both nodes, and which parts differ, without either machine having access to the other’s database.

At first glance, this seems impossible. How can the providing node determine whether or not the requesting node already has a record unless one sends the other its complete list of records?

Fortunately, there’s a powerful tool that’s perfectly situated for the task: Merkle trees.

# Merkle Trees

Even if you’ve never heard of them before, you’ve probably used them. Git is built on Merkle trees. BitTorrent magnet links can be considered Merkle trees.

The idea behind Merkle trees is simple, and can be done whenever you have hierarchical data in a tree-like structure. Creating a Merkle tree from your data has three steps:
1. Each node in your tree is assigned a cryptographic hash of its contents, such that two nodes have the same hash if and only if their contents (their data, their children, their children’s children, etc) are equal.
2. Each node is stored in a hash table, with its content hash as the hash table key.
3. Instead of each parent node holding a pointer to its children, each parent node stores the content hash of its children. Obtaining the children from the parent thus requires a lookup into a hash table.

A Mizu node creates a Merkle tree for each topic that it may need to query the network about. This tree serves as an index and contains every database record that is related to that topic. (For example, a topic might be all blog posts by a given author.) Then, when a requesting node asks for updates from a providing note, it becomes trivial to detect if both nodes already have the exact same set of records: the nodes can share the content hash for the root of the tree: if they match, then both nodes are up to date with each other.

But what if they don’t match? There’s another possible case: that even though the root hashes don’t match, one node already contains the other node’s root hash in their hash table. This would mean that one node’s index is a superset of the other’s, and that note can easily compute each record that the other node is missing.

In all other cases, the only thing that the nodes learn is that their set of records are not exactly the same. The smaller of the two sets is obviously missing records from the larger, but is the reverse true as well? Simply looking at the root hash can’t tell us that. The nodes will have to share more information, including the content hashes of the root node’s children, and potentially the children’s children, and so on recursively, stopping only when they encounter a familiar content hash (or the leaves of the graph.) By doing this, only the parts of the graph that have changed are visited.

This is a rough description of how Git pulls work, and how Mizu requests work. But the devil is in the details, because not all Merkle trees are created equal. As the depth of the tree grows, the number of round trips / hash table lookups required to read the whole thing increases. On the flip side, you could have an extremely shallow Merkle tree with only a single root and leaves: but if you’re comparing two such trees and the roots don’t match, you now have no other option but to compare every leaf in turn, losing all the benefits of using a Merkle tree.

There’s other considerations as well: you get much better performance comparing two Merkle trees if all of the differences are isolated in one part of the tree. In the worst case where the differences are uniformly spread out among the leaves, the time to compare scales linearly with the size of the trees.

And there’s one final pernicious thorn that we have to contend with. In Mizu, not every node is going to receive every database record in the same order. If we’re not careful with how these records are processed and stored, then two nodes who contain the same set of records may end up with a different tree structure, which will have a different content hash. Comparing two databases like this could end up with the worst case performance every time, even when the databases contain the exact same records and there’s no actual work to be done. We need “history independence.” That is, two databases which contain the same set of records must produce identical trees with identical content hashes, regardless of the order in which those records were added. This isn’t hard to do (you could always just sort all the records lexicographically)... but it is hard to do while also keeping the database *usable* (in a format where the user can query it), and it’s also hard to do while minimizing how frequently we invalidate and recalculate content hashes.

Clearly, just using Merkle trees is not a complete solution. The way that data is structured in the Merkle tree matters hugely. There are a lot of different constraints, and if we had to design a solution from scratch, we’d be in trouble. Fortunately, there’s a specific type of Merkle tree that solves nearly all these problems. It’s history-independent, so all trees storing the same data will have the same structure (and the same root hash). It’s a drop-in replacement for index trees used in actual databases. Insertions into the tree only invalidate a number of content hashes approximately equal to the tree depth. The average number of children per-node is configurable, and it’s self-balancing.

Prolly trees come to save the day. Without them, Mizu would be horrifically inefficient.

# Prolly Trees

Prolly trees were invented by the [Noms project](https://github.com/attic-labs/noms) to achieve a very similar goal to Mizu: a decentralized, syncable, queryable database. Although noms is no longer under active development, it lives on through [Dolt](https://docs.dolthub.com/), a version-controlled SQL database with git-like commits and push-and-pull semantics.

The exact mechanisms by which Prolly trees achieve these properties is a bit beyond the scope of this writeup. It’s also been covered in much more detail (and with diagrams!) by both of these prior projects. The simple summary is that Prolly trees are Merkle trees that can store a set of key-value pairs (note: not to be confused with how the content hashes themselves are used as keys in a hash table), and they work by sorting the keys and then using a [rolling hash](https://en.wikipedia.org/wiki/Rolling_hash) on the key-value pairs in order to deterministically place boundaries that dictate the structure of the resulting Merkle tree. If you’re interested in a deeper dive, I recommend the [Dolt docs](https://docs.dolthub.com/architecture/storage-engine/prolly-tree) on Prolly trees, or the [Noms intro file.](https://github.com/attic-labs/noms/blob/master/doc/intro.md#prolly-trees-probabilistic-b-trees)

The important thing is that now, for a given queryable topic, a Mizu node can create a Prolly tree which contains a content hash of each database record related to that topic. This allows two nodes communicating over the network to efficiently compute the difference between their databases and avoid sending records that the recipient already has. 

# Imposing a Total Ordering on Records

An astute reader might notice that I said earlier that Prolly trees solve *nearly* all of the problems faced by Mizu. In particular, simply using Prolly trees won’t save us from the performance problem that occurs when diffs are uniformly distributed through the keyspace (we really want new keys to be clustered to one part of the tree.) However, I also never specified that the keys used in the Prolly trees have to be the same keys used in our database. It’s nice if they are, because it means that we could query the tree like we query the database. But if our main motivation for using Prolly trees is purely to assign a content hash to a set of objects and allow for fast pulling of updates to this set… then the tree keys can be whatever we want them to be. We just need to pick a key where newer records are more likely to be compared greater than older records: this will ensure that newer records bunch up on the right side of the tree, while the left side of the tree is more “set in stone.”

(I say key here, but given that we’re using the prolly tree to implement a set, not a map, the records being stored are themselves the keys. So when I talk about choosing a key, what I’m really choosing is “a value to be prepended to the record, to influence their sorting order.” This value can be the same for multiple records, in which case those records will be sorted according to their natural lexicographic order. It removes the ability to look up whether a specific record is in the tree unless we know the extra value that we prepended, but for our purposes, that’s okay.)

This part of the protocol isn’t set in stone yet, but I have two ideas.

## Idea 1: Timestamps

Mizu does not rely on timestamps, because reasoning about timestamps has to account for the fact that not every client is synchronized to the same clock, and since timestamps on messages are easily forged, there hasn’t been any real benefit to using them before now anyway. But, assuming that nodes attach a timestamp to their transactions and these timestamps are honest, it becomes an easy way to sort records so that newly added records are all at the edge of the Prolly tree. Even if nodes use incorrect timestamps, the worst outcome is that queries that contain these records will have slightly worse performance, and this bad behavior can easily be detected. Nodes could attempt to prevent records with bad timestamps from propagating through the network by rejecting records whose timestamps are outside of an expected range. (Timestamps in the future are obviously wrong, and a node may assume that it already has all records that date from well before the last time the query was run.)

A similar option is to use a non-decreasing nonce as the key: a user node tracks the highest-valued key they’ve processed, and when they publish a new transaction, they use a different nonce value for each new record, each of which is higher than the highest previously noted value. This value doesn’t need to correspond to any real-world timestamp, provided that it’s higher than all values observed in the past.

## Idea 2: Topological Order of Transactions

What if, when a client publishes a transaction to the Mizu network, we capture the current state of the application, and use *that* (or a content hash of that state) as the key? Suppose that we take the hash at the head of one or more prolly trees (and if we used more than one content hash, making an object out of those hashes and then hashing *that*) and that becomes our key? In this case, the ordering function isn’t a simple lexicographic ordering of these keys, but a [topological ordering](https://en.wikipedia.org/wiki/Topological_sorting) of the keys, such that if content hash A points to an object which contains another content hash B (directly or transitively), then B comes before A in the ordering.

The hash table itself produces a partial ordering of all the content hashes: topological sorting would extend that partial order into a total order. However, care must be taken so that this ordering is stable: if content hashes C and D are not ordered in the partial ordering, then this extension should assign to them the same total ordering regardless of what other hashes are in the hash table.

If successful, this approach wouldn’t rely on users to play nice the way that option 1 does; normal Mizu use would automatically create a total ordering on transactions. But it’s not without its difficulties: the ordering algorithm is necessarily more complicated than a simple sorting of timestamps. It also requires publishers to divulge which records have already been processed by their nodes, which could be, *in theory* a privacy issue, depending on the application.

# Conclusion

No project is an island. Mizu is built extensively on research done by those who have come before. It uses the [json-rql query language](https://json-rql.org/), created by [m-ld](https://m-ld.org/) to run queries on schemaless data. It uses [libp2p](https://libp2p.io/) to connect its clients and [ipld](https://ipld.io/) as its data model, both of which were created by [Protocol Labs](https://protocol.ai/). And now, it uses prolly trees, which are a source of ongoing research and development at [Dolthub](https://dolthub.com/).

To the companies who develop and maintain each of these projects: Mizu only exists because of you. It still has a long way to go, but it’s built on a firm foundation. I’m excited to see where this journey leads, and I hope you are too.

(Also, if you work for any of these companies, and you like what you see and think I would be an asset to your team and want to hire me, shoot me a line at the.solipsis.project@gmail.com!)
