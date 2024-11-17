# COHERE Rerank

The **Rerank API** endpoint reorders a list of documents based on their relevance to a given query. It provides relevance scores for each document, enabling improved search and ranking capabilities.

---

## Endpoint

**POST** `https://api.cohere.com/v2/rerank`

---

## Purpose

Re-rank a list of documents based on their relevance to a search query.

---

## Headers

| Header Name         | Type   | Required | Description                              |
|---------------------|--------|----------|------------------------------------------|
| `Authorization`     | String | Required | Bearer token for authentication.        |
| `accept`            | String | Required | Must be set to `application/json`.       |
| `content-type`      | String | Required | Must be set to `application/json`.       |
| `X-Client-Name`     | String | Optional | Name of the project making the request. |

---

## Request Body

The request expects a JSON object with the following structure:

### Required Fields

| Field       | Type        | Description                                                                                     |
|-------------|-------------|-------------------------------------------------------------------------------------------------|
| `model`     | String      | Identifier of the rerank model to use. Options: `rerank-english-v3.0`, `rerank-multilingual-v3.0`, `rerank-english-v2.0`, `rerank-multilingual-v2.0`. |
| `query`     | String      | The search query.                                                                               |
| `documents` | List        | A list of documents (strings or objects) to be reranked. Each document must include a `text` field. |

### Optional Fields

| Field              | Type        | Description                                                                                     |
|--------------------|-------------|-------------------------------------------------------------------------------------------------|
| `top_n`            | Integer     | Number of top relevant documents to return. Defaults to the total number of documents.         |
| `rank_fields`      | List        | Specifies which fields to consider for reranking if documents are JSON objects. Defaults to `text`. |
| `return_documents` | Boolean     | If `true`, includes the document text in the response. Defaults to `true`.                     |
| `max_chunks_per_doc` | Integer   | Maximum chunks per document for internal processing.                                           |

---

### Document Format

Documents can either be plain strings or objects with a `text` field and optional metadata.

**Example Document List:**

```json
[
  "Carson City is the capital city of the American state of Nevada.",
  {
    "text": "Washington, D.C. is the capital of the United States.",
    "id": "document_2",
    "metadata": { "type": "geography" }
  }
]
```

---

## Response

The API returns a JSON object containing the results, including relevance scores for each document.

### Response Fields

| Field              | Type        | Description                                                                                     |
|--------------------|-------------|-------------------------------------------------------------------------------------------------|
| `results`          | List        | List of ranked documents, each with an index, relevance score, and optionally the document text.|
| `id`               | String      | Unique identifier for the API call.                                                           |
| `meta`             | Object      | Metadata about the request, including version and billing information.                        |
| `warnings`         | List        | Optional warnings about the request.                                                          |

### Result Object

Each result contains:

| Field              | Type        | Description                                                                                     |
|--------------------|-------------|-------------------------------------------------------------------------------------------------|
| `index`            | Integer     | Index of the document in the original list.                                                    |
| `relevance_score`  | Double      | Normalized relevance score between 0 and 1.                                                    |
| `document`         | Object      | (Optional) The original document if `return_documents` is `true`.                              |

---

### Example Request

**cURL Command:**

```bash
curl --request POST \
  --url https://api.cohere.com/v2/rerank \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --header "Authorization: Bearer $CO_API_KEY" \
  --data '{
    "model": "rerank-english-v3.0",
    "query": "What is the capital of the United States?",
    "top_n": 3,
    "documents": [
      "Carson City is the capital city of the American state of Nevada.",
      "The Commonwealth of the Northern Mariana Islands is a group of islands in the Pacific Ocean. Its capital is Saipan.",
      "Washington, D.C. (also known as simply Washington or D.C., and officially as the District of Columbia) is the capital of the United States. It is a federal district.",
      "Capitalization or capitalisation in English grammar is the use of a capital letter at the start of a word. English usage varies from capitalization in other languages.",
      "Capital punishment (the death penalty) has existed in the United States since before the United States was a country. As of 2017, capital punishment is legal in 30 of the 50 states."
    ]
  }'
```

---

### Example Response

```json
{
  "results": [
    {
      "index": 2,
      "relevance_score": 0.999071
    },
    {
      "index": 4,
      "relevance_score": 0.7867867
    },
    {
      "index": 0,
      "relevance_score": 0.32713068
    }
  ],
  "id": "07734bd2-2473-4f07-94e1-0d9f0e6843cf",
  "meta": {
    "api_version": {
      "version": "2",
      "is_experimental": true
    },
    "billed_units": {
      "search_units": 1
    },
    "warnings": [
      "You are using an experimental version, for more information please refer to https://docs.cohere.com/versioning-reference"
    ]
  }
}
```

---

### Explanation of Results

- **`index`**: Position of the document in the original `documents` list.
- **`relevance_score`**: A normalized score between 0 and 1 indicating relevance to the query.
- **`meta`**: Additional information, including API version and usage billing.

---

## Key Notes

- **Model Selection**: Choose the correct rerank model (`english` or `multilingual`) based on your document language.
- **Performance**: Keep document count under 1,000 for optimal performance.
- **Relevance Scores**: Scores are normalized but not directly comparable as linear scales (e.g., a score of 0.9 isn't "twice as relevant" as 0.45).

