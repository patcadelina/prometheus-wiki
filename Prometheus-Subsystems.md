## Configuration

Besides commandline flags, Prometheus is configured mainly via a configuration file in ASCII protocol buffer format. This file specifies global defaults, rule files to load, as well as jobs and their targets to scrape.

The configuration is loaded when Prometheus starts up and its subsections are propagated as appropriate to initialize Prometheus' subsystems.

The configuration management code lives in https://github.com/prometheus/prometheus/tree/master/config.
See also https://github.com/prometheus/prometheus/wiki/Configuration.

## Storage
### Tiers

**Memory**

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