# Common Engine or `libfarmOS`

## TODO
### Preparation
- [ ] Decide on a language/runtime to start with
  - Modelling language: Preserves or CUE?
  - Wire protocol/schema: Cap'n Proto, Syrup, or JSON Schema?
  - Runtime language: Rust, TypeScript, or Gleam?
- [ ] Migrate to Codeberg
  - [ ] Add remotes & push updates
    - [ ] the-runrig-plan
    - [ ] internal-notes
    - [ ] runrig-farmos
  - [ ] Open relevant issues
  - [ ] Add issues to Kanban(s)
- [ ] Address remaining Qs
- [ ] Re-org dir structure

### Primary Data Structures
- [ ] Model Assets & Logs in Preserves
- [ ] Implement Assets & Logs in Rust
- [ ] Implement CRUD for Assets & Logs based on schemas
- [ ] Primary relationships b/t entities
- [ ] Derive computed attrs/rels from multiple structures


## Outstanding Questions
- Do this first in TypeScript or Rust?
- What can be applied from ActivityStreams 2.0 (core or vocab)?
- Asset as a Functor/Function of Log/Event?
  - What individual attrs/rels can be computed from logs or some more atomic
    events?
- Can the Preserves data model be used internally?
  - Will that aid in the eventual use of OCapN as transport protocol?
  - Syrup as a wire format?
  - Would it translate well to other formats, like Cap'n Proto or JSON Schema?
  - Would it make implementation in Rust easier?

## Scope
- Languages & runtimes for modelling, implementing, validating, etc.
  - Rust
  - TypeScript
  - Preserves
  - OCapN/Syrup
  - Cap'n Proto
  - ActivityStreams
- Other data formats (maybe for middleware phase)
  - JSON Schema
  - JSON:API
  - JSON-LD
  - RDF (Agrovoc, DFC, SKOS, FOAF, etc)
- Fundamental data structures (entities)
  - Primary structures
    - Assets
    - Logs
  - Secondary structures
    - Quantities
    - Terms
  - Users (see auth)
- Ancillary data structures
  - Planning
    - Plans
    - Plan Records
  - I/O
    - Streams
    - Files
- Auth & User Entities
  - Users: a primary structure, but...
    - less about structured data, and
    - more about auth protocols.
  - Auth protocols?
    - CapTP
    - OAuth
    - DID or other decentralized protocol?
  - Figure out the boundary for between `libfarmOS` & auth protocols


## Logs as Functions
Verb classes as they pertain to different actions that may be taken on an asset

- Produce
  - Create (generate new UUID)
  - Destroy
  - Convert (type A -> type B)
  - Combine (1 of type A + 5 of type B = 3 of type C)
  - Divide (vice versa)
  - Purchase
  - Sell
- Group
- Location Logs
  - Move
  - Find
  - Assign
  - Originate
- Assess
  - Observe
  - Measure
  - Test
  - Medical
- Modify
  - Seed
  - Transplant
  - Harvest
  - Amend
  - Input
  - Cultivate
  - Mature

Should these verb classes be categorized based on the value type of whatever
attrs or rels must be modified in the process? That will be a more effective and
an organic way of composing the structures from the ground up.

Adding, removing or otherwise changing an asset's relationship to a taxonomy
term, most often, represents a means of _classifying_ the asset as a particular
type of animal, equipment, material, etc. This is similar to changing a `string`
or `enum` type of an attribute, such as the land type, which is essentially an
enumerated vocabulary defined by the developer rather than a taxonomy term
defined by the user. In both cases, it doesn't imply an alteration to the
asset's physical being, only how the asset is classified or described.

On the other hand, changing an asset's relationship to another may indicate a
physical change has taken place. An asset's relationship to a user indicates a
transfer of ownership, custody, or responsibility, possibly corresponding to
other changes to the asset's location or status.

Logs have many more complex relationships to many other classes of entity, but
as the properties of events, rather than the physical characteristics of an
object, there may be more flexibility with log attrs & rels.

