## Configuration

Besides commandline flags, Prometheus is configured mainly via a configuration file in ASCII protocol buffer format. This file specifies global defaults, rule files to load, as well as jobs and their targets to scrape.

The configuration is loaded when Prometheus starts up and its subsections are propagated as appropriate to initialize Prometheus' subsystems.

The configuration management code lives in https://github.com/prometheus/prometheus/tree/master/config.

See also [[Configuration]].

## Storage
### Tiers

Prometheus uses a tiered storage model to store and retrieve timeseries data. Samples are initially stored only in memory and are persisted to disk after a configured interval. In the future, samples will also be persisted to a third storage tier, comprised of a remote distributed timeseries database.

**Memory**

The in-memory storage arena is where new samples are stored first. For each distinct timeseries, it manages a skiplist of samples (using http://godoc.org/github.com/ryszard/goskiplist/skiplist), which allow efficient appends and lookups of data points. Samples older than a given age are persisted to disk storage at a configured interval and subsequently deleted from the in-memory storage.

TODO: Explain why the memory storage needs to exist.
TODO: Update this when we have overlapping memory and disk arenas.

**Disk**



**Remote Storage**

### Write Path
### Retrieval Path
### Storage Curation

**Compaction**

**Deletion**

**Downsampling**

## Metrics Ingestion
### Target Management
### Metrics Retrieval (Scraping)
## Expression Evaluation
## Web Interface and API
## Rules Engine
### Recording Rules
### Trigger Rules