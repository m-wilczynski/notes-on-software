[Databases](/engineering/databases)
# Elasticsearch index

### Database itself

**Elasticsearch** is a (probably) most popular full-text search, distributed database used nowadays. It builts upon Java-based full-text search index called **Lucene**.

Elasticsearch is used mostly for augmenting platforms with efficient phrase search and also for easy centralized log browsing for either application, APM or security logs and metrics (especially when augmented with **Logstash** and **Kibana**).

The building block of Elasticsearch and Lucene engine under-the-hood is **an index**, which... holds more than one meaning in this context.
Since Elasticsearch creators abuse **"index"** as a name a lot, I've decided to try to organize my understanding of it in my head. 

H|Elasticsearch index, shard, replica and segment|Engineering/Databases|M|

### Index is a "database instance"

From high-level view, Elasticsearch **index is sort of a database instance on database server**. Behind the scenes an Elasticsearch index is actually a logical groupping of one or more shards. 

Shard - as in any other distributed database - is a way of partitioning data between database servers. Elasticsearch uses document-based partitioning based on document ID. There are "**primary**" shards (actual data) and their failover copies - "**replicas**". Primary and replica shard (of same data) cannot be hosted on a same Elasticsearch node. 

### Index is a "Lucene engine instance" 

Behind the scenes, **shard** is an instance of **Lucene index** (sometimes in this context called a Lucene search engine - maybe because internally, Lucene index in a shard is fully independent?).
What shards (or Lucene engine instances) actually store are (JSON) documents that "are indexed" to be efficiently searched (ie. full-text search).

### Index is an "inverted index"

Documents stored in a shard/Lucene index get indexed using *inverted index* data structure.<br>
*A little more about inverted index in seperate note here: [Inverted index](/engineering/computer-science/inverted-index)*

Lucene stores inverted indexes (indices?) held in files called **segments** - that's where actual search gets performed (segment by segment - in sequence). Segments are sometimes called "secondary index" (I would say that's fourth index in a hierarchy, but hey...). 

Since segments are immutable (they are created on data ingestion) and are searched sequentionally, for search performance reasons they need to be **merged**. That's something that should be considered when planning node and cluster resources, since CPU and IO costs are high for such operation.

### Implications of distributed index

Since Elasticsearch uses document-based parition strategy, writes are easy (since each shard contains defined range of IDs) but reads are a little tricky. **Paritioning by ID forces searching every shard** (since data is distributed between them). In general - as in Elasticsearch case - it gets resolved by performing **parallel requests to each shard on search with *scatter-and-collect* strategy**.