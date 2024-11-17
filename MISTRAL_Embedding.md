
# MISTRAL Embedding

The **Embeddings API** generates vector representations of input text that can be used in downstream tasks such as similarity comparisons, clustering, or search indexing.

---

## Authorization

**HTTP Authorization Scheme**: `bearer`  
**Header**:  
```
Authorization: Bearer <API_KEY>
```

---

## Endpoint
```
POST https://api.mistral.ai/v1/embeddings
```

---

## Request Body Schema

The request body must be sent in **JSON format** with the following fields:

### Required Parameters

| Parameter           | Type                     | Description                                                      |
|---------------------|--------------------------|------------------------------------------------------------------|
| **input**           | string or array of strings | The text or list of texts to embed.                             |
| **model**           | string                   | The ID of the embedding model to use. Default: `"mistral-embed"`. |

### Optional Parameters

| Parameter           | Type                     | Default       | Description                                                      |
|---------------------|--------------------------|---------------|------------------------------------------------------------------|
| **encoding_format** | string or null           | `"float"`     | The format for the returned embeddings. Use `"float"` for numerical vectors. |

---

## Example Request

### Single Input
```bash
curl --location "https://api.mistral.ai/v1/embeddings" \
     --header "Content-Type: application/json" \
     --header "Authorization: Bearer $MISTRAL_API_KEY" \
     --data '{
        "input": "Embed this sentence.",
        "model": "mistral-embed",
        "encoding_format": "float"
      }'
```

### Batch Input
```bash
curl --location "https://api.mistral.ai/v1/embeddings" \
     --header "Content-Type: application/json" \
     --header "Authorization: Bearer $MISTRAL_API_KEY" \
     --data '{
        "input": [
          "Embed this sentence.",
          "As well as this one."
        ],
        "model": "mistral-embed",
        "encoding_format": "float"
      }'
```

---

## Responses

### Success Response (200)

| Field              | Type          | Description                                                                 |
|--------------------|---------------|-----------------------------------------------------------------------------|
| **id**             | string        | Unique identifier for the embedding request.                               |
| **object**         | string        | Always `"embedding"`.                                                      |
| **model**          | string        | The model used to generate the embeddings.                                 |
| **usage**          | object        | Token usage details, including `prompt_tokens`, `completion_tokens`, and `total_tokens`. |
| **data**           | array         | A list of embeddings. Each embedding contains an object, an array of numerical vectors, and an index. |

#### Example Success Response
```json
{
  "id": "cmpl-e5cc70bb28c444948073e77776eb30ef",
  "object": "embedding",
  "model": "mistral-embed",
  "usage": {
    "prompt_tokens": 16,
    "completion_tokens": 34,
    "total_tokens": 50
  },
  "data": [
    {
      "object": "embedding",
      "embedding": [0.1, 0.2, 0.3],
      "index": 0
    },
    {
      "object": "embedding",
      "embedding": [0.4, 0.5, 0.6],
      "index": 1
    }
  ]
}
```

### Error Response (422)

If validation fails, the API will return an error response.

| Field              | Type          | Description                                                                 |
|--------------------|---------------|-----------------------------------------------------------------------------|
| **detail**         | array         | List of error details.                                                     |
| **loc**            | array         | The location of the error.                                                 |
| **msg**            | string        | A detailed error message.                                                  |
| **type**           | string        | The type of error.                                                         |

#### Example Error Response
```json
{
  "detail": [
    {
      "loc": ["body", "input"],
      "msg": "Invalid input text format.",
      "type": "validation_error"
    }
  ]
}
```

---

## Notes

1. **Batch Processing**: The API supports multiple inputs in a single request. Each input is embedded separately, and the results are returned in the `data` array, indexed by their order in the input array.
2. **Embedding Format**: The embeddings are numerical vectors returned in the `encoding_format` specified (default is `"float"`).
3. **Model Selection**: Use the `"mistral-embed"` model for standard embeddings. Additional models may be available for specialized tasks.
4. **Token Usage**: The `usage` field provides information about token consumption for billing and optimization purposes.
5. **Error Handling**: Ensure inputs are properly formatted as strings or arrays of strings to avoid validation errors.

---
