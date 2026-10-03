# Deploy Guide — api-oss-cache
**Platform:** Anticloud sovereign infrastructure | Air-gap capable
**Stack:** Python 3.11, SQLite, FAISS (semantic cache), AIOSS_FORMAT

## Prerequisites
Python 3.11+, FAISS-cpu 1.7+, sentence-transformers 2.6+, SQLite (stdlib)

## AIOSS Integration
```bash
aioss init --module api-oss-cache --output ./api_oss_cache.aioss
aioss append --chain ./api_oss_cache.aioss --payload ./output.bin --module api-oss-cache
aioss verify --chain ./api_oss_cache.aioss
```

## Air-Gap Deployment
```bash
# On networked machine:
pip download -r requirements.txt -d ./wheels/
# Transfer wheels/ to air-gap host, then:
pip install --no-index --find-links ./wheels/ -r requirements.txt
```

## PAX 27B Harness Wiring
```python
from anticloud_pax import PAXHarness
harness = PAXHarness(
    model_path="./pax-27b-q4.gguf",
    module="api-oss-cache",
    aioss_chain="./api_oss_cache.aioss",
    classification="L5_NARROW_L2_GENERAL"
)
result = harness.process(input_data)
```

## Verification
```bash
aioss verify --chain ./api_oss_cache.aioss --verbose
python -m api_oss_cache.tests.smoke
```
