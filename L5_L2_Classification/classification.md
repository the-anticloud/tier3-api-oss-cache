# L5 Narrow / L2 General Classification — api-oss-cache
**Platform:** Anticloud | **PAX:** 27B | **IP:** USPTO pending 2026, Anticloud FZ LLE
**Domain:** Sovereign response cache: semantic deduplication and KV caching for Anticloud API

## L5 Narrow
api-oss-cache implements two caching layers for Anticloud: exact-match KV cache (SHA3-256 of prompt → response) and semantic cache (embedding similarity → cached response if cosine similarity > 0.95). All cached data is local — no Redis Cloud, no Memcached cluster.

## L2 General
L2 General: every Anticloud deployment benefits from the same caching without configuration. Cache hit rate improvements apply equally to clinical, robotics, and security use cases.

## PAX Integration
PAX 27B embeddings are used for semantic cache lookup: before invoking PAX inference, api-oss-cache checks if a semantically equivalent query has been answered before.

## AIOSS Audit Relevance
Every cache event (query hash + cache hit/miss + response hash + similarity score) is AIOSS-chained: H_n = SHA3-256(H_{n-1} || entry_hash_n || timestamp_n).
Tamper-evident, offline-verifiable, zero cloud dependency.

## Regulatory / Compliance
GDPR Art. 25 (cached data stays local), ISO 27001 A.12.1 (operational procedures)
