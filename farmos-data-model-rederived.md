# Rederive the farmOS Data Model
These notes are very, _very_ rough, and probably exceed my own knowledge of
category theory and proper database design, so please, don't judge or mistake my
explorations for expertise. It's just how I currently think that the ideal of a
[schema-first design] can best be realized.

The whole concept of rederiving the farmOS Data Model from first principles
stems from the pain points I experienced with [farmOS Field Kit], trying to
anticipate the different kinds of schema variations that might occur on
different server configurations. Even when JSON Schema was finally provided by
the server, it often contained inconsistencies and erroneous definitions, as
well as many extraneous fields that were only pertinent to Drupal's own
internals and modules. The biggest issues were ameliorated with a few bugfixes
and some considerable workarounds in farmOS.js – for instance, the [`adapter`]
module that tried to isolate these "Drupalisms" as much as possible so they did
not bleed into the JavaScript implementation for the model or client. Still, it
struck me it would remain prone to continuous issues of that sort so long as the
JSON Schema and was derived from the Drupal implementation, rather than the
other way around: a pure schema representation that could form the basis of the
Drupal service and also be serialized in its original form for ingestion by
external clients and peers.

It's clear to me, however, that this may likely require a more fine-grained set
of data primitives than Drupal's Entity-Relationship Model that comprises the
main backbone of the farmOS Data Model as it stands today. However, I believe
compatibility with Standard farmOS can still be preserved if approached with a
mind to compose those entities from smaller parts. The Entity-Relationship Model
is a sound base to start from, and it's further improved by the many borrowings
from "double-entry ledger" that I know underpins the design of farmOS logs.
Functional programming is well regarded for its emphasis on composition, perhaps
above all other features, so it seems the natural direction to take when aiming
to rederive farmOS entities by pure composition.

[schema-first design]: ./README.md#schema-first-design
[farmOS Field Kit]: https://github.com/farmOS/field-kit
[`adapter`]: https://github.com/farmOS/farmOS.js/blob/2.0.0-beta.16/src/client/adapter/index.js

## Entities as Functions
What if logs and assets are modelled as _pure functions_ or [morphisms] or
[Algebraic Data Types (ADTs)]?

What is the [Identity] function of Log? of Asset?

Also morphisms or ADTs:
- [Concat], i.e.: `Semigroup`
- [Equals], i.e.: `Setoid`
- [Empty], i.e.: `Monoid`
- [Map], i.e.: `Functor`
- [`Apply`]
- [Of], i.e.: `Applicative`
- [Chain], i.e.: `Monad`
- [List] or [Cons], i.e.: `( cons car cdr )`
- etc...

[morphisms]:
    https://mostly-adequate.gitbook.io/mostly-adequate-guide/ch05#category-theory
[Algebraic Data Types (ADTs)]:
    https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/
[Identity]:
    https://mostly-adequate.gitbook.io/mostly-adequate-guide/appendix_b#identity
[Concat]: https://beef.cat/en/2017/03/13/fantas-eel-and-specification-4
[Equals]: https://beef.cat/en/2017/03/09/fantas-eel-and-specification-3
[Empty]: https://beef.cat/en/2017/03/21/fantas-eel-and-specification-5
[Map]: https://beef.cat/en/2017/03/27/fantas-eel-and-specification-6
[`Apply`]: https://beef.cat/en/2017/04/10/fantas-eel-and-specification-8
[Of]: https://beef.cat/en/2017/04/17/fantas-eel-and-specification-9
[Chain]: https://beef.cat/en/2017/05/15/fantas-eel-and-specification-13
[List]: https://beef.cat/en/2017/03/03/fantas-eel-and-specification
[Cons]: https://en.wikipedia.org/wiki/Cons

### Entity Attributes & Relationships as Functions
How to compose the Log function up from smaller functions that provide
attributes, etc?

How are attribute functions the same and/or different from relationship
functions?

Maybe it's not so critical for attributes like `"name"`, `"flags"` and `"notes"`
be modeled as functions; they can just be values or metadata. But then to bundle
those values together and associate them with a reference to a log, they almost
become their own type of entity function, like a Metadata or Comment entity, or
even a distinct Attribute entity unto itself.

Attributes are probably the most straight forward to model as functions, in some
ways, because their parameters are just plain values – e.g., integers, strings,
or composite values, but no references.

Relationships get a little more complicated to model as functions because their
parameters and outputs are references to other entities, which implies various
effects such as database read/write operations and access control. However, if
those effects are deferred, then relationship functions only need to consider
its parameters and outputs as UUIDs or some other form of identifier and then
they become the simplest form of set operations.

