# Voyage Reranker API Documentation

The **Voyage Reranker API** evaluates and ranks documents based on their relevance to a given query. It supports various models optimized for different use cases and performance requirements.

---

## Endpoint
```
POST https://api.voyageai.com/v1/rerank
```

---

## Authorization

**HTTP Authorization Scheme**: `bearer`  
**Header**:  
```
Authorization: Bearer <VOYAGE_API_KEY>
```

---

## Request Body Parameters

The request body must be sent in **JSON format** with the following fields:

### Required Parameters

| Parameter       | Type                | Description                                                                                                        |
|-----------------|---------------------|--------------------------------------------------------------------------------------------------------------------|
| **query**       | string              | The query string to evaluate relevance. Token limits depend on the model (see **Constraints** below).              |
| **documents**   | array of strings    | A list of documents to be reranked. Token and document count constraints apply (see **Constraints** below).         |
| **model**       | string              | The model name to use for reranking. Recommended options: `rerank-2`, `rerank-2-lite`.                             |

### Optional Parameters

| Parameter           | Type      | Default  | Description                                                                                                        |
|---------------------|-----------|----------|--------------------------------------------------------------------------------------------------------------------|
| **top_k**           | integer   | null     | The number of most relevant documents to return. If not specified, all reranked documents are returned.            |
| **return_documents**| boolean   | false    | Whether to return the document content along with relevance scores.                                                |
| **truncation**      | boolean   | true     | Whether to truncate the query and documents to fit within the context length. If `false`, exceeding limits raises an error. |

---

## Constraints

1. **Query Length**:
   - Maximum tokens:
     - `rerank-2`: **4000 tokens**.
     - `rerank-2-lite`, `rerank-1`: **2000 tokens**.
     - `rerank-lite-1`: **1000 tokens**.

2. **Document Count**:
   - Maximum **1000 documents**.

3. **Token Limits**:
   - The total tokens for a query and any single document cannot exceed:
     - `rerank-2`: **16,000 tokens**.
     - `rerank-2-lite`, `rerank-1`: **8000 tokens**.
     - `rerank-lite-1`: **4000 tokens**.
   - The total tokens across all query and documents cannot exceed **300,000 tokens**. Calculated as:
     ```
     (Query tokens × Number of documents) + Sum of document tokens
     ```

---

## Example Request

### Basic Rerank Request

```bash
curl --request POST \
     --url https://api.voyageai.com/v1/rerank \
     --header "Authorization: Bearer $VOYAGE_API_KEY" \
     --header "content-type: application/json" \
     --data '{
       "query": "Sample query",
       "documents": [
         "Sample document 1",
         "Sample document 2"
       ],
       "model": "rerank-lite-1"
     }'
```

### Request with Top-K Results and Document Return

```bash
curl --request POST \
     --url https://api.voyageai.com/v1/rerank \
     --header "Authorization: Bearer $VOYAGE_API_KEY" \
     --header "content-type: application/json" \
     --data '{
       "query": "Find the most relevant document",
       "documents": [
         "This is document A.",
         "This is document B."
       ],
       "model": "rerank-2",
       "top_k": 1,
       "return_documents": true
     }'
```

---

## Example Response

### Success Response (200)

| Field              | Type                | Description                                                                 |
|--------------------|---------------------|-----------------------------------------------------------------------------|
| **object**         | string              | Always `"list"`.                                                            |
| **data**           | array of objects    | Array of reranked results, sorted by descending relevance score.            |
| **model**          | string              | The model used for reranking.                                               |
| **usage**          | object              | Token usage statistics, including `total_tokens`.                           |

#### Response Example (Without Documents)
```json
{
  "object": "list",
  "data": [
    {
      "relevance_score": 0.4375,
      "index": 0
    },
    {
      "relevance_score": 0.421875,
      "index": 1
    }
  ],
  "model": "rerank-lite-1",
  "usage": {
    "total_tokens": 26
  }
}
```

#### Response Example (With Documents)
```json
{
  "object": "list",
  "data": [
    {
      "relevance_score": 0.675,
      "index": 0,
      "document": "This is document A."
    }
  ],
  "model": "rerank-2",
  "usage": {
    "total_tokens": 50
  }
}
```

---

## Notes

1. **Model Options**:
   - **`rerank-2`**: Best for high token limits (up to 4000 tokens per query, 16,000 combined).
   - **`rerank-2-lite`**: Optimized for lower token limits.
   - **`rerank-lite-1`**: Lightweight model for minimal token requirements.

2. **Token Truncation**:
   - Enable `truncation: true` to automatically handle over-length inputs.
   - If `truncation: false`, errors are raised for queries/documents exceeding token limits.

3. **Returning Documents**:
   - Set `return_documents: true` to include document text in the response alongside relevance scores.

4. **Usage Statistics**:
   - Token usage in the `usage` object helps monitor costs and optimize queries.

5. **Relevance Scoring**:
   - Results are sorted by `relevance_score` in descending order.

---

## Error Responses

### Client Error (4XX)
Indicates issues with the request, such as exceeding token or document limits.

| Field              | Type          | Description                                                                 |
|--------------------|---------------|-----------------------------------------------------------------------------|
| **type**           | string        | The type of error (e.g., `"validation_error"`).                             |
| **message**        | string        | A detailed error message.                                                  |

#### Example
```json
{
  "type": "validation_error",
  "message": "Query exceeds maximum token limit for the selected model."
}
```

### Server Error (5XX)
Indicates server-side issues such as high traffic or unexpected failures.

---
