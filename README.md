# Runrig farmOS
_The following concept paper proposes an alternative implementation of farmOS,
purposely built for distributed computing scenarios and in accordance with
Runrig's design principles._

## Overview
_Runrig farmOS_ is a proposed suite of software packages for replicating and
interoperating with the original client-server implementation of farmOS, which
is written as a Drupal distribution in PHP. (To avoid confusion, we'll refer
here to the former as _Runrig farmOS_ or simply _Runrig_ and to the latter as
_Standard farmOS_ or _Standard_.) Runrig farmOS packages are intended to run on
servers, mobile devices, desktops, sensors, and embedded systems. They may be
deployed as a substitute for a Standard farmOS instance, as a supplement to an
existing one, or merely as one among many services bundled together as a Runrig
Autonomous Municipal Platform, or RAMP (more on this in a separate document).

Runrig farmOS is intended to enable local-first, peer-to-peer, and federated
architectures. The aim is to bring these and other features to Standard farmOS,
too, via Drupal modules that can be installed on any Standard farmOS instance.
This should allow users to exchange data and functionality freely between Runrig
and Standard and other free software services with ease. That way, as Runrig
farmOS matures and Standard farmOS continues to evolve, their combined feature
set can accommodate a wider range of requirements and use cases, complementing
each other's strengths and weaknesses rather than competing for users and
resources.

### Core Requirements
In pursuit of these aims, all packages in the Runrig farmOS suite must support
(or at least not prohibit) the following core requirements, wherever applicable:

1. __`libfarmOS`, a library implementation of the farmOS Data Model__, written
  in Rust or C/C++ and compiled to binary for optimal portability &
  performance; it should be capable of serving as a sort of driver underlying
  servers, mobile devices, IoT sensors, even embedded systems, without
  reference to higher level abstractions, such as the network, storage, or
  presentation layers.
2. __Multi-farm data representations and behavior__, either as the proposed
  [farmOS Organization entity] or other compatible extension of the farmOS Data
  Model, in order to support multi-tenancy and easier aggregation.
3. __A shared, deterministic consensus algorithm__, providing eventual
  consistency and predictable conflict resolution across concurrent devices and
  storage instances, but again, without reference to any specific network or
  storage protocols.
4. __Interoperation with Standard farmOS (v 3.0 or higher)__, if not in the form
  of realtime networked communication then at least by maintaining compatibility
  with data migrated to or from Standard servers.

The hope is that federated, P2P, and local-first architectures will emerge as
convenient byproducts of these four requirements alone, so support for
distributed architectures is not listed here specifically; however, that is
still considered an essential feature in its own right, so the core requirements
may be appended or amended if it does not emerge automatically as expected.

[farmOS Organization entity]: https://github.com/farmOS/farmOS/pull/849

### Schema-first Design
Another critical aspect of these packages is that they should adhere to a
principle we might call _schema-first design_. As much as possible, all
libraries and applications should derive their logic and structure from schema
definitions, ontologies, RPCs, code generation, and various types of static
configuration files that can be serialized, transmitted, and interpreted at
runtime – _not the other way around_.

For instance, if a plugin introduces a new type of farmOS asset or log, rather
than implement the entity in code and generate a schema and other static
representations from that, instead the plugin should provide the necessary
schema and behavior rules in a purely static form first – e.g., as a JSON Schema
document, RDF ontology, or Protocol Buffer services – allowing the common engine
or middleware to generate code based on that schema; or if code generation or
RPC is impractical, at least infer common procedures like writing to the
database or generating a route for API requests.

This approach should allow for greater _portability_ across diverse languages
and runtime environments, but without limiting the _modularity_ of specific
applications or the system as a whole. That is the ultimate object of
schema-first design, because while portability and modularity may be two equally
desirable characteristics of a given software system, they can also be the most
difficult qualities to reconcile.

To sum up the main intent,

> __Schema-first Design &#x21D2; Portability + Modularity__

where schema-first design enables portability and modularity to coincide within
the same system, like an emulsifier encourages oil and water to mix.


## Core Packages & Shared Utilities
These libraries and utilities should perform the lion's share of processing,
storing, and transmitting farmOS-compliant data. Downstream packages in the
Runrig farmOS suite, such as the [Runrig Server] and [Runrig Client], can then
act as thin wrappers around the Common Engine, while applications further
downstream from the server and client themselves can elect to install additional
middlewares or adapters, providing whatever encodings or platform support the
application requires. Initially, these packages may be implemented in
TypeScript, as an early expedient that at least meets the criteria for
portability; eventually, however, most of packages would be ported to a compiled
language, ideally Rust, for greater portability and improved performance.