Where things get a bit trickier are where computed fields come into play. These
can be either attributes or relationships, but in both cases they will require
an indefinite number of references to other entities and their computed or
concrete values, which in turn reference other entities and so on. But in the
end, this is not a new problem of the functional implementation per se; it's an
aspect already represented as the farmOS Data Model's [Logic Components]. The
benefit of the functional approach, however, is it provides a ready-made model
for how to formalize logic straight out of the gate: the function itself. This
can be ameliorated further by modeling [assets as functors of logs], or as a
sort of continuation of logs, as detailed below.

[Logic Components]: https://farmos.org/model/#logic
[assets as functors of logs]: #functor-of-log-asset

### Bundle Type
Can the log's entity type and bundle type be derived essentially from the
specific combination of the attribute and relationship function types?

If the entity type defines certain capabilities of the log to transform assets
or derive computed properties, how can that capability be represented as a
function, while also being a composition of attr/rel function types?

Bear in mind that the final Log type resulting from all of that composition
should itself be a function that takes one or more Asset types as parameters and
returns some combination of assets 

### Functor of Log (Asset?)
It may also be useful to have some Functor of Log, for an ordered sequence of
events that may relate to one location, asset, or some criterion for a
collection of entities. Or maybe something akin to the [Iterator/Iterable
protocol], with its `.next()` / `.done()` methods and `.value` property.

Or maybe that "Functor of Log" _is the Asset Type itself_, as a series of Logs
where only the identity (UUID) of the Asset is constant.

For that to work, the asset's attributes and relationships must be given to it
through the application of some log function from the start, when it is first
instantiated, and thereafter for every update or change to those attributes and
relationships, with their proper sequence derived from the logs' timestamps.
That would once again insinuate that each attribute and relationship should also
be a type of function.

[Iterator/Iterable
    protocol]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols

#### Computed Values
The "Asset as a Functor of Log" concept would almost automatically yield the
kind of "computed" attributes like `"geometry"` and `"inventory"` without any
other abstractions. This eliminates any need to persist them as distinct,
mutable values in any database, though they can still be indexed rather
trivially.

What makes this especially helpful here is that the computed attribute offers a
very narrow, well-defined set of criteria for querying a database or API
efficiently, assuming the ID of the asset is already in hand...

<!-- TODO: how so? -->


### Timeline of an Asset
An important characteristic of assets in the farmOS Data Model is that their
creation does not necessarily correspond to the creation or production of the
asset itself in the physical world.

This is important because it makes it easier to onboard and start using farmOS
w/o the need to reproduce the entire history of every asset, but it's also
problematic. It implies these eternal entity objects that have to be persisted
with their entire history meticulously ordered and updated in place, but without
any clearly defined frame of reference for that historical duration of time.

