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
(or at least not prohibit) these three core requirements, wherever applicable:

1. __Interoperation with Standard farmOS (v 3.0 or higher)__, if not in the form
  of realtime networked communication then at least by maintaining compatibility
  with data migrated to or from Standard servers.
2. __Multi-farm data representations and behavior__, either as a [farmOS
  Organization] entity or as a compatible superset thereof.
3. __A shared, deterministic consensus algorithm__, providing eventual
  consistency and predictable conflict resolution across concurrent devices and
  storage instances.

The hope is that federated, P2P, and local-first architectures will emerge as
convenient byproducts of these three requirements alone, so support for
distributed architectures is not listed here specifically; however, that is
still considered an essential feature in its own right, so the core requirements
may be appended or amended if it does not emerge automatically as expected.

[farmOS Organization]: https://github.com/farmOS/farmOS/pull/849

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

### Common Engine
The Common Engine will form the primary means of implementing the farmOS Data
Model and other [core requirements] stated above.

The Core Logic in particular will require special care to implement without
violating the principle of [schema-first design]. A degree of schema enforcement
can be achieved with remote procedure calls (RPC), but that won't necessarily
enforce the particulars of a given algorithm; mainly RPC would just guarantee a
consistent type signature for each procedure – e.g., `getGeometry :: Asset ->
Geometry`. Some amount of code-sharing will likely prove unavoidable in the
first implementations of Runrig farmOS. Thankfully, the occasions of specific
logic in the farmOS Data Model are fairly limited (only the three listed in the
requirements below at the time of this draft), and changes are unlikely to be
introduced to Standard between major version releases. The portability of the
core packages will likely suffice to propagate consistent implementation of
these algorithms. As necessity dictates, greater consistency might be achieved
with the adoption of more rigorous type systems, such as [Algebraic Data Types]
(ADTs). Even more comprehensive, in-band specification and enforcement
mechanisms might be possible via more innovative solutions, such as [Algebraic
Property Graphs] (essentially a marriage of IDL/RPC and RDF but reinforced by
using ADTs as its core primitives) or the distributed protocols in development
by the [Spritely Institute], like [OCapN], which expands upon Mark S. Miller's
[CapTP] with a particular eye towards improving federated protocols like
ActivityPub. As much as possible, OCapN aims to be backwards compatible with
certain existing implementations of CapTP, like Cap'n Proto, which could make
[Cap'n Proto's RPC protocol] and its [schema language] an especially attractive
target for early implementations.

Regarding the syncing protocol, this will essentially represent a shared,
deterministic consensus algorithm to handle concurrency across multiple systems.
That introduces another element of logic that must be shared, but fortunately
there exist a number of ready-made solutions for this, particularly
Conflict-free Replicated Data Types (CRDTs), as well as established libraries
like [Automerge] for implementing those data types consistently across disparate
environments. An advantage of CRDTs (and other solutions similarly based upon
data structures and mathematically rigorous transformations; ADTs excel here
too) is that they also remain network-agnostic, so that behavior should remain
predictable no matter the form of communication between nodes.

[core requirements]: #core-requirements
[schema-first design]: #schema-first-design
[Algebraic Data Types]: https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/
[Algebraic Property Graphs]: https://arxiv.org/abs/1909.04881
[Spritely Institute]: https://files.spritely.institute/papers/spritely-core.html
[OCapN]: https://ocapn.org/
[Cap'n Proto's RPC protocol]: https://capnproto.org/rpc.html
[schema language]: https://capnproto.org/language.html
[Conflict-free Replicated Data Types (CRDTs)]: https://www.youtube.com/watch?v=B5NULPSiOGw
[Automerge]: https://automerge.org/

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
  - TypeScript/JavaScript
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
[Phoenix]: https://www.phoenixframework.org/
[LiveView]: https://hexdocs.pm/phoenix_live_view
[Turso/libsql]: https://turso.tech/multi-tenancy

## Application Packages
### Server
A fully headless server that mostly provides remote storage and a syncing node
for local-first applications, federated servers, and standard farmOS instances
that have the [`farm_runrig` module](#farmos-module-farm_runrig) installed.

#### Requirements
- Wraps the [Common Engine], integrating components for:
  - Syncing
  - farmOS Data Model
  - Protocol Support
- farmOS JSON:API endpoints
- Connection to [Plugin Store]

### Client
This will be a fairly lightweight client application, practically just a daemon
running from the system tray on desktop or as a background notifier on mobile,
but it will also serve as the host or kernel for the client plugin applications
that can be installed from the [Plugin Store] and will act as the [app shell]
for the plugins' UI. In this respect, it will be quite similar to farmOS Field
Kit's corresponding field modules and app shell, perhaps even recycling some
of Field Kit's code where it hasn't been superseded by the [Common Engine] or
the [Plugin Store]. Depending on the platform target (eg, desktop, mobile, IoT,
etc.), the client may come stocked with a "Data Viewer" or "Tasks List" plugin
by default, similar to how Field Kit was planned to come with the "Tasks" Field
Module, so users have at least some kind of minimal interface for interacting
with data, without requiring any additional [Runrig Server] plugins or Standard
farmOS modules as dependencies, at least not beyond the [Common Engine] and
Standard farmOS's core modules, respectively.

#### Requirements
- Wraps the [Common Engine]
- [Runrig Server] Connection
- farmOS JSON:API Connection
- Connection to [Plugin Store]

[app shell]: https://developer.chrome.com/blog/app-shell/

### Plugin Store
For examples, see the [Nextcloud App Store](https://apps.nextcloud.com/) or
[Sandstorm](https://apps.sandstorm.io/).

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


[Common Engine]: #common-engine
[Runrig Server]: #server
[Runrig Client]: #client
[Plugin Store]: #plugin-store