### Common Engine, or `libfarmos`
The Common Engine will be the primary locus of development of Runrig farmOS's
[Core Requirements], particularly implementation of the farmOS Data Model (Reqs.
1 & 2) and the consensus algorithm (Req. 3). Interoperation with Standard farmOS
(Req. 4), as well as other implementations, will mostly be the task of
[middlewares & adapters], though of course those utilities will rely heavily on
the Common Engine.

As a sort of standard library, it might make sense to adopt the naming
convention common to low-level system libraries, especially those written in
C/C++ and Rust, and package this engine under the name `libfarmOS` or something
similar; this is akin to a range of widely used system libraries like
[`libcurl`], [`libsodium`], [`libpng`], [`libsignal`], [`libVLC`], and
[`libsql`]. These examples variously serve a number of roles and functions:
dependencies for other system libraries and programs; drivers for higher-level
languages through foreign function interface (FFI), WebAssembly (WASM), or
running as an independent daemon; plug-n-play implementations of standards and
protocols that can be easily embedded into end-user applications to extend their
compatibility; or even, in the case of `libVLC`, a little bit of everything
above _plus_ a headless CLI utility for processing large binary datasets. A
library like `libfarmOS` could go a tremendous ways to encouraging adoption of
the farmOS Data Model into existing projects, even if just as an afterthought.
Support for other platforms and runtimes could be provided more readily as
language bindings, similar to the roles currently fulfilled by the [farmOS.py]
and [farmOS.js] client libraries, but with far greater capabilities and far less
effort to maintain. In essence, they would function as fully fledged
implementations of the data model, which could then be coupled with the
[middlewares & adapters] or third-party networking utilities and storage drivers
to perform the full range of backend capabilities, as performed by Standard
farmOS servers today.

The Core Logic in particular will require special care to implement without
violating the principle of [schema-first design]. A degree of schema enforcement
might be achieved with remote procedure calls (RPC), but that won't necessarily
enforce the particulars of a given algorithm; mainly RPC would just guarantee a
consistent type signature for each procedure – e.g., `getGeometry :: Asset ->
Geometry`. Some amount of code-sharing will likely prove unavoidable in the
first implementations of Runrig farmOS. Thankfully, the occasions of specific
logic in the farmOS Data Model are fairly limited, at the time of this draft,
currently just the three procedures listed in the requirements below. Changes
are unlikely to be introduced to Standard between major version releases. Also,
the three current logic procedures are standard to all relevant entities,
regardless of the bundle, so there appears to be no risk that farmOS modules
could add new bundles that would possibly alter that logic. For now then, the
logic will be implemented mostly in code, but the portability of the core
packages should suffice to ensure consistent behavior across systems.

As necessity dictates, greater algorithmic consistency might be achieved with
the adoption of more rigorous type systems, such as [Algebraic Data Types]
(ADTs). Though they don't add much in the way of schema enforcement, semantic
ontologies like RDF may offer greater transparency of system configurations and
capabilities. Even more comprehensive, in-band specification and enforcement
mechanisms might be possible via more innovative solutions, such as [Algebraic
Property Graphs] (essentially a marriage of IDL/RPC and RDF but reinforced by
using ADTs as its core primitives) or the distributed protocols in development
by the [Spritely Institute], like [OCapN], which expands upon Mark S. Miller's
[CapTP] with a particular eye towards improving federated protocols like
ActivityPub. As much as possible, OCapN aims to be backwards compatible with
certain existing implementations of CapTP, like Cap'n Proto, which could make
[Cap'n Proto's RPC protocol] and its [schema language] an especially attractive
target for early implementations. Of course, we should remain careful not to
impose specific transport protocols or communication models on the Common Engine
itself – the actual implementation of Cap'n Proto or any RPC framework should be
the purview of [middlewares & adapters] below. The Common Engine, meanwhile,
should maintain its own internal representation of farmOS data structures and
procedures that can interface with those protocols and frameworks. Ultimately,
the Common Engine must be capable of ingesting schemas and logic extensions that
can be interpreted consistently, regardless of their external representation.

