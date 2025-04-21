# Next Steps for a Runrig farmOS MVP
Address the high-level questions for each of these areas of concern and scope out the required features for an MVP:

- [x] High-level Scoping
  - [x] [Common Engine Components](#common-engine--core-libraries)
  - [x] [Middleware, Server & Client Demo](#middleware-server--client-demo)
  - [x] [Schema-first Implementation](#schema-first-implementation)
- [ ] Low-level Scoping

## Common Engine & Core Libraries
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
the server via something like CoopCloud or CloudRon?

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