By adopting [Rich Hickey's process-oriented, functional programming approach],
we can sidestep the whole time problem while holding onto our version history at
no extra cost. This is because the _asset entity as a function_ is now just an
_identifier_, signifying a causally ordered time series of value-states – that
is, the series of _logs with an established relationship to the identifier_, the
UUID we just so happen to call "the asset."

<picture>
  <figure>
    <img src="https://raw.githubusercontent.com/matthiasn/talk-transcripts/d644becd0f4eebb3a165a63b3bdf1e8d6b881d33/Hickey_Rich/AreWeThereYet/00.22.34.jpg"/>
  </figure>
  <figcaption>
    <em>Fig. 1:</em> A slide from Rich Hickey's talk, <a href="http://www.infoq.com/presentations/Are-We-There-Yet-Rich-Hickey">"Are We There Yet?"</a>, in reference to Alfred North Whitehead's <a href="https://plato.stanford.edu/entries/process-philosophy/">process philosophy</a>. For a high-level overview of the talk and its concepts, see Daniel Higginbotham's <a href="https://www.flyingmachinestudios.com/programming/the-unofficial-guide-to-rich-hickeys-brain/">"The Unofficial Guide to Rich Hickey's Brain"</a>. 
  </figcaption>
</picture>

[Rich Hickey's process-oriented, functional programming approach]:
    https://github.com/matthiasn/talk-transcripts/blob/master/Hickey_Rich/AreWeThereYet.md


### Timeline of a Log
On the flipside, however, this does require a stricter implementation of logs as
immutable data types. This is why we'll try to compose logs from smaller
constituent pieces (i.e., their attributes and relationships) wherever possible,
because those smaller chunks will be easier to keep immutable.

Apart from the `"timestamp"` attribute of the log itself (i.e., the time when
the actual event took place on the farm), the constituent attributes and
relationships of the log must have a timestamp, roughly equivalent to the Drupal
entity's `"created_at"` attribute.  In fact, the `"timestamp"` itself, as a
function attribute, will need a timestamp for each time its updated. This will
be more critical for some attributes than others, such as updates to the
`"status"` attribute, which can bear significantly on whether an event occurred
too early or too late etc.

Also consider: when a log's `"status"` attribute is updated to the `"done"`
state, it is decoupled from the log's `"timestamp"` attribute. Therefore, that
particular status update's own timestamp is not strictly equivalent to the log's
`"timestamp"` attribute; this is probably correct, but should be kept in mind
when implementing such log functionality and how it is presented to end users.

### Targeted Values
Similar to [Computed Values], it may be possible to derive targeted values
without any further abstractions; more precisely, it would allow for
differentiating targeted from actual values, by comparing the attribute's
timestamp's from the log's `"timestamp"` attribute itself. Unlike computed
values, however, this would necessitate persisting a distinct record in the
database for each target value.

[Computed Values]: #computed-values


## ER Model vs Merkle Trees and DHTs
What if instead of the Entity Relationship (ER) Model we used something like a
Name-Value Model or Identifier-Value Model, akin to what Rich Hickey describes
in his talk, ["Are We There Yet?"] and the [Hash Array Mapped Trie] (HAMT) that
he used in the [Clojure implementation] of [persistent data structures]? HAMT
was first described by Phil Bagwell in his highly regard 2000 paper, ["Ideal
Hash Trees"], which I still need to read.

According to Wikipedia, there's also this thing called a [Concurrent Hash Trie]
(or Ctrie, not to be confused w/ C-trie), which boasts:

> Ctries support a lock-free, linearizable, constant-time snapshot operation,
> based on the insight obtained from persistent data structures. This is a
> breakthrough in concurrent data-structure design, since existing concurrent
> data-structures do not support snapshots.

The "snapshot" aspect is appealing, given how much emphasis Hickey gives to the
metaphor in describing Whitehead's Process Philosophy. But is immutability what
<!-- FIXME: This was a reference to my data independence entry. -->
a distributed system like the ones I'm describing above would actually need or
want? All of these other implementations are meant for in-memory data structures
or for optimizing many write operations, things like language interpreters and
[Software Transactional Memory] (another concept Hickey mentions in his talk);
that's really not much like the problem I'm hoping to solve. So is there a way
to simplify that functionality w/o breaking it? What I should be aiming for is a
model tailored to low-power, distributed environment that can run across a small
regional network, preferably even more localized by mesh networks and P2P
connections, not a data center running trillions of operations per second on
petabytes of data.

A trie (or tree) of some sort seems to me more and more like the kind of
structure that is wanted here, and there's certainly precedent for structures
like [Merkle Trees] in distributed systems. In fact, given the ubiquity of
Merkle tree implementations – they find their way into a vast array of popular
applications including Git, BitTorrent, Nix, IPFS, Hypercore, etc. – it seems
like the best first approach to get up and running quickly, if a tree of some
sort is indeedsuitable, given the availability of mature libraries in multiple
languages, the accessibility of literature and learning resources. Another
closely related structure, though not exactly a tree, are [Distributed Hash
Tables (DHTs)], which flatten the Merkle tree to a lookup table of common
resources that could be dumped into SQL and replicated more readily.


["Are We There Yet?"]:
    https://github.com/matthiasn/talk-transcripts/blob/master/Hickey_Rich/AreWeThereYet.md
[Hash Array Mapped Trie]: https://en.wikipedia.org/wiki/Hash_array_mapped_trie
[Clojure implementation]:
    https://github.com/clojure/clojure/blob/clojure-1.12.0/src/jvm/clojure/lang/PersistentHashMap.java
["Ideal Hash Trees"]: https://lampwww.epfl.ch/papers/idealhashtrees.pdf
[persistent data structures]:
    https://en.wikipedia.org/wiki/Persistent_data_structure
[Software Transactional Memory]:
    https://en.wikipedia.org/wiki/Software_transactional_memory
[Merkle Trees]: https://en.wikipedia.org/wiki/Merkle_tree
[Distributed Hash Tables (DHTs)]:
    https://en.wikipedia.org/wiki/Distributed_hash_table
[Concurrent Hash Trie]: https://en.wikipedia.org/wiki/Merkle_tree
