# VOYAGE Embedding

The **Voyage Embeddings API** generates vector embeddings for input text using specialized models. These embeddings can be used for tasks such as similarity comparison, clustering, and retrieval in domain-specific applications.

---

## Endpoint
```
POST https://api.voyageai.com/v1/embeddings
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

| Parameter       | Type                       | Description                                                                                                        |
|-----------------|----------------------------|--------------------------------------------------------------------------------------------------------------------|
| **input**       | string or array of strings | Text input for embedding. Supports a single string or a list of strings (max 128). See constraints in the notes.   |
| **model**       | string                     | The name of the model to use. Recommended options: `voyage-3`, `voyage-3-lite`, `voyage-finance-2`, etc.           |

### Optional Parameters

| Parameter       | Type     | Default | Description                                                                                                        |
|-----------------|----------|---------|--------------------------------------------------------------------------------------------------------------------|
| **input_type**  | string   | null    | Specifies the type of input text. Options: `query`, `document`.                                                    |
| **truncation**  | boolean  | true    | Whether to truncate input texts to fit the model's context length. If `false`, overly long texts will raise an error. |
| **encoding_format** | string | null   | Output format for embeddings. Options: `null` (list of floats), `base64` (compressed base64 encodings).             |

---

## Constraints

1. **List Size**:
   - Maximum of **128 texts** per request.
2. **Token Limits**:
   - **voyage-3-lite**: 1M tokens.
   - **voyage-3** and **voyage-2**: 320K tokens.
   - Domain-specific models (e.g., `voyage-law-2`, `voyage-finance-2`): 120K tokens.
3. **Text Length**:
   - Input texts exceeding the model's context length must either be truncated (if `truncation: true`) or will raise an error.

---

## Example Request

### Basic Request

```bash
curl --request POST \
     --url https://api.voyageai.com/v1/embeddings \
     --header "Authorization: Bearer $VOYAGE_API_KEY" \
     --header "content-type: application/json" \
     --data '{
       "input": [
         "Sample text 1",
         "Sample text 2"
       ],
       "model": "voyage-large-2"
     }'
```

---

## Example Response

### Success Response (200)

| Field              | Type                | Description                                                                 |
|--------------------|---------------------|-----------------------------------------------------------------------------|
| **object**         | string              | Always `"list"`.                                                            |
| **data**           | array of objects    | A list of embeddings generated for each input text.                         |
| **model**          | string              | The model used for generating embeddings.                                   |
| **usage**          | object              | Contains token usage statistics (`total_tokens`).                           |

#### Response Example
```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [
        0.0038915484,
        0.010964915,
        -0.035594109,
        "...",
        0.011034692
      ],
      "index": 0
    },
    {
      "object": "embedding",
      "embedding": [
        -0.01539533,
        -0.0011246679,
        0.021264801,
        "...",
        -0.046319865
      ],
      "index": 1
    }
  ],
  "model": "voyage-large-2",
  "usage": {
    "total_tokens": 10
  }
}
```

### Embedding Object Structure

| Field          | Type              | Description                                                                 |
|----------------|-------------------|-----------------------------------------------------------------------------|
| **object**     | string            | Always `"embedding"`.                                                      |
| **embedding**  | array of numbers  | The generated embedding vector as a list of floating-point numbers.        |
| **index**      | integer           | Index of the embedding in the input list.                                  |

---

### Error Responses

#### Client Error (4XX)
Indicates an issue with the request format or constraints (e.g., too many tokens or invalid input).

| Field          | Type                | Description                                                                 |
|----------------|---------------------|-----------------------------------------------------------------------------|
| **detail**     | array of objects    | List of errors explaining the issue.                                       |
| **loc**        | array of strings    | Location of the error (e.g., `"body", "input"`).                           |
| **msg**        | string              | Error message.                                                             |
| **type**       | string              | Type of error (e.g., `"validation_error"`).                                 |

#### Server Error (5XX)
Indicates server-side issues such as high traffic or unexpected failures.

---

## Notes

1. **Model Options**:
   - General-purpose: `voyage-3`, `voyage-3-lite`.
   - Domain-specific: `voyage-finance-2`, `voyage-law-2`, `voyage-multilingual-2`, `voyage-code-2`.
2. **Truncation**:
   - Ensure `truncation: true` for long texts, or validate text lengths beforehand.
3. **Embedding Format**:
   - Default: Floating-point vectors.
   - Use `encoding_format: "base64"` for compressed embeddings.
4. **Token Usage**:
   - The `usage.total_tokens` field provides insights into billing and token efficiency.

---
