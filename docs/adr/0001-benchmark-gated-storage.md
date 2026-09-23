# ADR 0001: Benchmark-gated storage evolution

Status: accepted as baseline
Date: 2026-09-23

## Context

ChessScope needs relational application state, exact position lookup, and large multidimensional aggregates. At 10 million games, an illustrative 80 ply average produces 800 million position occurrences. PostgreSQL, columnar systems, embedded analytics, and key-value stores optimize different parts of this workload.

Choosing the most distributed architecture now would increase operational risk without measured product load. Choosing a storage contract that cannot evolve would create a rewrite later.

## Decision

Start architecture and benchmarks with:

- PostgreSQL for canonical entities, application state, jobs, evidence, and the first position-index candidate;
- object storage for immutable raw inputs and large artifacts;
- Parquet/DuckDB for research, bulk analysis, and portable benchmark data;
- an abstract position-service contract that does not expose physical store keys.

Benchmark PostgreSQL, a columnar candidate such as ClickHouse, and a RocksDB-style reference using identical datasets and query semantics.

Add a specialized position/analytics store only when an agreed latency, ingest, maintenance, rebuild, or cost trigger is crossed.

## Consequences

Positive:

- simple first operational model;
- transactional correctness near metadata;
- measurable migration triggers;
- research can proceed independently in Parquet/DuckDB;
- future physical storage does not redefine evidence IDs.

Negative:

- the first PostgreSQL position schema may be temporary;
- dual-write/backfill work may be required later;
- abstraction discipline is needed before there are multiple stores;
- benchmarks become a mandatory deliverable.

## Not decided

- framework, ORM, hosting provider, or managed database vendor;
- final specialized store;
- exact partitioning and schema;
- maximum supported corpus for the first release.

## Supersession criteria

Supersede this ADR with benchmark evidence, workload traces, cost model, and migration plan—not preference alone.
