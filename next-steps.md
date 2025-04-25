# Next Steps for a Runrig farmOS MVP
Address the high-level questions for each of these areas of concern and scope out the required features for an MVP:

- [x] High-level Scoping
  - [x] [Common Engine Components](#common-engine-or-libfarmos)
  - [x] [Middleware, Server & Client Demo](#middleware-server--client-demo)
  - [x] [Schema-first Implementation](#schema-first-implementation)
- [ ] Low-level Scoping

## Common Engine or `libfarmOS`
What specific packages and components of the Common Engine would have provided a
viable Runrig farmOS server that I could have deployed behind Farm Flow if they
had been available at the start of 2024? Can farmOS.js form the basis for this?

- farmOS Records
  - Entities, including new additions:
    - `farm_organization`
    - `farm_plan_record`
  - Bundles
  - Attributes
  - Relationships
- Internal Schema Representation
  - ie, agnostic of formats like Cap'n Proto or JSON Schema
- farmOS Logic
  - [Location Geometry]
  - [Group Membership]
  - [Inventory Adjustment]
- Syncing or Reconciliation Protocol
  - ie, merging w/o network

## Middleware, Server & Client Demo
Which server frameworks, protocols, and databases will require middlewares to
support this first implementation? Is a client implementation necessary for
demonstration purposes at this point? Or a CI/CD utility to replicate and deploy
the server via something like CoopCloud or YunoHost?

- Middleware
  - Express
  - HTTP
  - JSON:API
  - JSON Schema
  - SQLite (native & WASM)
  - TypeScript
- Packaging utilities
  - Docker
  - npm
  - CoopCloud recipe
- Server
  - ie, ideally just a bundle of middlewares deployed as a Docker image 
- Client Demo
  - Field Kit
  - Farm Flow
  - Projection Engine
  - Something for 607?

### A Municipal Platform
As a prospective demonstration application, I'm considering how I can
sustainably build a municipal platform for the Hudson Valley & NYC. I would be
willing to get this started as a mutual aid project, perhaps in exchange for a
CSA share or other form of material support, rather than paying for it by
private contract or a grant.

Among other services, the platform could act as a proving ground to deploy
`libfarmOS`, giving farmers multiple, secure and flexible options for how to
access their farmOS data. That could take the shape of a dedicated Standard
farmOS instance, hosted in a similar fashion to [Farmier]. It could be persisted
on a headless, multi-tenant Runrig farmOS server built on Express and Postgres,
with separate means of accessing it through various client applications like
[Farm Flow], [Field Kit], or even [NocoDB]. If they don't want a full farmOS
instance or running server, they can keep the data as a SQLite file they can
store on their own device, perhaps as part of a simple local-first app, then
sync that data to the municipal platform as a backup and as an added node for
availability, a la Turso's ["database per tenant"] model built on [`libsql`].
These options are not mutually exclusive, either; a local-first app could be
used to access a Standard farmOS instance or data on an Express server.
Regardless, `libfarmOS` can provide the engine for inter-service communication
in all of these scenarios, allowing data to be aggregated and shared with other
platform members and services, like a cooperative CSA program that aggregates
products from several farms. If a farmer doesn't even care to ever look at that
data, it can still be held securely for them in a separate SQLite file, Postgres
table, or triplestore for later retrieval whenever they wish, or it can be
deleted when it's of no further use to any aggregate services; that should
always remain the farmer's prerogative.

The same principles can be applied to other sources of standardized data, such
as the [DFC Standard] used by Open Food Network, as well as federated services
like [CoopCycle].

[Farmier]: https://farmier.com/
[Farm Flow]: https://runrig.org/projects/farm-flow/strategy
[Field Kit]: https://github.com/farmOS/field-kit/
[NocoDB]: https://docs.nocodb.com/data-sources/data-source-overview
["database per tenant"]: https://turso.tech/multi-tenancy
[`libsql`]: https://github.com/tursodatabase/libsql
[DFC Standard]: https://dfc-standard.org/
[CoopCycle]: https://coopcycle.org/logiciel/


## Schema-first Implementation
Can the farmOS Data Model be specified more formally? If so, how? And is that a
prerequisite to developing the Common Engine? What aspects of the farmOS Logic
need to be schematized or serialized? Is it a requirement for this first
implementation? What schema language(s) should be used in this first
implementation? Cap'n Proto? JSON Schema?

- [Rederive the farmOS Data Model] from first principles
  - Logic as transforms, capabilities, or RPC methods
  - Computed values separate from attributes
  - Target values separate from attributes
  - SOPs as _a vocabulary of predefined transform parameters_
- Review Prior Art
  - Plugin stores/exchanges/commons, particularly [Sandstorm]
  - NALT & Nemo Inference Engine
  - [DFC Standard], Discover Regenerative, & OFN
  - Our Sci Conventions & farmOS Mirror
  - Conexus, APGs, & Hydra
- Assess Schema, RPC, and Ontology Candidates
  - [Cap'n Proto], [OCapN], or another [CapTP] implementation
  - [ATProto Lexicon]
  - [Atomic Data]
  - [JSON-LD]
  - [ShEx]
  - ADTs (a la [Fantasy Land])
  - [CUE]
  - [IPLD]
  - [RAMP Shapes] (RDF ADT Mapping)
    - tagline: "declarative RDF ↔ algebraic data type mapping"

### FINAL TAKEAWAY
__If farmOS records and logic can be represented in Cap'n Proto Schema, JSON__
__Schema, CUE, or their equivalent, can the various schema types, their__
__constituent interfaces, methods, and fields then be mapped to existing RDF__
__vocabularies or an original farmOS ontology, akin to or even as an extension__
__of the DFC Standard, with additional mappings to the ADTs in the RAMP Shapes__
__specification?__

[Location Geometry]: https://farmos.org/model/logic/location
[Group Membership]: https://farmos.org/model/logic/group
[Inventory Adjustment]: https://farmos.org/model/logic/inventory
[Rederive the farmOS Data Model]: ./farmos-data-model-rederived.md
[Sandstorm]: https://sandstorm.org/how-it-works
[DFC Standard]: https://dfc-standard.org/
[Cap'n Proto]: https://capnproto.org/rpc.html
[OCapN]: https://ocapn.org/
[CapTP]: http://erights.org/elib/distrib/captp/index.html
[ATProto Lexicon]: https://atproto.com/guides/lexicon
[Atomic Data]: https://docs.atomicdata.dev/core/json-ad
[JSON-LD]: https://json-ld.org/
[ShEx]: https://shex.io/
[Fantasy Land]: https://github.com/fantasyland/fantasy-land
[CUE]: https://cuelang.org/docs/tour/basics/json-superset/
[IPLD]: https://ipld.io/
[RAMP Shapes]: https://ramp-shapes.github.io/