Regarding the syncing protocol, this will strictly perform the role of a shared,
deterministic consensus algorithm to handle concurrency across multiple systems.
That introduces another element of logic that must be shared, but fortunately
there exist a number of ready-made solutions for this, particularly
Conflict-free Replicated Data Types (CRDTs), as well as established libraries
like [Automerge] for implementing those data types consistently across disparate
environments. An advantage of CRDTs (and other solutions similarly based upon
data structures and mathematically rigorous transformations; ADTs excel here
too) is that they also remain network-agnostic, so that behavior should remain
predictable no matter the form of communication between nodes. The Automerge
library is actually quite exemplary in this regard, insofar as it restricts
itself to the processing, merging, and transformations of the data structures
alone, in accordance to clearly specified behaviors and mathematical laws (e.g.,
commutativity, associativity, etc). It nevertheless remains thoroughly unaware
of the network, a key element of [Automerge's design principles], which are a
primary source of inspiration for much of this proposal. 

[core requirements]: #core-requirements
[`libcurl`]: https://curl.se/libcurl/
[`libsodium`]: https://libsodium.org
[`libpng`]: http://www.libpng.org/pub/png/libpng.html
[`libsignal`]: https://github.com/signalapp/libsignal
[`libVLC`]: https://wiki.videolan.org/LibVLC
[`libsql`]: https://turso.tech/libsql
[farmOS.js]: https://github.com/farmOS/farmOS.js
[farmOS.py]: https://github.com/farmOS/farmOS.py
[schema-first design]: #schema-first-design
[Algebraic Data Types]: https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/
[Algebraic Property Graphs]: https://arxiv.org/abs/1909.04881
[Spritely Institute]: https://files.spritely.institute/papers/spritely-core.html
[OCapN]: https://ocapn.org/
[Cap'n Proto's RPC protocol]: https://capnproto.org/rpc.html
[schema language]: https://capnproto.org/language.html
[Conflict-free Replicated Data Types (CRDTs)]: https://www.youtube.com/watch?v=B5NULPSiOGw
[Automerge]: https://automerge.org/
[Automerge's design principles]: https://automerge.org/docs/hello/#design-principles

#### Requirements
- Implementation of the farmOS Data Model
  - Core Entities & Bundles
  - Core Logic (preferably as RPC or [IDL])
    - [Location Geometry]
    - [Group Membership]
    - [Inventory Adjustment]
- Syncing Protocol

[Location Geometry]: https://farmos.org/model/logic/location
[Group Membership]: https://farmos.org/model/logic/group
[Inventory Adjustment]: https://farmos.org/model/logic/inventory


### Middlewares & Adapters
Optional libraries for hosting the Common Engine on specific platforms and
environments, or for adapting the farmOS Data Model to other more generic
protocols and standards. Unlike the common engine and application packages
below, these will not be distributed as a single package but many smaller ones,
preferably as granular as possible, though some may also be bundled together
for convenience.

#### Libraries & Components
- Encoders/decoders, validators, etc. for Distributed Protocols
  - [Cap'n Proto] for schema + RPC (or similar [CapTP] implementation)
  - Protocol Buffers, Apache Thrift, or other [IDL] formats
  - Linked Data: [Atomic Data], Solid, [IPLD], JSON-LD or other RDF
  - GraphQL
  - JSON Schema & JSON:API
  - farmOS Conventions
- Language bindings & Foreign Function Interfaces (FFI)
  - PHP
  - Rust
  - C/C++
  - TypeScript/JavaScript (likely superseding [farmOS.js])
  - Python
  - Ruby
  - Elixir
- Framework middlewares, mixins & plugins
  - Server: Express, Rails, Django, Laravel, [Phoenix]
  - CMS: Wordpress, Drupal, etc.
  - Frontend: React, Vue, Svelte, [LiveView], etc.
- Database adapters
  - farmOS on SQLite
    - 1 DB/tenant à la [Turso/libsql]
    - or similar: SqlSync, Electric, VLCN/cr-sqlite
  - Graph databases and triple stores
    - Neo4j
    - Apache Jena
    - General SPARQL support

[Cap'n Proto]: https://capnproto.org/rpc.html
[CapTP]: http://erights.org/elib/distrib/captp/index.html
[IDL]: https://en.wikipedia.org/wiki/Interface_description_language
[IPLD]: https://ipld.io/
[Atomic Data]: https://docs.atomicdata.dev/core/json-ad
[farmOS.js]: https://farmos.org/development/farmos-js/
[Phoenix]: https://www.phoenixframework.org/
[LiveView]: https://hexdocs.pm/phoenix_live_view
[Turso/libsql]: https://turso.tech/multi-tenancy

## Application Packages
### Server
A fully headless, minimalistic server implementation, mainly for the purpose of
providing remote storage and a syncing node for local-first applications,
federated servers, and standard farmOS instances that have the [`farm_runrig`
module] installed, all with very little overhead. For more comprehensive server
functionality, the server can be extended or wrapped by downstream applications,
or integrated into a microservice architecture.

#### Requirements
- Integrates the [Common Engine]
- Extensible via shared [middlewares & adapters], for a diversity of
  - Serialization formats
  - Network protocols
  - Framework middlewares
  - Database connections
  - Connection to the [Plugin Store]

[`farm_runrig` module]: #farmos-module-farm_runrig
[middlewares & adapters]: #middlewares--adapters

### Client
This will be a fairly lightweight client application, practically just a daemon
running from the system tray on desktop or as a background notifier on mobile,
but it will also serve as the host or kernel for the client plugin applications
that can be installed from the [Plugin Store]. In this respect, it will
supersede much of farmOS Field Kit's [intended functionality], like the proposed
[Field Module] system, [app shell], and the Vue composable [`useEntities`] for
CRUD & syncing operations. Much of this functionality will be replaced by the
[Common Engine] and [Plugin Store], but wherever feasible, we'll try to recycle
any relevant portions of Field Kit's codebase. Depending on the platform target
(eg, desktop, mobile, IoT, etc.), the client may come stocked with a "Data
Viewer" or "Tasks List" plugin by default, similar to how Field Kit was planned
to come with the "Tasks" Field Module, so users have at least some kind of
minimal interface for interacting with data, without requiring any additional
[Runrig Server] plugins or Standard farmOS modules as dependencies, at least not
beyond the [Common Engine] and Standard farmOS's core modules, respectively.

#### Requirements
- Integrates the [Common Engine]
- [Runrig Server] Connection
- farmOS JSON:API Connection
- Connection to [Plugin Store]

[intended functionality]: https://github.com/farmOS/field-kit/blob/develop/docs/index.md
[Field Module]: https://github.com/farmOS/field-kit/blob/develop/packages/create-field-module/README.md
[app shell]: https://developer.chrome.com/blog/app-shell/
[`useEntities`]: https://github.com/farmOS/field-kit/blob/e72ce401ae6aa987d7c28f67dce3a64ee9020931/packages/field-kit/src/entities/index.js#L144-L497

### Plugin Store & Schema Clearinghouse
Plugins can wrap various [middlewares & adapters], along with associated
schemas, vocabularies, UI widgets, and other dependencies, to extend the
functionality of servers and clients. By making these different components
available from a known secure provider, federated services and devices can
maintain compatibility with trusted peers within the network more easily and
without sacrificing extensibility. The store can also make useful extensions
more discoverable to end users and admins, with appropriate permissioning where
necessary or desirable, through a GUI web application and CLI package manager.

For relevant open source examples, see:

- [Nextcloud App Store]
  - [source](https://github.com/nextcloud/appstore)
- [Kimai Plugin Store]
  - [source](https://github.com/kimai/www.kimai.org/tree/main/_data/store)
- [Sandstorm App Market]
  - [How Sandstorm Works]
  - [docs](https://docs.sandstorm.io/en/latest/developing/)
- [YunoHost Application Catalog]
  - [docs](https://doc.yunohost.org/en/apps_overview)

[Nextcloud App Store]: https://apps.nextcloud.com/
[Kimai Plugin Store]: https://www.kimai.org/store/
[Sandstorm App Market]: https://apps.sandstorm.io/
[How Sandstorm Works]: https://sandstorm.org/how-it-works
[YunoHost Application Catalog]: https://apps.yunohost.org/catalog

#### Requirements
- Secure signing of authorized plugins
- [Runrig Server] plugins
- [Runrig Client] plugins
- [Common Engine] plugins, extending core schemas & protocols
- Schema & protocol publication
- Versioning & dependency management
- Package manager CLI (probably separate pkg)

The common plugins will mostly serve as dependencies for the client & server
plugins, so users won't install them independently and they can remain hidden
except to developers and sysadmins. As separate plugins, however, multiple
client/server plugins can all depend on one shared common plugin without any
redundancy. They can also depend on different versions of the same common plugin
without corrupting data or landing in dependency hell. To ensure this, the
client/server plugins themselves should be prohibited from making schema
changes, adding protocols, or otherwise altering characteristics of the [Common
Engine]. So if a developer is writing a client or server plugin and needs to
extend a schema, they will have to use a common plugin to do that and declare
the common plugin as a dependency, whether that it already exists or they have
to write their own. Accordingly, all plugins will need to declare what type they
are, `"client"`, `"server"`, or `"common"`, in some sort of package manifest or
config file.

## farmOS Modules
Drupal modules for installation on Standard farmOS.

### `farm_runrig`
This module will allow any Standard farmOS server to act as a client for any
[Runrig Server] instance, as well as a server for any [Runrig Client], although
it might be best to separate that functionality into two or more packages.

#### Requirements
- Wraps the [Common Engine]
- [Runrig Server] Support
- [Runrig Client] Support

### `farm_runrig_fieldkit`
This is essentially a pre-configured [Runrig Client], packaged up as a Drupal
module for Standard farmOS and capable of running in the browser as a [PWA].


[PWA]: https://web.dev/learn/pwa/progressive-web-apps


[Common Engine]: #common-engine-or-libfarmos
[Runrig Server]: #server
[Runrig Client]: #client
[Plugin Store]: #plugin-store--schema-clearinghouse
