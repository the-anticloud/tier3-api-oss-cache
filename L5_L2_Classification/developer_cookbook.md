# Developer Cookbook — api-oss-cache
**Stack:** Python 3.11, SQLite, FAISS (semantic cache), AIOSS_FORMAT
**Domain:** Sovereign response cache: semantic deduplication and KV caching for Anticloud API
**License:** Apache-2.0 | **IP:** USPTO pending 2026, Anticloud FZ LLE

## Core Usage

```python
from api_oss_cache import SemanticCache

cache = SemanticCache(
    db_path="./cache.db",
    embedding_model="all-MiniLM-L6-v2",
    similarity_threshold=0.95,
    aioss_chain="./cache.aioss"
)

# Check cache before PAX inference
cached = cache.lookup("What is the AIOSS chain format?")
if cached.hit:
    print(f"Cache hit (similarity={cached.similarity:.3f}): {cached.response}")
else:
    response = pax.infer(prompt)
    cache.store(prompt, response)
    print(f"Cache miss → PAX inference: {response}")
```

```python
# Exact-match cache (fastest)
response = cache.exact_lookup(prompt_hash)

# Cache stats
stats = cache.stats()
print(f"Hit rate: {stats.hit_rate:.1%}, Entries: {stats.n_entries}, Size: {stats.size_mb:.0f}MB")
```

## AIOSS Chain Append

```python
import hashlib, time

def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()

# After every api-oss-cache output:
chain_hash = aioss_append("./api_oss_cache.aioss",
                           result_bytes, "api-oss-cache")
```

## Performance & Integration

Semantic cache index: FAISS IVF with nlist=256. Embedding batch-precompute on startup for all historical prompts. SQLite for exact-match (O(1) lookup). Integration: sits in front of PAX_API_GATEWAY (T2), uses PAX_EMBEDDINGS (T2) for vector computation.
