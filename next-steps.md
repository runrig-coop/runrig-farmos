# Next Steps for a Runrig farmOS MVP
Address the high-level questions for each of these areas of concern and scope out the required features for an MVP:

- [ ] [Common Engine Components](#common-engine-components)
  - farmOS Records
  - farmOS Logic
  - Syncing or Reconciliation
- [ ] [Middlewares & Client-Server Demo](#middlewares--client-server-demo)
  - Middleware
  - Packaging Utilities
  - Server
  - Client
- [ ] [Schema-first Implementation](#schema-first-implementation)
  - Refine the farmOS Data Model
  - Review prior art
  - Assess schema, RPC, and ontology candidates

## Common Engine Components
What specific packages and components of the Common Engine would have provided a
viable Runrig farmOS server that I could have deployed for Farm Flow if they had
been available at the start of 2024? Can farmOS.js form the basis for this?

- farmOS Records
  - Entities, including new additions:
    - `farm_organization`
    - `farm_plan_record`
  - Bundles
  - Attributes
  - Relationships
- farmOS Logic
  - [Location Geometry]
  - [Group Membership]
  - [Inventory Adjustment]
- Syncing or Reconciliation Protocol
  - ie, merging w/o network

## Middlewares & Client-Server Demo
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
- Server (ideally just a bundle of the above middlewares)
- Client (demo)
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

- Refine the farmOS Data Model
  - Logic as transforms, capabilities, or RPC methods
  - Computed values separate from attributes
  - Target values separate from attributes
  - SOPs as _a vocabulary of predefined transform parameters_
- Review Prior Art
  - Sandstorm (and other plugin stores/exchanges/commons)
  - NALT & Nemo Inference Engine
  - DFC, Discover Regenerative, & OFN
  - Our Sci Conventions & farmOS Mirror
  - Conexus, APGs, & Hydra
- Assess Schema, RPC, and Ontology Candidates
  - Capn'n Proto
  - JSON-LD
  - Atomic
  - ShEx
  - ADTs (eg, Fantasy Land)
  - CUE

__MAIN TAKEAWAY:__ If farmOS records and logic can be represented as Cap'n Proto
Schema/RPCs (ie, `.capnp` files) or equivalent, can the various types, structs,
interfaces, methods, and fields in Cap'n Proto be mapped to existing RDF
vocabularies and/or an original farmOS ontology akin to the DFC?

[Location Geometry]: https://farmos.org/model/logic/location
[Group Membership]: https://farmos.org/model/logic/group
[Inventory Adjustment]: https://farmos.org/model/logic/inventory
