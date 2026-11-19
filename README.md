<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Amazon API

[![SocQ Amazon API](https://img.shields.io/badge/API-SocQ%20Amazon%20API-F64C31)](https://socq.ai/apis/amazon?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/amazon/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/amazon-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/amazon-api/actions/workflows/check.yml)

6 endpoints for public Amazon data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Amazon APIs](https://socq.ai/apis/amazon?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api) · [Documentation](https://docs.socq.ai/api-manual/amazon/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Amazon Best Sellers API](#amazon-best-sellers-api) | `amazon/best-sellers` | Collect products from public Amazon Best Sellers URLs with normalized titles, seller details, prices, ratings, review counts, sold counts, media, source links, and timestamps. | 0.15 credits ($0.00075) per result | 2026-08-14 |
| [Amazon Product Details API](#amazon-product-details-api) | `amazon/product-detail` | Collect public Amazon product details from known product URLs, returning normalized identity, description, seller, price, rating, review-count, sold-count, and media fields. | 0.15 credits ($0.00075) per result | 2026-08-14 |
| [Amazon Product Search API](#amazon-product-search-api) | `amazon/product-search` | Search a supported Amazon storefront by product phrase and return normalized public product, seller, price, rating, review, sold-count, media, and source fields when available. | 0.15 credits ($0.00075) per result | 2026-08-14 |
| [Amazon Reviews API](#amazon-reviews-api) | `amazon/reviews` | Collect public Amazon product reviews from supported URLs, returning text, ratings, author details, product references, timestamps, and media as normalized records. | 0.15 credits ($0.00075) per result | 2026-08-14 |
| [Amazon Seller Info API](#amazon-seller-info-api) | `amazon/seller-info` | Collect public seller-profile details from supported Amazon marketplaces, returning normalized seller identity, description, website, rating, feedback, location, source, and collection-time fields when available. | 0.15 credits ($0.00075) per result | 2026-08-14 |
| [Amazon Storefront API](#amazon-storefront-api) | `amazon/shop` | Read an Amazon creator storefront: profile, linked socials, idea lists with item counts, curated posts and trending picks with prices. | 0.5 credits ($0.0025) per call | 2026-11-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/amazon-api.git
cd amazon-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs amazon/best-sellers

# Python 3.8+ (standard library only)
python3 examples/python/socq.py amazon/best-sellers payloads/best-sellers.json

# cURL + jq
bash examples/curl/socq.sh amazon/best-sellers '{"urls":["https://www.amazon.com/Best-Sellers-Books/zgbs/books"],"results_limit":3}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Amazon Best Sellers API

Collect products from public Amazon Best Sellers URLs with normalized titles, seller details, prices, ratings, review counts, sold counts, media, source links, and timestamps.

- **Request:** `POST https://api.socq.ai/v1/amazon/best-sellers`
- **publicId:** `amazon/best-sellers`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-14
- **Example body:** [`payloads/best-sellers.json`](payloads/best-sellers.json)

Fetch normalized best sellers data from Amazon.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/amazon/best-sellers" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.amazon.com/Best-Sellers-Books/zgbs/books"
  ],
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/amazon/best-sellers](https://docs.socq.ai/api-manual/amazon/best-sellers) · Landing page: [socq.ai/apis/amazon/best-sellers](https://socq.ai/apis/amazon/best-sellers?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api)

### Amazon Product Details API

Collect public Amazon product details from known product URLs, returning normalized identity, description, seller, price, rating, review-count, sold-count, and media fields.

- **Request:** `POST https://api.socq.ai/v1/amazon/product-detail`
- **publicId:** `amazon/product-detail`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-14
- **Example body:** [`payloads/product-detail.json`](payloads/product-detail.json)

Fetch normalized products data from Amazon.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/amazon/product-detail" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.amazon.com/dp/B0CHHSFMRL"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/amazon/product-detail](https://docs.socq.ai/api-manual/amazon/product-detail) · Landing page: [socq.ai/apis/amazon/product-detail](https://socq.ai/apis/amazon/product-detail?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api)

### Amazon Product Search API

Search a supported Amazon storefront by product phrase and return normalized public product, seller, price, rating, review, sold-count, media, and source fields when available.

- **Request:** `POST https://api.socq.ai/v1/amazon/product-search`
- **publicId:** `amazon/product-search`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-14
- **Example body:** [`payloads/product-search.json`](payloads/product-search.json)

Fetch normalized search data from Amazon.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `domain` | string | Yes | - |
| `query` | string | Yes | Search query. |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |

```bash
curl -X POST "https://api.socq.ai/v1/amazon/product-search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "wireless earbuds",
  "domain": "https://www.amazon.com",
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/amazon/product-search](https://docs.socq.ai/api-manual/amazon/product-search) · Landing page: [socq.ai/apis/amazon/product-search](https://socq.ai/apis/amazon/product-search?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api)

### Amazon Reviews API

Collect public Amazon product reviews from supported URLs, returning text, ratings, author details, product references, timestamps, and media as normalized records.

- **Request:** `POST https://api.socq.ai/v1/amazon/reviews`
- **publicId:** `amazon/reviews`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-14
- **Example body:** [`payloads/reviews.json`](payloads/reviews.json)

Fetch normalized reviews data from Amazon.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/amazon/reviews" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.amazon.com/dp/B0CHHSFMRL"
  ],
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/amazon/reviews](https://docs.socq.ai/api-manual/amazon/reviews) · Landing page: [socq.ai/apis/amazon/reviews](https://socq.ai/apis/amazon/reviews?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api)

### Amazon Seller Info API

Collect public seller-profile details from supported Amazon marketplaces, returning normalized seller identity, description, website, rating, feedback, location, source, and collection-time fields when available.

- **Request:** `POST https://api.socq.ai/v1/amazon/seller-info`
- **publicId:** `amazon/seller-info`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-14
- **Example body:** [`payloads/seller-info.json`](payloads/seller-info.json)

Fetch normalized sellers data from Amazon.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/amazon/seller-info" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.amazon.com/sp?seller=A2L77EE7U53NWQ"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/amazon/seller-info](https://docs.socq.ai/api-manual/amazon/seller-info) · Landing page: [socq.ai/apis/amazon/seller-info](https://socq.ai/apis/amazon/seller-info?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api)

### Amazon Storefront API

Read an Amazon creator storefront: profile, linked socials, idea lists with item counts, curated posts and trending picks with prices.

- **Request:** `POST https://api.socq.ai/v1/amazon/shop`
- **publicId:** `amazon/shop`
- **Price:** 0.5 credits ($0.0025) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/shop.json`](payloads/shop.json)

Creators in the Amazon Influencer Program run a storefront at amazon.com/shop/name where they group products into idea lists and curated posts. Read one storefront to see who the creator is, which social accounts they link, how their lists are organized and how many products each holds, and which items the storefront currently pushes as trending picks, with prices. Useful for scouting affiliate partners or following what a creator recommends this season.

Provide one storefront URL in url, in the form amazon.com/shop/name (other Amazon marketplaces work the same way). Each request costs 0.5 credits and returns one row with the first page of the storefront's lists, curations and trending picks. A storefront that does not exist fails with a refund.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `url` | string | Yes | Amazon storefront URL, such as https://www.amazon.com/shop/sydneydelrey. |

```bash
curl -X POST "https://api.socq.ai/v1/amazon/shop" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.amazon.com/shop/sydneydelrey"
}'
```

Full docs: [https://docs.socq.ai/api-manual/amazon/shop](https://docs.socq.ai/api-manual/amazon/shop) · Landing page: [socq.ai/apis/amazon/shop](https://socq.ai/apis/amazon/shop?utm_source=github&utm_medium=readme&utm_campaign=socq-amazon-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Amazon data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
