[Distributed systems](/architecture/distributed-systems)
# Database per module

Concept that generally came from microservices land but can (and probably should) be adopted into general distributed systems architecture. Seems I'm more eager to accept separate database per group of services realizing concrete business process (or tightly coupled group of them), let's call it *database per module*.

#### Mono database - problems
- **everything can reference everything** (and even with high discipline you'll eventually end up with FK to many loosely related things because "consistency")
- **takes time to backup and restore** (so takes longer to prepare pre-production and UAT environments etc.)
- **generally needs powerful hardware** underneath since it needs to store most of your company's data
- **will eventually stop scaling easily** (or you'll need to spend unproportionally large sums to gain little gains)
- often **ends up as integration platform** instead of being data storage (so called 'integration through database' in distributed monoliths)
- **introduces canonical model** for every piece of data - if there is "order" table, all of the logic must bend to how it is structured (which is problematcuc, since order can be perceived and used differently by different processes and perhaps - needs less data than structure requires)
- **releases can potentially impact unrelated components/modules** (due to most of the above)

#### Separate databases - chances
- **data isolation** - data of particular processes is encapsulated by its service (either web service, Rabbit/Kafka host etc.)
- **business logic isolation** - if data is isolated, you can also enforce business rules upon it when accessing and modifying it (and somehow test it)
- **custom views/projections per application/process** (tweaked for particular case, anti-canonical)
- smaller impact of **schema changes** and **fewer dependencies** - you need to worry less about data and its consumers around you when you change anything
- **granular deployment** - since it's your module and your database, you don't need to stop everything on release

#### Separate databases - replicating state
If you decide to isolate yourself on module (application) level, you'll still find yourself needing some of the state from other stores to be present locally. There are few approaches to achieve that:
- replicate via native mechanics (example: SQL Server replication)
- organize scheduled ETLs
- publish events on changes from application that owns data and project it in your database in a scope you need it
- requesting data via public interfaces, on demand; mostly through web services

Generally my preferred way is by publishing events since it the most decoupled way you can go (subscriber can even be offline when event is published). However, such approach has several challenges that need to be addressed, ie: 
- **eventual consistency** - data won't be actual all the time
- **data can lose integrity** - there are no FKs in here; also could cause some conflicts on events like deleting/archiving some business entity
- **changes in events structure** need to be address somehow if they don't match
- **maintenance cost is huge** - you are now maintaining several databases, messaging broker and applications communicating with it
- **needs even more extensive monitoring and logging** - common pitfall of anything distributed
- **backups are tricky** - you published some data and then you are rolling back because "business says so" - what about the data that have been processed by subscribers already?
- **events need to be versioned and idempotently processed** - there is really no other way round; most messaging brokers will retry messages when something goes wrong (google: *Fallacies of distributed computing*)

#### Separate databases - missing parts
- **identifier strategies** - GUIDs vs sequentional ids vs natural keys; GUID seems natural for distributed but would need some sort of sequentionallity for indexing, something like [flake](https://github.com/boundary/flake) and its .NET implementation [NewId](https://github.com/phatboyg/NewId)
- **dealing with transactions** - R. Roger proposes pattern from Rotem-Gal-Oz's *SOA Patterns*, ie. to use reservations (and commiting or canceling them locally); of course provided we forget about distributed transactions on SQL level (since we're isolating databases on app level)

#### Inspirations
* Richard Roger, *The Tao of Microservices*
* Martin Kleppman, *Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems*
* Chris Richardson, *https://microservices.io*
